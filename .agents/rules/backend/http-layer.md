# Camada HTTP e Divisão de Responsabilidades no Backend

Este documento estabelece as responsabilidades e a aplicação estrita do paradigma de **Orientação a Objetos (OO) Clássica** em cada camada da API **Bora Backend**.

---

## 1. O Ciclo da Requisição HTTP sob Orientação a Objetos

Todas as camadas são organizadas em classes especializadas, com dependências injetadas explicitamente pelo construtor:

```
Client (App Mobile / Web)
        │
        ▼
   [ Routes ]          (src/routes/*.ts: Mapeia rotas do Router e delega para instâncias de Controller)
        │
        ▼
  [ Middlewares ]      (src/middlewares/*.ts: Classes de interceptação, auth, validação e erro)
        │
        ▼
  [ Controllers ]      (src/controllers/*.ts: Classes que recebem req/res, validam DTOs e chamam Services)
        │
        ▼
   [ Services ]        (src/services/*.ts: Classes com regras de negócio puras, orquestrando Repositories)
        │
        ▼
 [ Repositories ]      (src/repositories/*.ts: Classes que implementam interfaces de acesso a dados/cache)
```

---

## 2. Regras por Camada e Modelagem Orientada a Objetos

### 2.1. Nomenclatura de Arquivos
- **Obrigatoriamente em `snake_case`:** `auth_controller.ts`, `auth_service.ts`, `user_repository.ts`, `postgres_user_repository.ts`.
- **Proibido:** `kebab-case` ou `camelCase` em nomes de arquivos ou pastas.

### 2.2. Controllers (`src/controllers/`)
- **Implementados obrigatoriamente como Classes:**
  ```typescript
  export class AuthController {
    constructor(private readonly authService: AuthService) {}

    public async login(req: Request, res: Response): Promise<Response> {
      const dto: LoginDto = req.body;
      const session = await this.authService.login(dto);
      return res.status(200).json(session);
    }
  }
  ```
- **Responsabilidade:** Única camada que acessa `req: Request` e `res: Response`. Extrai parâmetros, valida DTOs e invoca o `Service`.
- **Proibição:** Não contém lógica de regras de negócio nem executa queries SQL/ORM diretas.

### 2.3. Services (`src/services/`)
- **Implementados obrigatoriamente como Classes:**
  ```typescript
  export class AuthService {
    constructor(private readonly userRepository: UserRepository) {}

    public async login(dto: LoginDto): Promise<Session> {
      const user = await this.userRepository.findByEmail(dto.email);
      if (!user) {
        throw new UnauthorizedException('Credenciais inválidas');
      }
      // Regra de negócio e emissão de token...
      return session;
    }
  }
  ```
- **Responsabilidade:** Concentra as regras de negócio puras.
- **Proibição:** Agnóstico ao Express. Nunca recebe nem manipula objetos `req` ou `res`. Lança exceções que estendem `ProblemDetailsException` (RFC 9457).

### 2.4. Repositories (`src/repositories/`)
- **Contratos e Implementações em Classes:**
  ```typescript
  // src/repositories/user_repository.ts
  export interface UserRepository {
    findByEmail(email: string): Promise<User | null>;
    save(user: User): Promise<void>;
  }

  // src/repositories/postgres_user_repository.ts
  export class PostgresUserRepository implements UserRepository {
    constructor(private readonly dbPool: Pool) {}

    public async findByEmail(email: string): Promise<User | null> {
      // Query SQL isolada...
    }
  }
  ```
- **Responsabilidade:** Isola completamente o mecanismo de persistência (PostgreSQL, Redis, etc.) e converte resultados em entidades de domínio (`Models`).

### 2.5. Routes (`src/routes/`)
- Registra caminhos no `Router()` do Express associando à chamada dos métodos das instâncias dos controllers:
  ```typescript
  export function createAuthRouter(controller: AuthController): Router {
    const router = Router();
    router.post('/login', (req, res, next) => controller.login(req, res).catch(next));
    return router;
  }
  ```
