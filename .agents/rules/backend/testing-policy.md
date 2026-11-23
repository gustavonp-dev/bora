# Política e Conduta de Testes no Backend

Este documento estabelece a conduta obrigatória para a criação e manutenção de testes automatizados na API **Bora Backend** (Node.js / TypeScript).

---

## 1. Pirâmide de Testes no Backend

```text
tests/ (ou src/**/__tests__/)
├── unit/                       # Testes de Lógica de Negócio Pura (Services)
│   └── services/               # Regras de cálculo, validações, fluxos de negócio
└── integration/                # Testes de Rotas e Endpoints HTTP
    └── routes/                 # Requisições completas com Supertest / mock de banco
```

---

## 2. Metodologias Mandatórias

### 2.1. TDD Obrigatório para Lógica de Negócio (`services/`)
1. **Red:** Escreva o teste unitário que define o comportamento esperado do método da Service. O teste deve falhar.
2. **Green:** Implemente o código mínimo na Service para atender ao contrato e fazer o teste passar.
3. **Refactor:** Otimize e organize o código mantendo o teste verde.
- Repositórios e dependências externas devem ser mockados nos testes unitários.

### 2.2. Testes de Integração para Endpoints (`routes/` e `controllers/`)
- Utilizar bibliotecas padrão (ex: `supertest`) para disparar requisições HTTP contra as rotas do Express.
- Validar:
  - Status codes corretos (`200`, `201`, `400`, `401`, `404`, `500`).
  - Schema de resposta (envelope e tipos de campos esperados).
  - Validação de payload inválido (garantir retorno `400` com detalhes do erro).

---

## 3. Cobertura e Conclusão
- Meta de cobertura mínima de **80%** nas regras de negócio críticas.
- Nenhuma rota ou serviço pode ser entregue sem testes automatizados que garantam seu funcionamento.
