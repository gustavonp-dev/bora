# Padrões Universais de Código e Engenharia

Este documento define os padrões universais de escrita, convenções, paradigmas e higiene de código que todos os agentes de IA devem seguir no ecossistema **Bora** (Mobile e Backend).

---

## 1. Nomenclatura e Convenções

### 1.1. Padrão Estrito de Arquivos e Pastas (`snake_case`)
Tanto no **Mobile (Flutter/Dart)** quanto no **Backend (Node.js/TypeScript)**:
- **Arquivos e Pastas:** **Obrigatoriamente em `snake_case`** (ex: `home_view.dart`, `auth_controller.ts`, `vehicle_position.ts`, `health_check.ts`).
- **Proibição:** É proibido utilizar `kebab-case` ou `camelCase` em nomes de arquivos ou diretórios em qualquer parte do projeto.

### 1.2. Nomenclatura de Código
- **Classes, Interfaces, Types e Enums:** Sempre em `PascalCase` (ex: `HomeViewModel`, `AuthController`, `CreateUserDto`, `VehiclePositionRepository`).
- **Métodos, Funções e Variáveis:** Sempre em `camelCase` (ex: `selectTab`, `authenticateUser`, `vehicleId`).
- **Constantes de Ambiente e Globais Imutáveis:** Sempre em `SCREAMING_SNAKE_CASE` (ex: `DEFAULT_PORT`, `JWT_SECRET`, `MAX_POINTS_PER_BATCH`).

---

## 2. Paradigma Arquitetural do Backend: Orientação a Objetos (OO) Clássica Obrigatória

No backend do **Bora**, a arquitetura de código é **estritamente orientada a objetos**:

1. **Classes para Todos os Componentes:**
   - Controllers, Services, Repositories, Middlewares e Handlers devem ser implementados como **Classes**.
   - Proibido exportar funções anônimas ou funções avulsas soltas para lidar com rotas ou regras de negócio.
2. **Injeção de Dependências Clássica via Construtor:**
   - Dependências devem ser recebidas explicitamente no construtor da classe (`constructor(private readonly service: AuthService) {}`).
   - Evita dependências ocultas, facilita inversão de controle e permite substituição trivial por mocks em testes unitários.
3. **Encapsulamento e Modificadores de Acesso:**
   - Faça uso estrito de `private`, `protected`, `public` e `readonly` para proteger o estado interno e os métodos auxiliares de cada classe.
4. **Contratos Explícitos (Interfaces):**
   - Repositórios e serviços que acessam recursos externos devem implementar interfaces claras (ex: `class PostgresVehicleRepository implements VehicleRepository`).
5. **Restrição ao Paradigma Funcional:**
   - Código puramente funcional não deve ser utilizado para orquestração, regras de negócio ou camadas arquiteturais.
   - Funções puras avulsas são toleradas apenas para utilitários matemáticos simples ou pequenos formatters de texto quando estritamente necessário.

---

## 3. Higiene de Código e Qualidade
- **Zero Warnings / Zero Errors:** Todo código entregue deve compilar e passar pelas ferramentas de análise estática sem nenhum aviso (`flutter analyze` ou `tsc --noEmit`).
- **Sem Código Morto:** Proibido deixar imports não utilizados, variáveis não lidas ou trechos de código comentados sem justificativa.
- **Responsabilidade Única (SRP):** Classes e métodos devem ter um único propósito bem delimitado.
- **Evitar Código Duplicado (DRY):** Extraia lógica compartilhada respeitando o encapsulamento em classes utilitárias ou serviços especializados.

---

## 4. Padrão de Commits (Conventional Commits)
Todo commit deve seguir a convenção semântica com descrição clara em português:
- `feat:` Nova funcionalidade
- `fix:` Correção de bug
- `refactor:` Alteração de código sem mudar comportamento externo
- `test:` Adição ou ajuste de testes automatizados
- `chore:` Manutenção de dependências, configurações ou ferramentas
- `docs:` Alterações em documentação ou especificações
