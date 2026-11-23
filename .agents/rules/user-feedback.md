# Diretriz de Feedback Visual ao Usuário (BoraSnackbar)

Este documento estabelece as regras de interface para fornecer feedback claro, imediato e acessível a qualquer ação do usuário no aplicativo **Bora**.

---

## 1. Princípio Fundamental
**Nenhuma ação iniciada pelo usuário deve terminar em silêncio.**
Seja um cadastro realizado, um formulário com campos incompletos, uma falha de conexão ou um aviso do sistema, a interface deve reagir fornecendo feedback visual correspondente.

---

## 2. Os 4 Estados do `BoraSnackbar`

Toda notificação temporária no app deve utilizar o componente oficial `BoraSnackbar` (`lib/ui/widgets/bora_snackbar.dart`), respeitando os 4 estados:

| Estado | Cor Base / Superfície | Ícone | Quando Usar | Método Utilitário |
| :--- | :--- | :--- | :--- | :--- |
| **Success** | Verde (`AppColors.successSurface` / `AppColors.success`) | `Icons.check_circle_rounded` | Ação executada com êxito (compra confirmada, cadastro concluído, rota favoritada). | `BoraSnackbar.showSuccess(context, ...)` |
| **Error** | Vermelho (`AppColors.failSurface` / `AppColors.fail`) | `Icons.cancel_rounded` | Falhas em operações bloqueantes (erro de rede, falha de autenticação, timeout). | `BoraSnackbar.showError(context, ...)` |
| **Warning** | Amarelo/Laranja (`AppColors.warningSurface` / `AppColors.warning`) | `Icons.warning_amber_rounded` | Operações incompletas ou ações que exigem atenção (campos pendentes, GPS desativado). | `BoraSnackbar.showWarning(context, ...)` |
| **Info** | Preto Claro / Slate (`AppColors.infoSurface` / `AppColors.info`) | `Icons.info_rounded` | Avisos gerais, notas informativas ou status do sistema (modo demonstrativo, horários especiais). | `BoraSnackbar.showInfo(context, ...)` |

---

## 3. Regras de Implementação
- **Proibição de SnackBar genérico:** É proibido instanciar `SnackBar` nativo cru sem a identidade visual do Bora.
- **Floating & Acessível:** Os feedbacks devem flutuar sobre o conteúdo com cantos arredondados (`AppSpacing.radiusControl`) e texto legível (`AppTypography.body` / `AppTypography.label`).
- **Integração com Observabilidade:** Se um erro for exibido via `BoraSnackbar.showError` por falha de sistema inesperada, ele deve ser acompanhado de `developer.log(...)` conforme a diretriz `exception-handling-observability.md`.
