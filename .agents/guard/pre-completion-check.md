# Guardrail: Checklist Pré-Conclusão (Definition of Done)

Nenhuma tarefa de desenvolvimento, refatoração ou correção de bug pode ser marcada como finalizada por um agente de IA sem a verificação rigorosa deste checklist condicional:

---

## 1. Verificação Condicional por Escopo Alterado

### A. Se houve alteração no Mobile (`app/`):
- [ ] **Análise Estática do Flutter Impecável:**
  - Comando: `cd app && flutter analyze lib/ test/`
  - Critério: **Zero issues** (0 warnings, 0 errors, 0 hints não autorizados).
- [ ] **Suíte de Testes do Flutter 100% Aprovada:**
  - Comando: `cd app && flutter test`
  - Critério: **Todos os testes passando**, sem skips ou falhas silenciosas.
- [ ] **Conformidade com a Matriz de Decisão (`specs/mobile/tests.md`):**
  - Se criou/alterou ViewModel/Service: há testes em `test/unit/`?
  - Se criou/alterou View: há teste BDD com anti-overflow (`320x568`) em `test/widgets/`?

### B. Se houve alteração no Backend (`backend/`):
- [ ] **Checagem Estática de Tipos TypeScript Impecável:**
  - Comando: `cd backend && npm run typecheck`
  - Critério: **Zero erros** de tipagem (`tsc --noEmit`).
- [ ] **Suíte de Testes do Backend Aprovada:**
  - Comando: `cd backend && npm test` (se configurado)
  - Critério: **100% dos testes passando**.
- [ ] **Conformidade com a Camada HTTP (`.agents/rules/backend/http-layer.md`):**
  - Controllers não contêm regras de negócio diretas?
  - Services não manipulam diretamente `req` ou `res` do Express?

---

## 2. Verificações Universais (Mobile e Backend)
- [ ] **Anti-Cheat:** Nenhuma asserção de teste foi enfraquecida, nenhum mock foi corrompido e nenhum aviso de linter foi silenciado com `// ignore: ...` ou `@ts-ignore`.
- [ ] **Limpeza de Arquivos e Código Morto:** Nenhum arquivo temporário, mock solto ou import não utilizado foi esquecido na árvore de arquivos.
