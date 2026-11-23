# Arquitetura Técnica & Rastreamento em Tempo Real
 
## 1. Visão Geral & Contexto do Problema

O **Bora** foi concebido para resolver o problema de mobilidade urbana e transporte coletivo, fornecendo bilhetagem digital e previsibilidade de chegada aos passageiros.

Em vez de depender de hardware dedicado proprietário (rastreadores OBD/GPS fixos) e homologações burocráticas complexas com operadores de frotas, o Bora adota o modelo de telemetria via **aplicativo instalado no smartphone do condutor (Driver App)**. 

Essa mesma abordagem é a base técnica de operações modernas de logística sob demanda e frotas terceirizadas/agregadas (ex.: iFood, entregadores autônomos e frotas urbanas).

---

## 2. Camada Mobile (Flutter): Permissões de SO & Execução em Segundo Plano

O principal desafio de rastreamento contínuo em smartphones são as políticas agressivas de economia de energia dos sistemas operacionais (*Doze Mode* no Android e suspensão de processos no iOS). 

Para impedir que o sistema operacional derrube a coleta de GPS enquanto o motorista conduz a rota, o aplicativo implementa as seguintes diretrizes nativas:

### 2.1. Android (`AndroidManifest.xml`)

* `ACCESS_FINE_LOCATION` & `ACCESS_COARSE_LOCATION`: Leitura precisa das coordenadas via satélite (GPS) e triangulação de rede.
* `ACCESS_BACKGROUND_LOCATION`: Obrigatório a partir do Android 10 (API 29) para manter acesso à localização com o app minimizado ou em segundo plano.
* `FOREGROUND_SERVICE` & `FOREGROUND_SERVICE_LOCATION`: Exigência crítica introduzida no Android 14 (API 34), registrando explicitamente o serviço em primeiro plano como do tipo `location`.
* **Foreground Service com Notificação Persistente:** O rastreador roda atrelado a um serviço em primeiro plano nativo, exibindo obrigatoriamente uma notificação fixa na bandeja do sistema (*"Bora: Rota de viagem em andamento"*). Isso impede que o gerenciador de memória/bateria do Android finalize o processo.
* **Isenção de Bateria (`REQUEST_IGNORE_BATTERY_OPTIMIZATIONS`):** Orientação ativa ao condutor para desabilitar a otimização de bateria específica do aparelho (vital para mitigar fechamentos forçados em sistemas como Xiaomi/MIUI e Samsung).

### 2.2. iOS (`Info.plist` & Capabilities)

* `NSLocationWhenInUseUsageDescription` & `NSLocationAlwaysAndWhenInUseUsageDescription`: Justificativas transparentes exigidas pela Apple para aprovação na App Store, demonstrando que o uso de GPS contínuo é fundamental para a telemetria da rota.
* `UIBackgroundModes`: Habilitação obrigatória de `location` (*Location updates*) e `fetch` (*Background processing*) nas Capabilities do Xcode.
* `allowsBackgroundLocationUpdates = true`: Configuração no `LocationManager` para manter o fluxo de coordenadas ativo mesmo com a tela do aparelho bloqueada.

---

## 3. Estratégia de Comunicação & Dimensionamento de Tráfego

### 3.1. Estudo de Caso Real: Frota de Juiz de Fora (~600 Ônibus)

Tomando como parâmetro a frota municipal integral de Juiz de Fora (MG), composta por aproximadamente 600 ônibus em circulação simultânea nos horários de pico:

$$\text{Taxa de Requisições} = \frac{600 \text{ veículos}}{15 \text{ segundos}} = 40 \text{ requisições/segundo (req/s)}$$

* **Conclusão de Engenharia:** Uma carga de **40 req/s** com payloads JSON leves (~300 bytes) é uma demanda trivial para um único servidor modesto (VPS de 2 vCPUs e 4GB de RAM).
* **Decisão contra o Overengineering:** Provisionar Load Balancers, clusters de Kubernetes ou brokers complexos de mensageria nessa fase seria puro desperdício financeiro e complexidade operacional precoce.

### 3.2. Comparativo de Protocolos

| Critério | HTTP POST em Lote (Batching) | MQTT (Broker Dedicado) | WebSocket |
| :--- | :--- | :--- | :--- |
| **Arquitetura Servidor** | **Stateless.** Roda em servidor único (VPS) sem balanceador. | **Stateful.** Exige Broker (EMQX, Mosquitto ou IoT Core). | **Stateful.** Mantém conexões TCP abertas + cluster Redis. |
| **Custo de Nuvem** | **Mínimo** (~R$ 50/mês em VPS simples). | **Médio/Alto.** Cobrança por conexões/mensagens ativas. | **Médio.** Exige memória dedicada permanente por socket. |
| **Comportamento em 4G** | **Excelente.** Rádio 4G liga, envia lote e desliga. | **Muito bom.** Reconexão leve (headers de 2 bytes). | **Instável.** Quedas de torre geram handshakes pesados. |
| **Latência** | **Near Real-Time** (10 a 15 segundos). | **Real-Time puro** (< 1 segundo). | **Real-Time puro** (< 1 segundo). |
| **Resiliência Offline** | **Nativa.** Buffer SQLite local segura dados sem perda. | Exige persistência complexa de sessão na ponta. | Complexa; exige fila manual de reconexão no mobile. |

