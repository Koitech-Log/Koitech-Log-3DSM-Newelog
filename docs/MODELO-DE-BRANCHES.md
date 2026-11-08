# 🌿 Modelo de branches

Este documento descreve como a equipe Koitech-Log organiza as branches nos repositórios do NEWELOG. O modelo vale para este repositório principal e para cada microsserviço.

## Visão geral

```text
main      ●───────────────────────────●──────────────►  (versões estáveis / tags)
           \                         /
develop     ●────●──────────●───────●────────────────►  (integração e testes)
                  \        / \     /
feature/login      ●──●───●   \   /
                               \ /
feature/dashboard               ●
```

| Branch | Para que serve | Quem faz commit direto |
| :--- | :--- | :--- |
| `main` | Versão **estável**, testada e pronta para apresentação ou entrega. Cada versão recebe uma tag (`v0.1.0`, `v0.2.0`, ...). | Ninguém. Só recebe merge vindo da `develop`. |
| `develop` | Branch de **integração**. Junta todas as funcionalidades concluídas para testar o sistema completo antes de ir para a `main`. | Ninguém. Só recebe merge das branches de funcionalidade. |
| `feature/<nome>` | Uma branch **por funcionalidade**, criada a partir da `develop`. | O(s) desenvolvedor(es) da funcionalidade. |

## Passo a passo

### 1. Começar uma funcionalidade

Sempre parta da `develop` atualizada:

```bash
git checkout develop
git pull origin develop
git checkout -b feature/nome-da-funcionalidade
```

### 2. Desenvolver

Faça commits pequenos e com mensagens claras (veja o [padrão de commits](#padrão-de-commits)):

```bash
git add .
git commit -m "feat: adiciona tela de cadastro de veículos"
git push -u origin feature/nome-da-funcionalidade
```

### 3. Abrir Pull Request para a `develop`

No GitHub, abra um PR de `feature/nome-da-funcionalidade` → `develop`. Antes do merge:

- o CI (build e testes) precisa passar;
- pelo menos um colega revisa o código;
- os conflitos são resolvidos na própria branch de funcionalidade.

Depois do merge, a branch de funcionalidade pode ser apagada.

### 4. Testar na `develop`

Com as funcionalidades integradas, a equipe testa o sistema completo a partir da `develop`. Bugs encontrados nessa etapa são corrigidos em uma branch `fix/<nome>` criada a partir da `develop`, que volta para ela por PR.

### 5. Levar para a `main`

Quando a `develop` está estável (por exemplo, ao fim de uma sprint), abra um PR `develop` → `main`. Depois do merge, crie a tag da nova versão:

```bash
git checkout main
git pull origin main
git tag -a v0.2.0 -m "Sprint 2: descrição da entrega"
git push origin v0.2.0
```

## Nomes de branch

| Prefixo | Uso | Exemplo |
| :--- | :--- | :--- |
| `feature/` | Nova funcionalidade | `feature/importacao-manifesto` |
| `fix/` | Correção de bug | `fix/login-token-expirado` |
| `docs/` | Apenas documentação | `docs/readme-auth-service` |
| `chore/` | Configuração, dependências, submodules | `chore/atualiza-submodules` |

Use letras minúsculas, sem acentos, com palavras separadas por hífen. Se houver card no quadro da sprint, o código pode entrar no nome: `feature/KOI-144-lista-veiculos`.

## Padrão de commits

As mensagens seguem o formato `tipo: descrição curta no imperativo`:

| Tipo | Quando usar |
| :--- | :--- |
| `feat` | Nova funcionalidade |
| `fix` | Correção de bug |
| `docs` | Documentação |
| `refactor` | Mudança de código sem alterar comportamento |
| `test` | Testes |
| `chore` | Configuração, dependências, build |

## Submodules neste repositório

Este repositório aponta para um commit específico de cada microsserviço. Para trazer as versões mais recentes da `main` de cada serviço, siga o mesmo fluxo: crie uma branch a partir da `develop`, atualize e abra PR para a `develop`.

```bash
git checkout develop && git pull
git checkout -b chore/atualiza-submodules
git submodule update --remote
git add Backend frontend
git commit -m "chore: atualiza submodules dos microsserviços"
git push -u origin chore/atualiza-submodules
```

## Resumo

1. **Nunca** faça commit direto na `main` nem na `develop`.
2. Cada funcionalidade tem sua própria branch, criada a partir da `develop`.
3. A funcionalidade entra na `develop` por Pull Request revisado.
4. A `develop` é testada e só então vai para a `main`.
5. Cada entrega na `main` recebe uma tag de versão.
