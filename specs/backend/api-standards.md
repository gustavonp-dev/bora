# Especificação de Padrões da API REST - Bora Backend

Este documento estabelece as convenções de contratos HTTP, formatos de payloads e respostas na API **Bora Backend**.

---

## 1. Padrões de Requisição e Resposta

- **Formato:** JSON em todas as comunicações (`Content-Type: application/json`).
- **Nomenclatura nos Payloads:** `camelCase` para propriedades JSON (ex: `vehicleId`, `updatedAt`, `lineName`).
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

### 2.2. Respostas de Erro (RFC 7807)
```json
{
  "error": {
    "code": "RESOURCE_NOT_FOUND",
    "message": "Veículo com o identificador informado não foi encontrado",
    "timestamp": "2026-11-23T18:00:00.000Z"
  }
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
