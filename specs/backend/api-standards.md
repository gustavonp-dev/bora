# Especificação de Padrões da API REST - Bora Backend

Este documento estabelece as convenções de contratos HTTP, formatos de payloads e respostas na API **Bora Backend**.

---

## 1. Padrões de Requisição e Resposta

- **Formato:** JSON em todas as comunicações (`Content-Type: application/json` ou `application/problem+json`).
- **Nomenclatura nos Payloads:** `camelCase` para propriedades JSON (ex: `vehicleId`, `updatedAt`, `lineName`).
- **Nomenclatura de Arquivos e Pastas:** Estritamente `snake_case` em todo o código backend.
- **Datas e Horários:** Padrão ISO 8601 em UTC (ex: `2026-11-23T18:00:00.000Z`).

---

## 2. Envelope de Resposta

### 2.1. Respostas de Sucesso
- **Recurso Único:** Retorna o objeto diretamente ou envolvido em `{ data: { ... } }`.
- **Listas e Coleções Paginadas:**
  ```json
  {
    "data": [ ... ],
    "meta": {
      "page": 1,
      "limit": 20,
      "total": 142,
      "totalPages": 8
    }
  }
  ```

### 2.2. Respostas de Erro: RFC 9457 (Problem Details for HTTP APIs)
> [!IMPORTANT]
> A API Bora adota estritamente o padrão oficial **RFC 9457**, que substitui e descontinua a RFC 7807.

Toda resposta com status code $\ge 400$ deve ser retornada com `Content-Type: application/problem+json`:

```json
{
  "type": "https://api.bora.com/problems/validation-failed",
  "title": "Validation Failed",
  "status": 400,
  "detail": "O campo 'points' deve conter ao menos uma coordenada válida",
  "instance": "/api/v1/telemetry/positions",
  "code": "INVALID_BATCH_PAYLOAD",
  "invalidParams": [
    {
      "name": "points",
      "reason": "O lote de coordenadas não pode estar vazio"
    }
  ],
  "timestamp": "2026-11-23T18:00:00.000Z"
}
```

---

## 3. Uso Semântico de Verbos e Status Codes HTTP

| Verbo | Uso | Sucesso Típico | Erro Comum |
| :--- | :--- | :--- | :--- |
| `GET` | Consulta e leitura idempotente | `200 OK` | `404 Not Found` |
| `POST` | Criação de recurso ou ação não idempotente | `201 Created` / `200 OK` | `400 Bad Request`, `422 Unprocessable` |
| `PUT` | Substituição completa de recurso | `200 OK` | `404 Not Found`, `400 Bad Request` |
| `PATCH` | Atualização parcial de campos | `200 OK` | `404 Not Found`, `400 Bad Request` |
| `DELETE` | Remoção de recurso | `204 No Content` / `200 OK` | `404 Not Found` |
