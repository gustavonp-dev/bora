# Tratamento de Erros e Observabilidade no Backend

Este documento define o padrão arquitetural de tratamento de exceções, formatação de respostas de erro e observabilidade na API **Bora Backend**.

---

## 1. Princípios Fundamentais
1. **Nunca Derrubar o Processo do Node:** Nenhuma exceção não capturada (`uncaughtException` ou `unhandledRejection`) deve passar batida.
2. **Resposta Padronizada ao Cliente:** O frontend (ou app mobile) deve receber um formato previsível de erro, evitando expor stacktraces internos em produção.
3. **Logs Estruturados:** Erros de sistema (500) devem ser registrados no console com contexto claro (método, rota, payload resumido e stacktrace).

---

## 2. Formato Padronizado de Erro (RFC 7807 / Problem Details)

Toda resposta de erro da API deve seguir uma estrutura previsível:

```json
{
  "error": {
    "code": "VALIDATION_FAILED",
    "message": "Dados de entrada inválidos",
    "details": [
      { "field": "email", "message": "Formato de e-mail inválido" }
    ],
    "timestamp": "2026-11-23T18:00:00.000Z"
  }
}
```

---

## 3. Middleware Global de Erro no Express

Toda rota deve delegar erros inesperados para o middleware de erro do Express:

```typescript
import { Request, Response, NextFunction } from 'express';

export function errorHandler(
  err: any,
  req: Request,
  res: Response,
  next: NextFunction
) {
  const statusCode = err.statusCode || 500;
  const code = err.code || 'INTERNAL_SERVER_ERROR';
  const message = err.isOperational ? err.message : 'Ocorreu um erro interno no servidor';

  console.error(`[ERROR] ${req.method} ${req.url} - ${code}: ${err.message}`, {
    stack: err.stack,
  });

  return res.status(statusCode).json({
    error: {
      code,
      message,
      ...(err.details && { details: err.details }),
      timestamp: new Date().toISOString(),
    },
  });
}
```

---

## 4. Classificação de Erros
- **Erros Operacionais (4xx):** Falhas esperadas de negócio ou validação (ex: `BadRequestError`, `UnauthorizedError`, `NotFoundError`). Têm `isOperational = true` e mensagens seguras para o cliente.
- **Erros de Programação / Sistema (500):** Falhas inesperadas (ex: falha de conexão com PostgreSQL, bug em parsing). A mensagem para o cliente deve ser genérica ("Erro interno"), e o stacktrace completo gravado nos logs.
