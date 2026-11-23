# Fluxo Arquitetural Unidirecional e Limites de Camadas

Este documento estabelece o princípio arquitetural que rege tanto o aplicativo mobile quanto a API backend do **Bora**.

---

## 1. O Princípio da Flecha Unidirecional

A dependência e o fluxo de dados transitam sempre em uma direção previsível: da camada de entrada para o domínio e a infraestrutura.

```
[ Camada de Apresentação / Entrada ]
   • Mobile: View -> ViewModel
   • Backend: Route -> Controller
                 │
                 ▼ (DTOs de entrada / parâmetros validados)
        [ Camada de Domínio / Serviços ]
   • Service (Regras de negócio puras, orquestração)
                 │
                 ▼ (Models de domínio)
     [ Camada de Acesso a Dados / Infra ]
   • Repository (Banco de dados, APIs externas, Cache)
```

---

## 2. Regras Universais por Camada

### 2.1. Camada de Apresentação / Entrada
- **Mobile (`ui/`):** As Views renderizam; as ViewModels guardam o estado e delegam lógica para os Services.
- **Backend (`routes/` e `controllers/`):** As rotas mapeiam endpoints HTTP; os Controllers extraem parâmetros, validam DTOs e delegam para os Services. **Controllers nunca contêm regras de negócio**.

### 2.2. Camada de Negócio (`services/`)
- Guardiã exclusiva das regras de negócio.
- **Totalmente agnóstica à infraestrutura:** Não conhece widgets Flutter, nem objetos HTTP (`Request`, `Response` do Express).
- Recebe dados estruturados/DTOs, orquestra Repositories e opera com entidades de domínio (`Models`).

### 2.3. Camada de Acesso a Dados (`repositories/`)
- Responsável pela comunicação com o mundo externo (PostgreSQL, Redis, APIs externas, SQLite).
- É a fronteira que converte dados brutos (JSON, linhas de banco) em **Models limpos** antes de devolver para os Services.

### 2.4. Models vs. DTOs
- **`models/`:** Entidades de domínio ricas e estáveis do Bora (ex: `User`, `VehiclePosition`, `BusLine`).
- **`dto/` ou `dtos/`:** Objetos de transferência de dados que atendem especificamente às interfaces de entrada/saída (ex: `CreateRideDto`, `UpdateLocationBatchDto`).
