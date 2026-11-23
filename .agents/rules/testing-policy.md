# Política e Conduta de Testes

Este documento estabelece a conduta obrigatória para a criação e manutenção de testes automatizados por agentes de IA.

---

## 1. Princípios Fundamentais
- **TDD Obrigatório para Unidade (`test/unit/`):**
  1. Primeiro escreva o teste unitário que falha (**Red**).
  2. Implemente o código mínimo para atender ao contrato (**Green**).
  3. Refatore mantendo o teste passando (**Refactor**).
  - É proibido renderizar widgets dentro de testes unitários.
- **BDD Obrigatório para Interface (`test/widgets/`):**
  - Toda asserção deve seguir a estrutura semântica: **Dado (Given) -> Quando (When) -> Então (Then)**.
  - O foco deve ser o comportamento do usuário e o reflexo do estado na tela, não a existência vazia de tags de widgets.

---

## 2. Regra Anti-Overflow
- Todo teste de View completa deve validar a ausência de overflow em duas resoluções:
  1. `393 x 852` (resolução moderna padrão).
  2. `320 x 568` (resolução compacta / iPhone SE 1st gen).
- O agente deve utilizar o helper `TestScreenSizes.setScreenSize(tester, TestScreenSizes.compact)`.

---

## 3. Cobertura e Conclusão
- Nenhuma feature ou correção é considerada concluída sem testes automatizados correspondentes.
- Meta de cobertura mínima de **80%** das regras e telas implementadas.
- Consulte sempre a **Matriz de Decisão** em `specs/tests.md` para determinar onde criar o teste.
