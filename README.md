# NEWELOG — Repositório principal

> Projeto Integrador 3º Semestre · DSM · FATEC São José dos Campos  
> **Equipe:** Koitech-Log  
> **Cliente:** NEWE Logística Integrada

---

## 📖 Sobre

O **NEWELOG** é um sistema de gestão de frota com motoristas agregados da NEWE Logística Integrada. Ele cobre o cadastro de motoristas e veículos, a importação de manifestos em CSV, o dashboard de rentabilidade e a autenticação de usuários.

O sistema usa **microsserviços**. Cada serviço fica no próprio repositório da organização [Koitech-Log](https://github.com/Koitech-Log). Este repositório reúne todos eles como **git submodules**, então um único clone traz o projeto inteiro.

---

## 🧩 Serviços

| Serviço | Pasta | Repositório | Porta | Responsabilidade |
| :--- | :--- | :--- | :--- | :--- |
| Auth Service | `Backend/auth-service` | [newelog-auth-service](https://github.com/Koitech-Log/newelog-auth-service) | `8082` | Login, emissão de JWT e gestão de usuários |
| Motoristas Service | `Backend/motorista-service` | [newelog-motoristas-service](https://github.com/Koitech-Log/newelog-motoristas-service) | `8080` | Motoristas, veículos, viagens e dashboard |
| Manifesto Service | `Backend/manifesto-service` | [newelog-manifesto-service](https://github.com/Koitech-Log/newelog-manifesto-service) | `8081` | Importação de manifestos CSV, integrada ao motoristas-service |
| Frontend | `frontend` | [newelog-frota-web](https://github.com/Koitech-Log/newelog-frota-web) | `5173` | Interface web (React + TypeScript + Vite) |

Todos os serviços de backend validam o mesmo **JWT (HS256)** emitido pelo `auth-service`. Por isso, o `JWT_SECRET` precisa ser **igual** em todos eles.

---

## 🛠️ Tecnologias

| Tecnologia | Uso |
| :--- | :--- |
| Java 21 + Spring Boot 3.5 | Microsserviços de backend |
| Spring Security + JWT | Autenticação e autorização |
| PostgreSQL 16 + Flyway | Banco de dados e migrations (auth e motoristas) |
| React + TypeScript + Vite | Frontend |
| Docker / Docker Compose | Execução local dos serviços e bancos |
| GitHub Actions | CI (build e testes) |

---

## 🚀 Como começar

```bash
# 1. Clone o repositório já com todos os submodules
git clone --recurse-submodules https://github.com/Koitech-Log/Koitech-Log-3DSM-Newelog.git
cd Koitech-Log-3DSM-Newelog

# Se já clonou sem --recurse-submodules:
git submodule update --init --recursive

# 2. Atualize os submodules para o último commit da main de cada serviço
git submodule update --remote
```

Depois, suba cada serviço seguindo o README dele. Ordem sugerida:

1. **auth-service:** `cd Backend/auth-service && cp .env.example .env && docker compose up --build`
2. **motorista-service:** `cd Backend/motorista-service && cp .env.example .env && docker compose up --build`
3. **manifesto-service:** `cd Backend/manifesto-service && cp .env.example .env && ./mvnw spring-boot:run`
4. **frontend:** `cd frontend && cp .env.example .env && npm install && npm run dev`

Acesse em `http://localhost:5173`.

> ⚠️ Preencha o `JWT_SECRET` com o **mesmo valor** em todos os `.env` de backend (mínimo de 32 caracteres; gere com `openssl rand -base64 48`).

---

## 🌿 Fluxo de trabalho (resumo)

```text
feature/* ──► develop ──► main
```

Cada funcionalidade é desenvolvida em uma branch própria a partir da `develop` e entra nela por Pull Request. Depois de testada, a `develop` é integrada à `main`, que guarda sempre a versão estável. Os detalhes estão em [docs/MODELO-DE-BRANCHES.md](docs/MODELO-DE-BRANCHES.md).
