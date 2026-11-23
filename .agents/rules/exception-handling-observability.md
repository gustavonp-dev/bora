# Diretriz de Tratamento de Exceções e Observabilidade no Mobile

Este documento estabelece o padrão arquitetural e obrigatório para tratamento de erros, resiliência e registro de telemetria no aplicativo **Bora**.

---

## 1. Contexto e Motivação

No desenvolvimento mobile, dois antipadrões comuns comprometem gravemente a estabilidade e a capacidade de diagnóstico do app:

1. **Engolir exceções em silêncio (*Silent Swallow*):** Usar blocos como `catch (_) {}` esconde erros de rede, parsing ou integrações. O app falha de forma fantasma e os desenvolvedores ficam sem dados para reproduzir o problema.
2. **Uso de `print()` / `debugPrint()` solto:** Apenas despeja strings planas no terminal. Não permite filtragem, não estrutura stacktraces e polui a saída de console sem integração com as ferramentas oficiais de diagnóstico.

Para solucionar isso de ponta a ponta, adotamos o padrão oficial de observabilidade do Dart via `dart:developer`, que se conecta diretamente à aba de **Logging do Flutter DevTools**.

---

## 2. A Regra de Ouro: Duas Categorias de Operações

Toda captura de exceção no app deve ser classificada estritamente em uma de duas naturezas:

### A. Operações Bloqueantes (Ações Diretas do Usuário)
São tarefas intencionais onde a falha impede diretamente o objetivo do usuário.

- **Exemplos:** Login, finalização de compra de passagem, envio de formulário de cadastro, recarga de saldo.
- **Conduta Obrigatória:**
  1. A exceção deve interromper o fluxo da ViewModel.
  2. O usuário **deve receber feedback visual explícito e amigável** (`BoraSnackbar.showError`, banner na tela ou mensagem contextual no formulário).
  3. Se a falha for um erro inesperado de sistema (500, timeout, crash de parsing), ela também deve ser registrada via `developer.log(...)`.

### B. Operações Não Bloqueantes (Background e Degradação Graciosa)
São rotinas auxiliares ou secundárias onde uma falha técnica temporária não pode travar nem interromper a experiência principal do usuário.

- **Exemplos:**
  - Busca automática de endereço por CEP: se o serviço cair, o usuário continua digitando manualmente (degradação graciosa).
  - Revogação de token Push no Logout: a prioridade é deslogar localmente. Se o request na nuvem falhar, o logout continua.
  - Telemetria e analytics em background.
- **Conduta Obrigatória:**
  1. O app **não exibe mensagens de erro alarmantes** para o usuário.
  2. É **terminantemente obrigatório** registrar a ocorrência no DevTools com contexto, erro e stacktrace estruturados via `developer.log(...)`.

---

## 3. O Padrão Estruturado com `dart:developer`

Toda captura com log deve seguir este formato padrão:

```dart
import 'dart:developer' as developer;

try {
  // Chamada de serviço, parsing ou operação em background
  return await servico.executar();
} catch (error, stackTrace) {
  developer.log(
    'Descrição clara do que falhou e o contexto (ex: Falha ao consultar CEP 01001-000)',
    name: 'NomeDoModuloOuServico',
    error: error,
    stackTrace: stackTrace,
  );
  // Retorno seguro ou comportamento de fallback
  return null;
}
```

### Por que usar os parâmetros `name`, `error` e `stackTrace`?

| Parâmetro | Benefício no Flutter DevTools |
| :--- | :--- |
| `message` | Mensagem textual descritiva explicando o contexto da falha em linguagem clara. |
| `name:` | Cria uma tag de filtro. No DevTools, você filtra digitando apenas `name:AuthService` ou `name:LocationService`, isolando o problema em segundos. |
| `error:` | Mantém a tipagem do objeto de exceção original (`SocketException`, `FormatException`, `HttpException`), permitindo inspecionar propriedades detalhadas no inspetor. |
| `stackTrace:` | Habilita a árvore de rastreamento clicável no DevTools, permitindo navegar direto para a linha exata do código onde a exceção nasceu. |

---

## 4. Resumo Prático para Code Reviews

- ❌ **Reprovado:** `catch (_) {}` (engolir erro em silêncio).
- ❌ **Reprovado:** `catch (e) { print(e); }` (texto plano solto).
- ✅ **Aprovado (Ação direta):** Trata erro, exibe feedback visual (`BoraSnackbar.showError`) e registra no log se for falha de sistema.
- ✅ **Aprovado (Ação secundária):** Aplica degradação graciosa (ex: `return null;` ou continua o fluxo) e grava obrigatoriamente `developer.log(..., name: ..., error: e, stackTrace: s)`.