---

## 4. Decisão Arquitetural do Bora

A arquitetura final do Bora foi desenhada sob quatro pilares estratégicos:

```
[ GPS Flutter ] 
      │ (a cada 3s)
      ▼
[ SQLite Local ] ──(a cada 15s em lote)──► [ API Backend  ]
                                                   │
                         ┌─────────────────────────┴─────────────────────────┐
                         ▼                                                   ▼
                [ Cache Redis ]                                     [ Pool de Conexões ]
             (Última Posição / TTL)                                          │
                         │                                                   ▼
                         ▼                                          [ Banco de Dados ]
           [ Consulta do Passageiro ]                             (Posição Atual + Histórico)
```

### 4.1. Pilares da Decisão

1. **Infraestrutura Enxuta em Servidor Único (VPS):**  
   Com a média de 40 req/s, uma única máquina virtual com SSD NVMe processa o backend e o banco sem saturação de recursos.

2. **Resiliência Total a Túneis e Áreas de Sombra (Offline-First via SQLite):**  
   Ao atravessar túneis, garagens ou rodovias sem sinal, as coordenadas capturadas a cada 3 segundos são acumuladas em uma tabela local no **SQLite** (via `sqflite` / `drift`). A cada 15 segundos, o app dispara o lote acumulado via HTTP. Se a requisição falhar (sem internet ou status != 200), nada é perdido: os pontos permanecem no banco local e são descarregados automaticamente na próxima janela de conexão.

3. **Simplicidade de Manutenção & Operação:**  
   Endpoints REST tradicionais são universais, fáceis de debugar, testáveis via Postman/curl e livres do gerenciamento de desconexões silenciosas de sockets.

4. **Alinhamento com a Tolerância do Público-Alvo:**  
   Para quem aguarda o transporte coletivo, um intervalo de atualização de 10 a 15 segundos é imperceptível visualmente. Historicamente, os usuários não possuíam visibilidade alguma; receber a previsão com delay de 15s entrega 100% do valor percebido com custo de infraestrutura quase nulo.

---

## 5. Estratégia de Banco de Dados & Mitigação de Gargalos

O gargalo de 40 req/s nunca é a CPU do servidor, mas sim o **I/O de disco do banco de dados**. Para evitar degradação sob alta volumetria (600 ônibus geram ~69 milhões de coordenadas por mês), adotamos a seguinte arquitetura de persistência:

### 5.1. Connection Pooling (Pool de Conexões)
* Abrir e fechar conexões TCP a cada uma das 40 req/s travaria o banco por esgotamento de *file descriptors* e alocação de memória.
* **Solução:** Configuração de um pool ativo no backend  mantendo **15 a 25 conexões persistentes e aquecidas**. As requisições pegam uma conexão emprestada, executam em ~2ms e devolvem ao pool.

### 5.2. Cache em Memória com Redis (Leitura Instantânea do Passageiro)
* O passageiro que abre o mapa quer saber apenas onde o ônibus está **agora**. Consultar a tabela histórica para isso seria ineficiente.
* A posição mais recente é salva no **Redis** com chave indexada e tempo de expiração (TTL):
  ```redis
  SET onibus:102:posicao '{"lat": -21.7612, "lng": -43.3501, "speed": 38, "updated_at": 1792500000}' EX 120
  ```
* O `EX 120` (expira em 2 minutos) atua como trava de segurança: se o ônibus desligar ou perder o sinal prolongadamente, a chave expira automaticamente, indicando que o veículo está offline.
* O passageiro consulta o Redis em memória RAM com latência inferior a **0.1ms**, sem gerar nenhuma carga de leitura no PostgreSQL.

### 5.3. Modelagem Relacional & Otimização de Busca

Para suporte, relatórios e fallback (caso a chave do Redis expire), o PostgreSQL adota duas tabelas segregadas:

#### Tabela de Estado Atual (`veiculo_posicao_atual`)
Possui **exatamente 600 linhas** (uma por veículo da frota). A cada lote, o backend realiza um `UPSERT`:
```sql
INSERT INTO veiculo_posicao_atual (onibus_id, linha_id, lat, lng, speed, updated_at)
VALUES ($1, $2, $3, $4, $5, $6)
ON CONFLICT (onibus_id) 
DO UPDATE SET 
    lat = EXCLUDED.lat,
    lng = EXCLUDED.lng,
    speed = EXCLUDED.speed,
    updated_at = EXCLUDED.updated_at;
```
* **Performance:** Buscar a posição atual de qualquer linha ou veículo nessa tabela leva menos de **0.5ms**, pois a tabela inteira cabe na memória compartilhada (*shared buffers*) do PostgreSQL.

