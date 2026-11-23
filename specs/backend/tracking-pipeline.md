# Especificação: Pipeline de Ingestão de Telemetria (Rastreamento)

Especificação do fluxo de backend para recebimento e processamento em lote da telemetria dos veículos da frota Bora.

---

## 1. Endpoint de Ingestão em Lote

- **Rota:** `POST /api/v1/telemetry/positions`
- **Autenticação:** Bearer Token de motorista / dispositivo autorizado.
- **Frequência esperada:** 1 requisição a cada 15 segundos por veículo ativo (~40 req/s para frota de 600 ônibus).

### Payload de Entrada (Batch DTO):
```json
{
  "vehicleId": "JF-102",
  "lineId": "102",
  "points": [
    {
      "latitude": -21.7612,
      "longitude": -43.3501,
      "speed": 35.4,
      "heading": 182.0,
      "timestamp": "2026-11-23T18:14:45.000Z"
    },
    {
      "latitude": -21.7618,
      "longitude": -43.3504,
      "speed": 38.0,
      "heading": 180.5,
      "timestamp": "2026-11-23T18:15:00.000Z"
    }
  ]
}
```

---

## 2. Pipeline de Processamento no Backend

1. **Validação do Batch:** Garante que o array `points` contenha pelo menos 1 ponto e no máximo 100 pontos por requisição.
2. **Atualização de Cache em Memória (Redis):**
   - Extrai o ponto mais recente do lote.
   - Executa `SET veiculo:<vehicleId>:posicao '<json>' EX 120`.
   - Permite que passageiros consultem a posição em `< 0.1ms`.
3. **Persistência Relacional (PostgreSQL):**
   - **Tabela `veiculo_posicao_atual`:** Realiza `UPSERT` com a posição mais recente (para buscas consolidadas em fallback).
   - **Tabela `historico_posicoes`:** Insere todos os pontos do lote via *Bulk Insert* para auditoria de trajeto.
