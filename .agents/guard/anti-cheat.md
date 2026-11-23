# Guardrail: Anti-Cheat e Integridade de Código

Este documento estabelece salvaguardas invioláveis para impedir atalhos ou degradação da base de código por agentes de IA.

---

## Regras Invioláveis de Integridade

1. **Proibido Burlar Testes:**
   - Se um teste que passava anteriormente começar a falhar após uma alteração, **o código de produção está com defeito ou incompleto**.
   - É terminantemente proibido remover asserções, enfraquecer `expect(...)`, ou alterar dados mockados válidos apenas para forçar o status "Green".

2. **Proibido Silenciar Linter:**
   - Nunca adicione diretivas `// ignore: ...` ou `// ignore_for_file: ...` para ocultar warnings de análise estática, exceto em casos específicos de bibliotecas terceiras de teste quando aprovado formalmente.
   - O código deve ser corrigido na fonte para satisfazer o compilador e as regras de linting.

3. **Proibido Deixar Código em Estado Inválido:**
   - Nunca considere uma tarefa concluída com erros de compilação ou testes quebrados na suíte.
   - Se houver conflito de regras, pare, explique a discrepância ao usuário e aguarde orientação.
