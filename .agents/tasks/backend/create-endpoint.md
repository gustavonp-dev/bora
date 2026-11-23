# Playbook: Criação de Novo Endpoint / Módulo no Backend

Guia passo a passo obrigatório para a criação de novos endpoints e módulos na API **Bora Backend**, seguindo a **Orientação a Objetos (OO) Clássica** e o padrão de erros **RFC 9457**.

---

## Procedimento Operacional

### 1. Nomenclatura e Modelagem (snake_case)
- Todos os arquivos e diretórios devem ser criados estritamente em `snake_case`.
- **DTOs (`src/dtos/<modulo>_dto.ts`):** Defina as interfaces ou classes de transferência de dados de entrada e saída.
- **Models (`src/models/<entidade>.ts`):** Defina as entidades ricas de domínio.

### 2. Criação da Service em Classe (TDD)
- Crie o teste unitário em `tests/unit/services/<modulo>_service.test.ts`.
- Implemente o teste que falha (**Red**), mockando as interfaces dos repositórios.
- Crie a classe em `src/services/<modulo>_service.ts`:
  - Receba os repositórios via construtor (`constructor(private readonly repo: ModuloRepository)`).
  - Lance exceções operacionais que estendem `ProblemDetailsException` (RFC 9457) em caso de erro de negócio.
- Execute o teste até obter aprovação (**Green**) e refatore mantendo a classe limpa (**Refactor**).

### 3. Criação do Repositório (Interface + Classe)
- Crie a interface do repositório em `src/repositories/<modulo>_repository.ts`.
- Crie a classe concreta (ex: `src/repositories/postgres_<modulo>_repository.ts`) implementando a interface e injetando a conexão de banco via construtor.

### 4. Criação do Controller em Classe
- Crie a classe em `src/controllers/<modulo>_controller.ts`:
  - Injete a Service via construtor (`constructor(private readonly service: ModuloService)`).
  - Implemente métodos públicos assíncronos (`public async handle(req: Request, res: Response): Promise<Response>`).
  - Extraia dados de `req.body`, `req.params` ou `req.query`, valide e chame a Service.
  - Retorne respostas com status HTTP adequados (`200 OK`, `201 Created`, `204 No Content`).

### 5. Criação da Rota e Injeção de Dependências
- Crie a rota em `src/routes/<modulo>_routes.ts`.
- Instancie as classes com injeção de dependências ou crie uma função factory:
  ```typescript
  const repository = new PostgresModuloRepository(dbPool);
  const service = new ModuloService(repository);
  const controller = new ModuloController(service);
  ```
- Registre o roteador em `src/index.ts`.

### 6. Teste de Integração HTTP (Supertest)
- Crie `tests/integration/routes/<modulo>_routes.test.ts`.
- Valide cenários de sucesso e respostas de erro estruturadas na RFC 9457 (`type`, `title`, `status`, `detail`).

### 7. Validação Final (Definition of Done)
- Execute o checklist do `pre-completion-check.md` para o backend:
  - `cd backend && npm run typecheck` com **0 erros**.
  - `cd backend && npm test` com **100% dos testes aprovados**.
