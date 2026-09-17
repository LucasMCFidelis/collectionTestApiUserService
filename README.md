# 👤 collectionTestApiUserService

Repositório de suporte a testes do **UserService** do projeto *Catálogo de Eventos*. Reúne:

1. A **collection do Postman** com os testes de integração do próprio UserService (cadastro, edição, busca, recuperação de senha, permissões, exclusão).
2. Os **mappings do WireMock** que simulam o UserService, reaproveitados como dependência mockada pelas suítes de teste do [`auth-service`](https://github.com/LucasMCFidelis/auth-service-eventsCatalog-) e do [`email-service`](https://github.com/LucasMCFidelis/email-service-eventsCatalog-).

> ⚠️ **Sem *environments* ainda**: diferente das outras collections do projeto, este repositório ainda não tem arquivos de *environment* (`local-mock`, `ci`, etc.) versionados. As variáveis precisam ser criadas manualmente no Postman — veja a seção [Variáveis](#️-variáveis-necessárias-para-executar-a-collection) abaixo.

---

## 📁 Estrutura

```
collectionTestApiUserService/
├── postman/
│   └── collections/
│       └── user-service.postman_collection
└── wiremock/
    └── mappings/
        ├── 00-fallback.json
        ├── 01-validate-success-user.json
        ├── 02-validate-success-admin.json
        ├── 03-invalid-credentials.json
        ├── 04-user-not-found.json
        ├── 05-invalid-email.json
        └── 06-get-to-email-success-user.json
```

---

## ▶️ Como executar

### Opção 1 — Postman (interface gráfica)
1. Importe a collection em `postman/collections/user-service.postman_collection`.
2. Crie um Environment no Postman e cadastre as variáveis da seção abaixo (não há arquivo de environment pronto para importar ainda).
3. Execute a collection inteira via **Runner**, ou cada request individualmente.

### Opção 2 — Newman (linha de comando / CI)
```bash
npm install -g newman

newman run postman/collections/user-service.postman_collection \
  --env-var "user_service_url=http://localhost:8089/users" \
  --env-var "auth_service_url=http://localhost:3232/auth" \
  --env-var "email_service_url=http://localhost:3000/emails" \
  --env-var "useMock=true" \
  --env-var "adminEmail=admin@teste.com" \
  --env-var "adminPassword=SuaSenhaAdmin@123"
```

---

## ⚙️ Variáveis necessárias para executar a Collection

### 🌐 URLs dos serviços obrigatórios

Como o UserService participa dos fluxos de login e recuperação de senha, a collection depende também do **AuthService** e do **EmailService** reais (ou de instâncias locais deles) para completar alguns cenários — não é um teste isolado do UserService sozinho.

| Variável            | Descrição                                              |
|---------------------|-----------------------------------------------------------|
| `user_service_url`  | URL base do UserService sendo testado                       |
| `auth_service_url`  | URL base do AuthService, usado para obter tokens de login (`performLogin`) |
| `email_service_url` | URL base do EmailService, usado para gerar códigos de recuperação (`sendRecoveryCodeRequest`) |

### 🎭 Modo de execução (mock x real)

| Variável  | Descrição |
|-----------|---|
| `useMock` | `"true"` pula a criação/remoção real de usuário (usa dados fixos) e envia `x-mock-scenario` nas chamadas de login; `"false"` roda a integração real ponta a ponta. |

### 👤 Credenciais

| Variável        | Descrição                 |
|-----------------|----------------------------|
| `adminEmail`    | Email de um usuário com permissão de Admin, usado nos testes de atualização de permissão |
| `adminPassword` | Senha desse administrador |
| `emailDefaultToRecoveryPassword` | Email padrão para testes de atualização de senha |

### 🧪 Variáveis de fluxo (preenchidas automaticamente)

Criadas e atualizadas pelos scripts da collection a partir das funções auxiliares. Não é necessário preenchê-las manualmente.

| Variável                     | Descrição                     |
|------------------------------|----------------------------------|
| `userRecentCadastreId`       | ID do usuário de teste criado             |
| `userRecentCadastreEmail`    | Email do usuário de teste criado          |
| `userRecentCadastrePassword` | Senha usada na criação           |
| `userRecentCadastreToken`    | Token JWT do usuário de teste criado      |
| `adminToken`                 | Token JWT do admin, obtido via `performLogin` |
| `conflictEmail`              | E-mail de um segundo usuário criado só para testar conflito de e-mail duplicado |
| `invalidToken`               | Token inválido fixo (`"invalid"`), usado no teste de token inválido |
| `emailToRecoveryCode`        | E-mail usado no fluxo de recuperação de senha |
| `recoveryCode`               | Código de recuperação mais recente gerado pelo EmailService |
| `recoveryCodeToExpired`      | Código de recuperação antigo, mantido de propósito para simular expiração |
| `newPassword`                | Nova senha usada nos testes de atualização de senha |

---

## 🧪 Testes da collection

Todos os requests validam o status HTTP retornado e o conteúdo da resposta (corpo, `message` de erro ou dados do usuário).

### 📂 Cadastro Success (`POST {{user_service_url}}`)
| Cenário | Retorno esperado |
|---|---|
| Cadastro de usuário com todos os campos | `201` |
| Cadastro de usuário com campos obrigatórios | `201` |

### 📂 Cadastro campo: nome / sobrenome
6 cenários cada, cobrindo: vazio, ausente, caracteres inválidos, tamanho menor que o esperado, tipo inválido e preenchido só com espaços. Todos retornam `400`.

### 📂 Cadastro campo: email
| Cenário | Retorno esperado |
|---|---|
| Sem passar o email / email vazio / tipo inválido / apenas espaços | `400` |
| Email já cadastrado anteriormente | `409` |

### 📂 Cadastro campo: telefone
| Cenário | Retorno esperado |
|---|---|
| Tipo inválido / menor que o esperado / maior que o limite | `400` |
| Sem passar o telefone | `201` *(telefone é opcional)* |

### 📂 Cadastro campo: senha
6 cenários: vazia, ausente, menor que o esperado, sem letra maiúscula, sem número, sem caractere especial. Todos retornam `400`.

### 📂 Atualizar senha (`PATCH {{user_service_url}}/recuperacao/atualizar-senha`)
| Cenário | O que valida | Retorno esperado |
|---|---|---|
| Alterar senha de um usuário válido | Gera um código de recuperação real via EmailService e usa para trocar a senha | `200` |
| Alterar senha com senha fraca | Nova senha não atende aos requisitos | `400` |
| Alterar senha com código expirado | Usa um código de recuperação antigo, já substituído por um mais novo | `400` |
| Alterar senha com código de recuperação incorreto | Código aleatório inválido | `400` |
| Alterar senha com email inválido | Formato de e-mail inválido | `400` |
| Alterar senha sem passar `recoveryCode` | Campo obrigatório ausente | `400` |

### 📂 Editar usuário (`PUT {{user_service_url}}?userId=...`)
| Cenário | Retorno esperado |
|---|---|
| Editar dados básicos do usuário válido | `200` |
| Editar com e-mail já cadastrado por outro usuário | `409` |
| Editar sem informar id | `400` |
| Editar com id inválido | `400` |

### 📂 Buscar usuário (`GET {{user_service_url}}`)
| Cenário | Retorno esperado |
|---|---|
| Buscar por ID válido | `200` |
| Buscar por Email válido | `200` |
| Buscar por ID inválido (formato) | `400` |
| Buscar usuário inexistente (ID válido mas não cadastrado) | `404` |
| Buscar sem informar ID | `400` |

### 📂 Validação das credenciais de usuário (`POST {{user_service_url}}/validate-credentials`)
| Cenário | Retorno esperado |
|---|---|
| Email inválido | `400` |
| Credenciais corretas | `200` |
| Email não cadastrado | `404` |
| Credenciais incorretas (senha errada) | `401` |

### 📂 Atualizar permissão de usuário (`PUT {{user_service_url}}/update-role-user?userId=...`)
| Cenário | Retorno esperado |
|---|---|
| Atualizar permissão (admin autenticado) | `200` |
| Atualizar para a mesma permissão que o usuário já tem | `409` |
| Token inválido | `401` |
| Token sem permissão de administrador | `403` |
| Permissão de destino inválida | `400` |

### 📂 Deletar usuário (`DELETE {{user_service_url}}?userId=...`)
| Cenário | Retorno esperado |
|---|---|
| Deletar usuário existente (usando o próprio token) | `200` |
| Deletar com id diferente do usuário logado | `403` |
| Deletar sem informar ID | `400` |

---

## 🛠️ Funções auxiliares (scripts de collection)

Definidas no pre-request script da collection e compartilhadas entre requests via variáveis de collection:

- **`createSimpleUserRequest(attempt)`** — cria um usuário comum no `user_service_url` com dados aleatórios, até 5 tentativas em caso de falha. Em modo mock, simula a criação com dados fixos. Preenche `userRecentCadastreId/Email/Password/Token`.
- **`deleteSimpleUserRequest(token, userId)`** — remove o usuário de teste e limpa as variáveis correspondentes. Ignorado em modo mock.
- **`performLogin(email, password, scenario, tokenVarName = "adminToken")`** — loga no `auth_service_url` e salva o token na variável indicada. Em modo mock, envia `x-mock-scenario`.
- **`sendRecoveryCodeRequest(email, scenario)`** — dispara o envio de um código de recuperação via `email_service_url`, guardando o e-mail em `emailToRecoveryCode` e o código em `recoveryCode` quando exposto na resposta.

---

## 🧩 Mocks do UserService (WireMock)

Além da collection própria, este repositório mantém os mappings do WireMock consumidos pelos pipelines de CI do `auth-service` e do `email-service`, para simular o UserService como dependência externa sem depender do serviço real.

| Arquivo | Endpoint simulado | `X-Mock-Scenario` | Resposta |
|---|---|---|---|
| `00-fallback.json` | `POST /users/validate-credentials` | *(qualquer, sem cenário reconhecido)* | `500` — erro genérico avisando que o cenário não foi definido |
| `01-validate-success-user.json` | `POST /users/validate-credentials` | `SUCCESS_VALIDATE_USER` | `200` — retorna `userId`, `userEmail` e `roleName: "User"` |
| `02-validate-success-admin.json` | `POST /users/validate-credentials` | `SUCCESS_VALIDATE_ADMIN` | `200` — retorna `userId`, `userEmail` e `roleName: "Admin"` |
| `03-invalid-credentials.json` | `POST /users/validate-credentials` | `INVALID_CREDENTIALS` | `401` — "Credenciais inválidas" |
| `04-user-not-found.json` | `ANY /users*` | `USER_NOT_FOUND` | `404` — "Usuário não encontrado" |
| `05-invalid-email.json` | `ANY /users/*` | `INVALID_EMAIL` | `400` — "email deve ser um email válido" |
| `06-get-to-email-success-user.json` | `GET /users?userEmail=...` | `SUCCESS_GET_USER` | `200` — retorna um usuário mockado, ecoando o `userEmail` da query na resposta (via `response-template`) |

A prioridade (`priority`) de cada mapping evita conflito entre regras mais genéricas (`ANY /users*`) e as mais específicas por método/endpoint — o fallback (`00`) tem a prioridade mais baixa e só responde quando nenhum outro mapping casa com a requisição.

### Subindo o mock isoladamente

```bash
docker run -d --name wiremock-user-service -p 8089:8080 \
  -v "$(pwd)/wiremock:/home/wiremock" \
  wiremock/wiremock
```

Isso expõe o mock em `http://localhost:8089`. Depois, basta apontar a variável de URL do UserService do serviço sendo testado para `http://localhost:8089` e enviar o header `X-Mock-Scenario` desejado.

---

## 🚧 Próximos passos

- [ ] Criar arquivos de *environment* (`local-mock`, `local-real`, `ci`, `production-like`) para a nova collection, no padrão das demais.