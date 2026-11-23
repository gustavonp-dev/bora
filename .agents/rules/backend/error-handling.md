# Tratamento de Erros e Observabilidade no Backend

Este documento define o padrão arquitetural de tratamento de exceções, formatação de respostas de erro e observabilidade na API **Bora Backend**.

> [!IMPORTANT]
> O Bora adota a especificação oficial **RFC 9457 (Problem Details for HTTP APIs)**, que substitui e descontinua a antiga RFC 7807. Este padrão se aplica tanto à API (geração de erros) quanto ao aplicativo mobile (consumo e parsing).

---

## 1. Princípios Fundamentais
1. **Nunca Derrubar o Processo do Node:** Nenhuma exceção não capturada (`uncaughtException` ou `unhandledRejection`) deve interromper o servidor.
2. **Resposta Padronizada via RFC 9457:** O cliente (app mobile ou web) sempre recebe um payload estruturado, consistente e previsível sob `Content-Type: application/problem+json` ou `application/json`.
3. **Orientação a Objetos no Tratamento de Exceções:** Exceções operacionais devem ser representadas por classes que estendem uma classe abstrata de erro base.
4. **Logs Estruturados:** Erros de sistema (500) devem ser registrados no console com contexto claro (método HTTP, URL de instância, código e stacktrace completo).

---

## 2. Estrutura Padrão RFC 9457

A RFC 9457 define membros padronizados para descrever erros de HTTP:

| Membro | Tipo | Descrição |
| :--- | :--- | :--- |
| `type` | `string` (URI) | Identificador URI único da categoria do problema. Quando não houver documentação externa, usar `"about:blank"`. |
| `title` | `string` | Resumo curto e legível do tipo do problema (não varia por ocorrência). Ex: `"Validation Failed"`, `"Resource Not Found"`. |
| `status` | `number` | Código de status HTTP emitido pelo servidor para este erro. |
| `detail` | `string` | Explicação human-readable detalhada e específica para esta ocorrência do problema. |
| `instance` | `string` (URI) | Caminho da rota/recurso onde o erro foi gerado (ex: `/api/v1/auth/login`). |
| `invalidParams` | `array` *(Extensão)* | Lista de parâmetros com falha de validação em erros `400` / `422`. |
| `code` | `string` *(Extensão)* | Código de erro interno para consumo programático da máquina (ex: `INVALID_CREDENTIALS`). |
| `timestamp` | `string` *(Extensão)* | Data e hora em formato ISO 8601 UTC. |

### Exemplo de Resposta de Validação (400 Bad Request):
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

### Exemplo de Resposta de Recurso Não Encontrado (404 Not Found):
```json
{
  "type": "about:blank",
  "title": "Resource Not Found",
  "status": 404,
  "detail": "Nenhum veículo encontrado com o identificador 'JF-999'",
  "instance": "/api/v1/vehicles/JF-999",
  "code": "VEHICLE_NOT_FOUND",
  "timestamp": "2026-11-23T18:00:00.000Z"
}
```

---

## 3. Implementação Orientada a Objetos: Hierarquia de Exceções

O tratamento de erros deve usar classes e polimorfismo:

```typescript
// src/commons/errors/problem_details_exception.ts
export interface InvalidParam {
  name: string;
  reason: string;
}

export abstract class ProblemDetailsException extends Error {
  public abstract readonly type: string;
  public abstract readonly title: string;
  public abstract readonly status: number;
  public readonly isOperational: boolean = true;
  public readonly code?: string;
  public readonly invalidParams?: InvalidParam[];

  constructor(
    message: string,
    options?: {
      code?: string;
      invalidParams?: InvalidParam[];
    }
  ) {
    super(message);
    this.name = this.constructor.name;
    this.code = options?.code;
    this.invalidParams = options?.invalidParams;
    Error.captureStackTrace(this, this.constructor);
  }
}

// Subclasses especializadas:
export class ValidationException extends ProblemDetailsException {
  public readonly type = 'https://api.bora.com/problems/validation-failed';
  public readonly title = 'Validation Failed';
  public readonly status = 400;
}

export class NotFoundException extends ProblemDetailsException {
  public readonly type = 'about:blank';
  public readonly title = 'Resource Not Found';
  public readonly status = 404;
}

export class UnauthorizedException extends ProblemDetailsException {
  public readonly type = 'about:blank';
  public readonly title = 'Unauthorized';
  public readonly status = 401;
}
```

---

## 4. Middleware Global de Erro (Classe de Tratamento)

```typescript
// src/middlewares/error_handler_middleware.ts
import { Request, Response, NextFunction } from 'express';
import { ProblemDetailsException } from '../commons/errors/problem_details_exception.js';

export class ErrorHandlerMiddleware {
  public handle(err: any, req: Request, res: Response, _next: NextFunction): Response {
    const timestamp = new Date().toISOString();
    const instance = req.originalUrl || req.url;

    if (err instanceof ProblemDetailsException) {
      return res.status(err.status).contentType('application/problem+json').json({
        type: err.type,
        title: err.title,
        status: err.status,
        detail: err.message,
        instance,
        ...(err.code && { code: err.code }),
        ...(err.invalidParams && { invalidParams: err.invalidParams }),
        timestamp,
      });
    }

    // Erros não operacionais / inesperados (500)
    console.error(`[UNHANDLED_ERROR] ${req.method} ${instance}:`, {
      message: err?.message,
      stack: err?.stack,
    });

    return res.status(500).contentType('application/problem+json').json({
      type: 'about:blank',
      title: 'Internal Server Error',
      status: 500,
      detail: 'Ocorreu um erro interno inesperado no servidor',
      instance,
      code: 'INTERNAL_SERVER_ERROR',
      timestamp,
    });
  }
}
```
