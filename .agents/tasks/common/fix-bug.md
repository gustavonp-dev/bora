# Playbook: Correção de Bugs (Bug-First TDD)

Procedimento mandatório para investigação e correção de qualquer defeito reportado no projeto **Bora** (Mobile e Backend).

---

## Procedimento Operacional

1. **Localização da Camada:**
   - Identifique em qual camada arquitetural o bug reside:
     - **Mobile:** Model, Service, Repository, ViewModel ou View.
     - **Backend:** Model, DTO, Service, Repository ou Controller.

2. **Criação do Teste de Reprodução (Red):**
   - **Antes de encostar no código de produção**, crie ou expanda um teste que reproduza exatamente o cenário da falha reportada.
   - Execute o teste e garanta que ele falha (**Red**). Isso comprova a existência do bug e impede falsos positivos.

3. **Correção Cirúrgica (Green):**
   - Altere o código de produção necessário para corrigir o problema.
   - Execute novamente o teste de reprodução até que ele seja aprovado com sucesso (**Green**).

4. **Verificação de Regressão e Validação Geral:**
   - Execute a suíte de testes e a análise estática correspondente ao escopo alterado:
     - Se alterou o mobile: `cd app && flutter test && flutter analyze lib/ test/`.
     - Se alterou o backend: `cd backend && npm run typecheck && npm test`.
