# Padrões Universais de Código e Engenharia

Este documento define os padrões universais de escrita, convenções e higiene de código que todos os agentes de IA devem seguir no ecossistema **Bora** (Mobile e Backend).

---

## 1. Nomenclatura e Convenções

### 1.1. Mobile (Dart / Flutter)
- **Arquivos e Pastas:** Sempre em `snake_case` (ex: `home_view.dart`, `upcoming_bus_card.dart`).
- **Classes, Enums e Tipos:** Sempre em `PascalCase` (ex: `HomeViewModel`, `UpcomingBus`).
- **Métodos, Funções e Variáveis:** Sempre em `camelCase` (ex: `selectTab`, `isUserLoggedIn`).
- **Constantes:** Em `camelCase` ou `SCREAMING_SNAKE_CASE` quando globais imutáveis.

### 1.2. Backend (TypeScript / Node.js)
- **Arquivos e Pastas:** `snake_case` ou `kebab-case` para nomes compostos (ex: `auth_controller.ts`, `health_check.ts`).
- **Classes, Interfaces e Types:** Sempre em `PascalCase` (ex: `AuthController`, `CreateUserDto`, `VehiclePosition`).
- **Métodos, Funções e Variáveis:** Sempre em `camelCase` (ex: `authenticateUser`, `vehicleId`).
- **Constantes de Ambiente e Globais:** Sempre em `SCREAMING_SNAKE_CASE` (ex: `DEFAULT_PORT`, `JWT_SECRET`).

---

## 2. Higiene de Código e Qualidade
- **Zero Warnings / Zero Errors:** Todo código entregue deve compilar e passar pelas ferramentas de análise estática sem nenhum aviso (`flutter analyze` ou `tsc --noEmit`).
- **Sem Código Morto:** Proibido deixar imports não utilizados, variáveis não lidas ou trechos de código comentados sem justificativa.
- **Responsabilidade Única (SRP):** Funções e classes devem ter um único propósito bem delimitado.
- **Evitar Código Duplicado (DRY):** Extraia funções utilitárias ou regras comuns sempre que houver repetição de lógica.

---

## 3. Padrão de Commits (Conventional Commits)
Todo commit deve seguir a convenção semântica com descrição clara em português:
- `feat:` Nova funcionalidade
- `fix:` Correção de bug
- `refactor:` Alteração de código sem mudar comportamento externo
- `test:` Adição ou ajuste de testes automatizados
- `chore:` Manutenção de dependências, configurações ou ferramentas
- `docs:` Alterações em documentação ou especificações
