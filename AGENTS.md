# Diretrizes Mestras para Agentes de IA - Projeto Bora

Você é um agente autônomo de engenharia de software trabalhando no ecossistema **Bora** (Mobile e Backend).

Antes de planejar, codificar ou alterar qualquer arquivo neste repositório, você **DEVE obrigatoriamente** consultar e obedecer a divisão de responsabilidades abaixo:

---

## 1. Regras de Conduta e Operação Técnica (`.agents/`)

### 1.1. Regras Globais e Universais (`.agents/rules/common/`)
- `code-standards.md`: Convenções de nomenclatura estritamente em `snake_case` (sem `kebab-case`), **Orientação a Objetos clássica obrigatória no backend**, zero warnings e conventional commits.
- `architecture-flow.md`: Respeito estrito ao fluxo unidirecional (DTO -> Service -> Repository).

### 1.2. Regras Específicas do Mobile (`.agents/rules/mobile/`)
- `testing-policy.md`: TDD obrigatório em testes unitários, BDD em testes de widgets e regra anti-overflow `320x568`.
- `user-feedback.md`: Diretriz de feedback visual obrigatório na interface via `BoraSnackbar` (4 estados).
- `exception-handling-observability.md`: Separação entre operações bloqueantes e não bloqueantes, consumo de erros no padrão **RFC 9457** e logs estruturados via `dart:developer`.

### 1.3. Regras Específicas do Backend (`.agents/rules/backend/`)
- `http-layer.md`: Separação de responsabilidades sob o paradigma de **Orientação a Objetos Clássica** (Classes para Controllers, Services e Repositories com injeção de dependências via construtor).
- `error-handling.md`: Middleware global de erro, hierarquia de classes de exceção, respostas no padrão oficial **RFC 9457 (Problem Details)** e logs estruturados.
- `testing-policy.md`: TDD em Services e testes de integração de rotas HTTP com Supertest.

### 1.4. Guardrails e Integridade (`.agents/guard/`)
- `anti-cheat.md`: Proibição estrita de enfraquecer testes, alterar mocks válidos ou silenciar analisadores estáticos.
- `pre-completion-check.md`: Checklist obrigatório e condicional executado antes de declarar qualquer entrega como concluída.

### 1.5. Playbooks Operacionais (`.agents/tasks/`)
- `common/fix-bug.md`: Playbook de Bug-First TDD para investigar e corrigir defeitos em qualquer camada.
- `mobile/create-screen.md`: Playbook passo a passo para criar novas telas e fluxos no Flutter.
- `backend/create-endpoint.md`: Playbook passo a passo para criar novos endpoints e módulos orientados a objetos na API Express.

---

## 2. Especificações de Domínio e Produto Bora (`specs/`)

### 2.1. Produto & Negócio Compartilhado (`specs/product/`)
- `tracking.md`: Arquitetura de telemetria em tempo real, dimensionamento de frota e rastreamento via GPS.

### 2.2. Mobile (`specs/mobile/`)
- `ui.md`: Especificação visual do Bora, design tokens (`AppColors`, `AppTypography`, `AppSpacing`) e layout.
- `tests.md`: Matriz de Decisão de Testes do Bora e critérios de aceitação no Flutter.

### 2.3. Backend (`specs/backend/`)
- `api-standards.md`: Padrões de design REST, envelopes de dados, respostas de erro em conformidade com a **RFC 9457** e verbos HTTP.
- `tracking-pipeline.md`: Especificação do endpoint de ingestão de GPS em lote, cache Redis e PostgreSQL.

---

## 3. Critério de Conclusão Obrigatório (Definition of Done)
Nenhuma entrega é considerada finalizada sem a validação do escopo afetado:
1. **Se alterou arquivos em `app/`:**
   - `cd app && flutter analyze lib/ test/` com **0 issues**.
   - `cd app && flutter test` com **100% dos testes aprovados**.
2. **Se alterou arquivos em `backend/`:**
   - `cd backend && npm run typecheck` com **0 erros**.
   - `cd backend && npm test` com **100% dos testes aprovados** (quando houver testes configurados).
3. **Se alterou ambos:** Ambas as baterias devem ser executadas e aprovadas com 100% de sucesso.
