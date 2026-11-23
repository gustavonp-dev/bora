# Playbook: Criação de Nova Tela (Feature/View)

Guia passo a passo obrigatório para criação de novas telas no aplicativo **Bora**.

---

## Procedimento Operacional

1. **Definição e Modelagem:**
   - Consulte `specs/ui.md` e o layout/requisito da tela.
   - Se a tela demandar novas entidades ou contratos, crie os modelos em `lib/models/` ou DTOs em `lib/dto/`.

2. **Criação da ViewModel (TDD):**
   - Crie `lib/ui/<tela>/<tela>_view_model.dart`.
   - Crie o teste unitário isolado em `test/unit/viewmodels/<tela>_view_model_test.dart`.
   - Siga o ciclo **Red -> Green -> Refactor**, cobrindo estados iniciais, mutações, erros e notificações.

3. **Criação da View & Componentes Modulares:**
   - Crie `lib/ui/<tela>/<tela>_view.dart`.
   - Seções complexas devem ser extraídas para `lib/ui/<tela>/widgets/`.
   - Utilize estritamente os design tokens de `lib/commons/theme/`.

4. **Criação dos Testes de Interface (BDD):**
   - Crie `test/widgets/views/<tela>_view_test.dart`.
   - Escreva testes no padrão BDD: *Dado (Given) -> Quando (When) -> Então (Then)*.
   - **Obrigatório:** Adicione teste de responsividade anti-overflow na resolução compacta (`320 x 568`) usando `TestScreenSizes.setScreenSize`.

5. **Injeção de Dependências e Rotas:**
   - Registre a nova ViewModel no GetIt (`lib/commons/di/injection.dart`).
   - Adicione a rota correspondente no GoRouter (`lib/commons/routes/router.dart`).

6. **Validação Final:**
   - Execute o checklist de `guard/pre-completion-check.md`.
