# Guardrail: Checklist Pré-Conclusão (Definition of Done)

Nenhuma tarefa de desenvolvimento, refatoração ou correção de bug pode ser marcada como finalizada por um agente de IA sem a verificação rigorosa deste checklist:

---

## Checklist Obrigatório

- [ ] **1. Análise Estática Impecável:**
  - Comando: `cd app && flutter analyze lib/ test/`
  - Critério: **Zero issues** (0 warnings, 0 errors, 0 hints não autorizados).

- [ ] **2. Suíte de Testes 100% Aprovada:**
  - Comando: `cd app && flutter test`
  - Critério: **Todos os testes passando**, sem skips ou falhas silenciosas.

- [ ] **3. Conformidade com a Matriz de Decisão:**
  - Verifique `specs/tests.md`: se você criou ou alterou uma ViewModel/Service, há testes em `test/unit/`? Se criou/alterou uma View, há teste BDD com anti-overflow em `test/widgets/`?

- [ ] **4. Limpeza de Arquivos e Imports:**
  - Nenhum arquivo temporário, mock solto ou import morto foi esquecido na árvore de arquivos.
