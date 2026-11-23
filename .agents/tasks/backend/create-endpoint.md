# Playbook: Criação de Novo Endpoint / Módulo no Backend

Guia passo a passo obrigatório para a criação de novos endpoints e módulos na API **Bora Backend**.

---

## Procedimento Operacional

1. **Modelagem de Dados (DTOs e Models):**
   - Defina os contratos de entrada e saída em `src/dtos/` (ex: `create_ride_dto.ts`).
   - Se houver nova entidade de banco ou domínio, defina o modelo em `src/models/` (ex: `ride.ts`).

2. **Criação da Regra de Negócio na Service (TDD):**
   - Crie a especificação de teste da Service em `tests/unit/services/<modulo>_service.test.ts`.
   - Implemente o teste que falha (**Red**).
   - Implemente a lógica correspondente em `src/services/<modulo>_service.ts` até que passe (**Green**).
   - Refatore o código conforme necessário (**Refactor**).

3. **Criação do Repositório (se envolver persistência):**
   - Crie a interface e implementação em `src/repositories/<modulo>_repository.ts`.
   - Mantenha queries SQL ou comandos de cache isolados no repositório.

4. **Criação do Controller:**
   - Crie `src/controllers/<modulo>_controller.ts`.
   - Extraia dados de `req.body`, `req.params` ou `req.query`, valide e chame a Service.
   - Responda com o status HTTP adequado (`200`, `201`, `204`).

5. **Criação da Rota e Registro:**
   - Crie `src/routes/<modulo>_routes.ts` com o `Router` do Express.
   - Anexe middlewares necessários (ex: autenticação, validação).
   - Registre o roteador em `src/index.ts` (ou no agregador de rotas).

6. **Teste de Integração HTTP:**
   - Crie o teste de rota (ex: `tests/integration/routes/<modulo>_routes.test.ts`) utilizando Supertest.
   - Valide status codes de sucesso e cenários de erro/validação.

7. **Validação Final:**
   - Execute o checklist do `pre-completion-check.md` para o backend (`npm run typecheck && npm test`).
