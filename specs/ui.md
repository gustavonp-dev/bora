# Especificação de UI e Arquitetura - Bora

Este documento estabelece as diretrizes de interface, arquitetura de camadas e regras para criação de telas e componentes no aplicativo mobile **Bora**.

---

## 1. Arquitetura de Camadas

A aplicação adota uma organização em camadas bem delimitadas, garantindo baixo acoplamento e um fluxo unidirecional de dados claro:

```text
lib/
├── ui/                       # Camada de apresentação e telas
│   ├── widgets/              # Componentes visuais globais/compartilhados (BoraButton, BoraHeader...)
│   └── <nome_da_tela>/       # Módulo por tela
│       ├── <nome>_view.dart        # Interface da tela (renderização exclusiva)
│       ├── <nome>_view_model.dart  # Gestão de estado da tela e comunicação com serviços
│       └── widgets/                # Componentes exclusivos da tela (somente se necessário)
├── models/                   # Entidades de domínio da aplicação (UserModel, UpcomingBus, Session...)
├── dto/                      # Data Transfer Objects para trânsito de dados com bordas da arquitetura
├── repositories/             # Camada de borda externa (APIs, persistência local, SDKs nativos)
├── services/                 # Regras de negócio agnósticas a UI/rede (processamento, orquestração)
└── commons/                  # Recursos transversais compartilhados
    ├── di/                   # Injeção de dependências (GetIt)
    ├── routes/               # Configuração e guardas de rotas (GoRouter)
    ├── theme/                # Design tokens de tema (Cores, Tipografia, Espaçamentos)
    ├── validators/           # Validadores de formulários
    ├── extensions/           # Extensões auxiliares
    └── types/                # Tipos de dados e resultados utilitários (AuthResult, etc.)
```

### 1.1. Fluxo de Responsabilidades ("A Flecha")

```text
View  ──(eventos/ações)──>  ViewModel  ──(solicitações)──>  Service  ──(DTOs de saída)──>  Repository  ──>  Mundo Externo (API/Storage)
View  <──(notifica estado)── ViewModel <──(Models limpos)── Service <──(Models convertidos)─ Repository <──  Dados crus/JSON
```

1. **View:** Responsável exclusivamente por montar a árvore de widgets e renderizar. Não contém lógica de negócios.
2. **ViewModel:** Gerencia o estado da tela correspondente (MVVM) e encaminha ações do usuário para os `services`.
3. **Service:** Camada central de lógica e regras de negócio. Agnóstica a Flutter e a implementações de rede. Pode receber dados/DTOs, orquestra repositórios e devolve `Models` de domínio para as ViewModels.
4. **Repository:** Camada especialista de comunicação com o mundo externo (APIs, bancos locais, cache). É responsável por enviar DTOs e converter respostas externas brutas em `Models` de domínio para os `services`.
5. **Models vs. DTOs:**
   - `models/`: Entidades de domínio ricas e estáveis do Bora.
   - `dto/`: Objetos de transferência de dados que atendem especificamente às interfaces de entrada/saída externas.

### 1.2. Regras de Telas e Componentização (UI)
- **Uma View e uma ViewModel por tela:** Ficam sempre na mesma pasta dentro de `ui/<nome_da_tela>/`.
- **Componentização modular:** Seções complexas de uma tela devem ser extraídas para `ui/<tela>/widgets/`.
- **Widgets Globais:** Componentes reaproveitados em várias telas (ex: `BoraButton`, `BoraHeader`) pertencem a `ui/widgets/`.

---

## 2. Design Tokens e Sistema de Temas (`lib/commons/theme`)

Todas as telas e componentes **devem** consumir estritamente as definições de tema centralizadas. **Nunca utilize cores hexadecimais, estilos de texto soltos ou espaçamentos arbitrários diretamente no código**.

### 2.1. Cores (`AppColors`)
Arquivo: `lib/commons/theme/app_colors.dart`

- **Brand / Destaque:**
  - `AppColors.primary` (`#FFFF6B00`): Laranja vibrante oficial do Bora. Usado em botões de ação primária, indicadores ativos, ícones de destaque e tags de categoria.
  - `AppColors.secondary` (`#FF1A2B4C`): Azul marinho escuro. Usado em destaques secundários e contraste forte.
- **Tipografia:**
  - `AppColors.textPrimary` (`#FF1A2B4C`): Cor principal para títulos, subtítulos escuros e textos de alta ênfase.
  - `AppColors.textSecondary` (`#FF64718A`): Cinza azulado para descrições, captions, textos de apoio e estados inativos.
- **Superfícies e Fundos:**
  - `AppColors.background` (`#FFF3F5F9`): Fundo padrão de telas.
  - `AppColors.surface` (`#FFFFFFFF`): Fundo de cards, bottom sheets e inputs.
  - `AppColors.border` (`#FFE0E5ED`): Bordas de cards, divisores e inputs desativados.
- **Feedback & Status:**
  - `AppColors.success`, `AppColors.textSuccess`, `AppColors.successSurface`: Confirmações e operações bem-sucedidas.
  - `AppColors.fail`: Erros e validações negativas.

### 2.2. Tipografia (`AppTypography`)
Arquivo: `lib/commons/theme/app_typography.dart`  
*Baseada na família tipográfica **Inter** (via `GoogleFonts.inter`):*

| Token | Tamanho | Peso | Cor Padrão | Uso Recomendado |
| :--- | :--- | :--- | :--- | :--- |
| `AppTypography.headline` | 34px | w800, itálico | `textPrimary` | Títulos de boas-vindas / Hero |
| `AppTypography.title` | 28px | Bold (w700) | `textPrimary` | Título principal de telas |
| `AppTypography.section` | 18px | w700 | `textPrimary` | Cabeçalho de cards e seções |
| `AppTypography.body` | 16px | w400 | `textSecondary` | Parágrafos de leitura e explicações |
| `AppTypography.label` | 14px | w600 | - | Rótulos de botões, tags e links |
| `AppTypography.caption` | 12px | w400 | `textSecondary` | Notas de rodapé, termos legais e dicas |

### 2.3. Espaçamentos e Bordas (`AppSpacing`)
Arquivo: `lib/commons/theme/app_spacing.dart`

- **Métricas de Padding / Gap:**
  - `xs`: 8.0
  - `sm`: 12.0
  - `md`: 16.0
  - `lg`: 20.0
  - `xl`: 24.0
- **Raios de Borda (`BorderRadius`):**
  - `radiusControl`: `Radius.circular(12)` (inputs, botões padrão, chips)
  - `radiusCard`: `Radius.circular(18)` (cards informativos, banners de ilustração)
  - `radiusFull`: `Radius.circular(999)` (badges, pílulas de paginação)


