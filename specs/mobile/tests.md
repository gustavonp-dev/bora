# Especificação e Diretrizes de Testes - Bora

Este documento estabelece as regras mandatórias, padrões de desenvolvimento (TDD/BDD) e critérios de qualidade para a criação e execução de testes no aplicativo mobile **Bora**.

Como o desenvolvimento é orientado e executado por agentes de IA e desenvolvedores, **nenhuma funcionalidade ou correção é considerada concluída sem a respectiva suíte de testes aprovada**.

---

## 1. Pirâmide e Divisão de Testes

A pasta `test/` é estritamente segregada em três categorias, cada uma com escopo, velocidade e responsabilidades bem definidos:

```text
test/
├── unit/                       # Testes de Lógica Pura & Contratos (Rápidos / Sem árvore de Widgets)
│   ├── viewmodels/             # Mutação de estado, comandos e fluxos das ViewModels
│   ├── services/               # Regras de negócio centrais e orquestrações
│   ├── repositories/           # Serialização, tratamento de erros de API e persistência
│   └── commons/                # Validadores, extensions e formatters
├── widgets/                    # Testes de Interface & Comportamento (BDD / Headless / Mockado)
│   ├── views/                  # Telas completas (ciclo de vida, navegação, preenchimento, temas)
│   └── widgets/                # Componentes isolados (cards, inputs, sheets, badges, botões)
└── integration/                # Testes de Ponta a Ponta / E2E (Jornadas reais em dispositivo/emulador)
    └── flows/                  # Fluxos críticos de ponta a ponta
```

### 1.1. `test/unit/` (Unidade - Foco em Lógica e Estado)
- **Objetivo:** Testar regras de negócio, mutações de estado e contratos sem instanciar árvores visuais pesadas do Flutter.
- **Isolamento Total:** Repositórios, APIs, conexões externas e serviços periféricos devem ser 100% mockados/fakes.
- **Metodologia:** **TDD Obrigatório (Test-Driven Development)**:
  1. **Red:** Escreva o teste unitário que define a regra esperada antes de implementar. O teste deve falhar.
  2. **Green:** Escreva a implementação mínima para fazer o teste passar.
  3. **Refactor:** Limpe o código mantendo o teste passando.
- Proibido renderizar widgets (`WidgetTester`, `pumpWidget`) dentro de `test/unit/`.

### 1.2. `test/widgets/` (Widgets & Views - Foco em Experiência do Usuário)
- **Objetivo:** Garantir fidelidade visual, ausência de overflows, responsividade e reatividade aos estados.
- **Dados Mockados:** A interface consome ViewModels e dados mockados; não há requisições reais de rede ou banco.
- **Metodologia:** **BDD Obrigatório (Behavior-Driven Development)**:
  - Estrutura mental e descritiva dos testes: **Dado (Given) -> Quando (When) -> Então (Then)**.
  - Exemplo de descrição semântica:
    ```dart
    testWidgets('Dado que o usuário está na tela de login, quando tocar em Continuar com Google, então deve navegar para a Home', (tester) async { ... });
    ```
- **Responsividade Anti-Overflow Obrigatória:**
  - Telas completas devem ser testadas em pelo menos duas resoluções:
    1. **Padrão:** `393 x 852` (formato moderno alto).
    2. **Compacta:** `320 x 568` (formato compacto / iPhone SE 1st gen) para garantir que não ocorra `RenderFlex overflow`.

### 1.3. `test/integration/` (Integração & E2E - Foco em Jornadas Críticas)
- **Objetivo:** Validar fluxos completos de ponta a ponta sem dados fictícios na apresentação, cobrindo jornadas inteiras do usuário.
- **Execução:** Destinados a rodar com emulador/simulador ativo ou celular físico conectado.
- **Escopo Inicial:** Mapeado progressivamente conforme as jornadas críticas da aplicação forem fechadas.

---

## 2. Matriz de Decisão de Testes

Para eliminar dúvidas sobre qual tipo de teste criar ou atualizar ao alterar arquivos, todos os agentes e desenvolvedores devem seguir estritamente a matriz abaixo:

| O que foi criado ou alterado? | Onde deve ser testado? | Tipo de Teste & Regra Mandatória |
| :--- | :--- | :--- |
| **Nova ViewModel, Repository, Service ou Support** | `test/unit/<camada>/` | **TDD Obrigatório:** Criar teste unitário isolado e mockado cobrindo as regras de negócio, tratamentos de erro e mutações de estado. |
| **Alteração em ViewModel, Repository ou Service existente** | `test/unit/<camada>/` | **Ajuste de Teste Obrigatório:** Atualizar e expandir os testes unitários existentes para cobrir a nova regra/condição sem quebrar contratos anteriores. |
| **Nova View (Tela completa)** | `test/widgets/views/` | **Interface BDD:** Testar ciclo de vida, navegação, preenchimento de formulários, feedback visual (snackbars/banners/toasts), alternância de tema e responsividade anti-overflow (`393x852` e `320x568`). |
| **Novo Widget isolado (Card, Step, Sheet, Dialog, Input)** | `test/widgets/widgets/` | **Interface BDD:** Testar renderização nos estados (carregando, sucesso, erro, vazio) e interações específicas do componente em tela padrão e compacta. |
| **Alteração em View ou Widget existente** | `test/widgets/` | **Ajuste de Interface Obrigatório:** Atualizar os testes existentes para refletir as mudanças visuais e garantir que nenhuma regressão de layout ou overflow foi introduzida. |
| **Alteração que impacte uma Jornada Crítica de Usuário** | `test/integration/` | **Regressão E2E Obrigatória:** Executar o teste de ponta a ponta sem mocks correspondente à jornada em dispositivo físico ou virtual conectado. |
| **Correção de Bug reportado** | Camada onde o bug reside | **Bug-First TDD:** Antes de alterar o código de produção, criar obrigatoriamente um teste que reproduza a falha (Red); em seguida, corrigir o código até o teste passar (Green). |

---

## 3. Critérios de Execução e Qualidade

1. **Meta de Cobertura:** Cobertura mínima de **80%** das funcionalidades por testes unitários e de widgets.
2. **Checagem de Dispositivo para Testes de Integração:**
   - O agente ou desenvolvedor deve verificar dispositivos disponíveis com `flutter devices`.
   - **Dispositivo conectado:** Se houver aparelho físico (USB/Wi-Fi), priorizar a execução dos testes de integração nele.
   - **Sem dispositivo físico:** Inicializar um emulador Android ou simulador iOS para validação.
3. **Anti-Burlar Testes:**
   - É terminantemente proibido enfraquecer asserts, remover verificações ou alterar testes válidos apenas para forçar o "Green". O código de produção deve atender aos requisitos do teste, e não o inverso.
4. **Critério de Conclusão (Definition of Done):**
   - Nenhuma feature ou refatoração é considerada entregue sem a execução e validação da suíte de testes com zero erros (`flutter test`).
