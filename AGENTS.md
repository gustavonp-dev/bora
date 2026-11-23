# Diretrizes Mestras para Agentes de IA - Projeto Bora

Você é um agente autônomo de engenharia de software trabalhando no ecossistema mobile **Bora**.

Antes de planejar, codificar ou alterar qualquer arquivo neste repositório, você **DEVE obrigatoriamente** consultar e obedecer a divisão de responsabilidades abaixo:

---

## 1. Regras de Conduta e Operação Técnica (`.agents/`)
Contém as regras de como você deve se comportar como engenheiro de software:

- **`.agents/rules/`:**
  - `code-standards.md`: Convenções de nomenclatura (snake_case), zero warnings, modularização.
  - `testing-policy.md`: TDD obrigatório em unitários, BDD em widgets, anti-overflow `320x568`.
  - `architecture-flow.md`: Respeito ao fluxo da flecha (View -> ViewModel -> Service -> Repository).
  - `exception-handling-observability.md`: Diretriz de exceções (bloqueantes vs não bloqueantes) e logs via `dart:developer`.
  - `user-feedback.md`: Diretriz de feedback visual obrigatório via `BoraSnackbar` (Success, Error, Warning, Info).
- **`.agents/guard/`:**
  - `anti-cheat.md`: Proibição estrita de enfraquecer testes ou silenciar o linter.
  - `pre-completion-check.md`: Checklist obrigatório antes de declarar tarefa concluída (analyze 0 issues + test 100%).
- **`.agents/tasks/`:**
  - `create-screen.md`: Playbook passo a passo para criar novas telas e fluxos.
  - `fix-bug.md`: Playbook de Bug-First TDD para investigar e corrigir defeitos.

---

## 2. Especificações de Domínio e Produto Bora (`specs/`)
Contém o detalhamento de negócio, produto e regras específicas da aplicação Bora:

- **`specs/ui.md`:** Especificação visual do Bora, design tokens (`commons/theme`), layout das telas e regras de apresentação.
- **`specs/tests.md`:** Matriz de Decisão de Testes do Bora e critérios de aceitação.
- **`specs/tracking.md`:** Estratégia de mapas (`flutter_map`), rastreamento de veículos e polylines.

---

## 3. Critério de Conclusão Obrigatório
Nenhuma entrega é considerada finalizada sem:
1. `cd app && flutter analyze lib/ test/` com **0 issues**.
2. `cd app && flutter test` com **100% dos testes aprovados**.
