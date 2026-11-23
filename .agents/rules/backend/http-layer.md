# Camada HTTP e Divisão de Responsabilidades no Backend

Este documento estabelece as responsabilidades de cada camada no ciclo de uma requisição HTTP no **Bora Backend**.

---

## 1. O Ciclo da Requisição HTTP

```
Client (App Mobile / Web)
        │
        ▼
   [ Routes ]          (Mapeamento de rotas e injeção de middlewares)
        │
        ▼
  [ Middlewares ]      (Autenticação JWT, validação de schema de entrada, CORS)
        │
        ▼
  [ Controllers ]      (Extração de params/body, conversão em DTO, chamada de Service)
        │
        ▼
   [ Services ]        (Regras de negócio puras, sem conhecimento de HTTP)
        │
        ▼
 [ Repositories ]      (Comunicação com banco de dados ou caches)
```

---

## 2. Regras por Camada

### 2.1. Routes (`src/routes/`)
- Apenas registra caminhos no `Router()` do Express e associa com os métodos do controller.
- Não contém lógica inline `(req, res) => { ... }` para regras complexas.

### 2.2. Controllers (`src/controllers/`)
- **Única camada que acessa `req: Request` e `res: Response`:**
  - Extrai `req.body`, `req.params`, `req.query`.
  - Passa os parâmetros tipados ou DTO para o `Service`.
  - Retorna a resposta com o status HTTP correspondente (`200 OK`, `201 Created`, `204 No Content`).
- **Proibição:** Controllers nunca realizam consultas SQL/ORM diretas nem contêm cálculos de regras de negócio.

### 2.3. Services (`src/services/`)
- Contém a inteligência do sistema.
- **Proibição:** Não recebe nem manipula objetos `req` ou `res`.
- Retorna dados puros (Models ou tipos primitivos) ou dispara exceções customizadas (ex: `AppError`).

### 2.4. Repositories (`src/repositories/`)
- Isola o banco de dados (PostgreSQL, Redis, Prisma, etc.).
- Se a tecnologia de banco mudar no futuro, apenas os repositórios são alterados.
