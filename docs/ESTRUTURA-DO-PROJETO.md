# 📂 Estrutura do projeto

Este documento explica como o repositório principal do NEWELOG está organizado e o papel de cada parte.

## Árvore de pastas

```text
Koitech-Log-3DSM-Newelog/
├── Backend/
│   ├── auth-service/          # submodule → newelog-auth-service
│   ├── manifesto-service/     # submodule → newelog-manifesto-service
│   └── motorista-service/     # submodule → newelog-motoristas-service
├── frontend/                  # submodule → newelog-frota-web
├── docs/                      # documentação geral do projeto
│   ├── ESTRUTURA-DO-PROJETO.md
│   └── MODELO-DE-BRANCHES.md
├── src/                       # código do template base de microsserviço (Spring Boot)
├── .github/workflows/ci.yml   # pipeline de CI (build + testes com Maven)
├── .mvn/, mvnw, mvnw.cmd      # Maven Wrapper
├── pom.xml                    # projeto Maven do template
├── Dockerfile                 # imagem Docker do template (build multi-stage)
├── .gitmodules                # mapeamento dos submodules
└── README.md
```

## Repositório principal + submodules

Cada microsserviço e o frontend têm **repositório próprio**, com histórico, CI e Pull Requests independentes. Este repositório apenas os referencia como **git submodules**.

Um submodule guarda o **commit exato** do serviço que está em uso. Por isso:

- clonar com `--recurse-submodules` traz todos os serviços de uma vez;
- uma alteração dentro de `Backend/auth-service` é commitada e enviada **no repositório do auth-service**, não aqui;
- para registrar aqui uma versão mais nova de um serviço, rode `git submodule update --remote` e faça commit do novo ponteiro (veja o [modelo de branches](MODELO-DE-BRANCHES.md#submodules-neste-repositório)).

Todos os submodules acompanham a branch `main` do seu repositório (`branch = main` no `.gitmodules`).

## Arquitetura

```text
                         ┌──────────────────────┐
                         │      Frontend        │
                         │  React + Vite :5173  │
                         └──────────┬───────────┘
              login / JWT           │            requisições com Bearer JWT
        ┌───────────────────────────┼──────────────────────────────┐
        ▼                           ▼                              ▼
┌───────────────┐        ┌────────────────────┐        ┌────────────────────┐
│ auth-service  │        │ motorista-service  │◄───────│ manifesto-service  │
│     :8082     │        │       :8080        │  REST  │       :8081        │
└───────┬───────┘        └─────────┬──────────┘        └────────────────────┘
        ▼                          ▼                     (sem banco próprio)
   PostgreSQL                 PostgreSQL
   (auth_db)               (motoristas_db)
```

- O **auth-service** emite o JWT no login. Os demais serviços validam o token com o mesmo `JWT_SECRET`.
- O **manifesto-service** recebe o CSV, extrai os registros e chama o **motorista-service** para validar e cadastrar os motoristas, repassando o token do usuário.
- Cada serviço com persistência tem **banco próprio** (database per service), com migrations versionadas pelo Flyway.

## Serviços

### `Backend/auth-service`: autenticação

- **Stack:** Java 21, Spring Boot, Spring Security, JPA, Flyway, PostgreSQL 16, jjwt.
- **Porta:** `8082` (Postgres em `5433` no host).
- **Rotas principais:** `POST /api/auth/login`, `GET /api/auth/me`, `POST /api/auth/alterar-senha`, `/api/usuarios` (gestão de usuários).
- **Pacotes:** `br.com.newelog.auth` → `config`, `controller`, `dto`, `model`, `repository`, `security`, `service`.
- No primeiro start, cria o gestor inicial a partir de `ADMIN_EMAIL` / `ADMIN_PASSWORD`.

### `Backend/motorista-service`: motoristas e frota

- **Stack:** Java 21, Spring Boot, Spring Security (resource server JWT), JPA, Flyway, PostgreSQL 16.
- **Porta:** `8080` (Postgres em `5432`).
- **Rotas principais:** `/api/motoristas`, `/api/veiculos`, `/api/dashboard`, `/api/dashboard/rentabilidade-media`, além das rotas de integração com o manifesto (`validar-manifesto`, `cadastrar-de-manifesto`, `cadastrar-de-csv`).
- **Pacotes:** `br.com.newelog.motoristas` → `config`, `controller`, `dto`, `model`, `repository`, `service`.

### `Backend/manifesto-service`: importação de manifestos

- **Stack:** Java 21, Spring Boot, Spring Security (resource server JWT).
- **Porta:** `8081`. Não tem banco: toda a persistência fica no motorista-service.
- **Rota principal:** `POST /manifestos/upload` (arquivo CSV).
- **Pacotes:** `br.com.newelog` → `config`, `controller`, `dto`, `service`, `validation`.
- Depende da variável `MOTORISTAS_SERVICE_URL`.

### `frontend`: interface web

- **Stack:** React, TypeScript, Vite, oxlint.
- **Porta:** `5173`.
- **Pastas em `src/`:** `api` (chamadas HTTP), `components`, `context`, `hooks`, `pages`, `types`, `utils`, `assets`.
- **Variáveis:** `VITE_API_BASE_URL` (motoristas), `VITE_MANIFESTO_SERVICE_URL`, `VITE_AUTH_SERVICE_URL`.

## Template de microsserviço (raiz)

Os arquivos na raiz (`src/`, `pom.xml`, `Dockerfile`, `.mvn/`, `mvnw`, `.github/workflows/ci.yml`) formam um **template base** de microsserviço Spring Boot (`newelog-service-template`), que serve de ponto de partida para novos serviços. Ele já traz:

- estrutura Maven com Spring Web MVC e Actuator;
- `Dockerfile` multi-stage (build com JDK 21, execução com JRE 21);
- workflow de CI no GitHub Actions que roda `./mvnw clean verify` a cada push e PR para a `main`.

Para criar um novo microsserviço:

1. Crie o repositório `newelog-<nome>-service` na organização a partir do template.
2. Adicione-o aqui como submodule:
   ```bash
   git submodule add -b main https://github.com/Koitech-Log/newelog-<nome>-service Backend/<nome>-service
   ```
3. Atualize a tabela de serviços do [README](../README.md) e este documento.

## Configuração e segredos

- Cada serviço tem um `.env.example` documentando as variáveis. Copie para `.env` e preencha. O `.env` **nunca** é versionado.
- O `JWT_SECRET` precisa ser o **mesmo** no auth, motorista e manifesto service.
- Os detalhes de execução de cada serviço estão no README do próprio serviço.
