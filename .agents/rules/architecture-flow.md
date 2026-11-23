# Fluxo Arquitetural e Limites de Camadas

Este documento estabelece as regras de comunicação entre camadas no projeto **Bora**.

---

## 1. O Fluxo Unidirecional ("A Flecha")

```text
View  ──(ações/eventos)──>  ViewModel  ──(solicitações)──>  Service  ──(DTOs saída)──>  Repository  ──>  Mundo Externo (API/Storage)
View  <──(notifica estado)── ViewModel <──(Models limpos)── Service <──(Models convertidos)─ Repository <──  Dados crus/JSON
```

---

## 2. Regras de Ouro de Cada Camada

1. **View (`lib/ui/<tela>/`):**
   - Responsabilidade exclusiva de renderização.
   - Não contém regras de negócio ou chamadas a serviços/repositórios. Delega tudo para sua `ViewModel`.

2. **ViewModel (`lib/ui/<tela>/`):**
   - Gerencia o estado reativo da tela correspondente (MVVM).
   - Aciona os `Services` e atualiza o estado para que a View reaja.

3. **Service (`lib/services/`):**
   - Guardião das regras de negócio.
   - Totalmente agnóstico a Flutter (`dart:ui`, widgets) e a bibliotecas de rede (`http`, `dio`).
   - Pode receber DTOs nas solicitações de entrada (formulários), mas **nunca cria DTOs**.
   - Orquestra chamadas a Repositories e opera com entidades de domínio (`Models`).

4. **Repository (`lib/repositories/`):**
   - Camada especialista de comunicação com o mundo externo (APIs REST, GraphQL, SharedPreferences, SQLite).
   - É o único responsável por transformar respostas brutas e DTOs em `Models` de domínio limpos antes de retornar ao `Service`.

5. **Commons (`lib/commons/`):**
   - Recursos transversais compartilhados: `theme/`, `routes/`, `di/`, `validators/`, `extensions/`, `types/`.
   - Proibido colocar lógica de negócio ou telas dentro de `commons/`.