#### Tabela de Histórico (`historico_posicoes`)
Recebe os lotes via *Bulk Insert* para auditoria de trajeto e conformidade operacional.

* **Índice Composto Otimizado:**
  ```sql
  CREATE INDEX idx_historico_onibus_data ON historico_posicoes (onibus_id, created_at DESC);
  ```
  Permite que buscas de fallback ou do último trajeto utilizem uma varredura direta no topo da árvore B-Tree:
  ```sql
  SELECT lat, lng, updated_at 
  FROM historico_posicoes 
  WHERE onibus_id = 'JF-102' 
  ORDER BY created_at DESC 
  LIMIT 1;
  ```
  Tempo de resposta: **~1ms**, mesmo com dezenas de milhões de registros.

---

## 6. Camada de Mapas & Representação de Trajetos (Análise de Custos & Decisão)

A renderização cartográfica e o traçado de rotas representam um dos maiores riscos de custos recorrentes (*vendor lock-in*) em aplicações geográficas se forem utilizados serviços proprietários de forma ingênua.

### 6.1. Comparativo de Provedores de Mapas

| Provedor | Custo de Exibição de Mapa | Custo de Traçado de Rotas / Directions | Limitações & Riscos |
| :--- | :--- | :--- | :--- |
| **Google Maps Platform** | Gratuito no Mobile (SDK nativo). Porém, na Web custa **\$7 por 1.000 cargas** após crédito de \$200. | **\$5 por 1.000 requisições** (Directions API). Se 5.000 passageiros abrirem rotas/dia = **\$750/mês (~R$ 4.200)**. | Risco financeiro altíssimo se o aplicativo escalar; dependência total de faturamento em dólar. |
| **Apple MapKit** | Gratuito no ecossistema iOS (incluso na anuidade de dev de \$99). | Incluso gratuitamente para chamadas dentro do app iOS. | **Inviável para o público amplo:** Não funciona no Android (que representa >85% dos usuários de transporte coletivo no Brasil). |
| **OpenStreetMap (OSM) via `flutter_map`** | **100% Gratuito e Open-Source.** Tiles abertos e livres de royalties. | **R$ 0,00.** Trajetos pré-processados e servidos internamente. | Servidor público oficial (`tile.openstreetmap.org`) não suporta tráfego comercial em massa sem cache. |

### 6.2. Estratégia de Trajetos Pré-calculados (Polylines / GeoJSON)

O trajeto de uma linha de ônibus municipal (ex.: Linha 102 - Santa Luzia / Centro) **é estático**; ele não se altera dinamicamente como o trajeto de um táxi ou Uber. 

* **Anti-Pattern Evitado:** Nunca chamar APIs de direção (Google Directions / Mapbox) dinamicamente a cada visualização do passageiro.
* **Solução Adotada:** 
  1. O trajeto oficial de cada linha é gerado uma única vez (via ferramenta externa gratuita como QGIS ou OSRM - *Open Source Routing Machine*).
  2. A geometria das ruas é codificada em **Encoded Polyline** (algoritmo padrão do Google) ou **GeoJSON compacto**, pesando **menos de 3 KB por linha**.
  3. Essas strings são salvas na tabela de `linhas` do banco de dados e enviadas ao app apenas no momento em que o usuário seleciona a linha desejada:
     ```sql
     SELECT id, nome, polyline_trajeto FROM linhas WHERE id = '102';
     ```
  4. O Flutter renderiza a rota instantaneamente no asfalto com a camada `PolylineLayer` do pacote `flutter_map`, sem qualquer consumo de API externa.

### 6.3. Decisão de Custos: OpenStreetMap + Cache Local no Flutter

A decisão adotada para o Bora foi a utilização do **OpenStreetMap** através do pacote **`flutter_map`**, complementado por **cache local de tiles no dispositivo**:

1. **Custo Zero de Licenciamento:**  
   Elimina completamente a dependência de créditos de nuvem do Google Maps ou faturas surpresa em dólar.

2. **Cache Local no Smartphone (`flutter_map_cache`):**  
   Como a área geográfica de operação é delimitada (ex.: o município de Juiz de Fora), o aplicativo armazena em disco local (cache SQLite/Hive) os blocos cartográficos (*tiles*) navegados pelo usuário. 
   * **Benefício 1:** O mapa abre instantaneamente, sem delay de carregamento.
   * **Benefício 2:** Funciona mesmo quando o passageiro estiver com conexão 3G instável ou offline no ponto de ônibus.
   * **Benefício 3:** Reduz a zero a sobrecarga em servidores de terceiros.

3. **Renderização dos Ônibus:**  
   Em cima da linha estática (Polyline do trajeto), o Flutter apenas renderiza marcadores animados (`MarkerLayer`) posicionados na latitude/longitude atual consumida do cache Redis a cada 15 segundos.
