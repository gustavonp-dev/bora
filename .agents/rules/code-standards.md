# Padrões de Código e Engenharia

Este documento define os padrões de escrita, estilo e boas práticas de engenharia que todos os agentes de IA devem seguir no projeto **Bora**.

---

## 1. Nomenclatura e Organização de Arquivos
- **Arquivos e Pastas:** Sempre em `snake_case` (ex: `home_view.dart`, `upcoming_bus_card.dart`).
- **Classes, Enums e Tipos:** Sempre em `PascalCase` (ex: `HomeViewModel`, `UpcomingBus`).
- **Métodos, Funções e Variáveis:** Sempre em `camelCase` (ex: `selectTab`, `isUserLoggedIn`).
- **Constantes:** Sempre em `camelCase` ou `SCREAMING_SNAKE_CASE` quando globais imutáveis.

---

## 2. Higiene de Código
- **Zero Warnings:** Todo código finalizado deve passar por `flutter analyze` com **0 issues**, 0 warnings e 0 infos.
- **Sem Código Morto:** Proibido deixar imports não utilizados, variáveis não lidas ou métodos comentados sem justificativa.
- **Modularização de Widgets:** Nunca crie árvores gigantescas em um único arquivo. Decomponha seções lógicas em widgets dedicados em `ui/<tela>/widgets/` ou `ui/widgets/`.

---

## 3. Uso Estrito de Design Tokens
- **Cores:** Utilize sempre os tokens de `AppColors` (`lib/commons/theme/app_colors.dart`). Nunca use `Color(0xFF...)` ou `Colors.blue` diretamente na UI.
- **Tipografia:** Utilize sempre os tokens de `AppTypography` (`lib/commons/theme/app_typography.dart`).
- **Espaçamentos e Bordas:** Utilize os tokens de `AppSpacing` (`lib/commons/theme/app_spacing.dart`).
