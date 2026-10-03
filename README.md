# 👤 collectionTestApiUserService

Collection do Postman para testes de integração (end-to-end) do **UserService** do projeto *Catálogo de Eventos*. A collection cobre os fluxos de **cadastro**, **edição**, **busca**, **recuperação de senha**, **permissões**, **exclusão** e **favoritos**, incluindo cenários de sucesso, erro de validação e erros de permissão.

---

## ▶️ Como executar

Essa collection é usada de duas formas: **automaticamente**, dentro do `docker compose` do serviço testado, ou **manualmente**, via Postman/Newman, para depurar um cenário específico. Veja a seção [🌎 Ambientes disponíveis](#-ambientes-disponíveis) para saber qual environment usar em cada caso.

Para rodar manualmente — seja pelo Postman, seja pelo Newman — primeiro suba o ambiente de teste do UserService via `docker compose`, no repositório do serviço, [`user-service-eventsCatalog`](https://github.com/LucasMCFidelis/user-service-eventsCatalog) — é lá que estão as instruções detalhadas de setup, profiles e variáveis de ambiente:

```bash
docker compose --profile test up --build -d
```

Isso builda este repositório internamente e já roda a collection automaticamente (contra `ci.environment.json`), mas mantém o UserService e os mocks do AuthService e EmailService publicados nas portas padrão do host (`8081`/`8082`/`8083`) enquanto os containers estiverem de pé — é contra essas portas que o environment `local` aponta, usado abaixo em ambas as opções. Ao terminar, encerre o ambiente com `docker compose --profile test down -v` no repositório do UserService.

Clone este repositório também — é dele que vêm a collection e os environments usados nas duas opções abaixo:

```bash
git clone https://github.com/LucasMCFidelis/collectionTestApiUserService.git
cd collectionTestApiUserService
```

### Opção 1 — Postman (interface gráfica)
1. Importe a collection em `postman/collections/user-service.postman_collection.json`.
2. Importe o(s) environment(s) desejado(s) em `postman/environments/` (`local.environment.json` e/ou `ci.environment.json`).
3. Selecione o environment no canto superior direito do Postman.
4. Preencha as variáveis obrigatórias (ver seção [Variáveis](#️-variáveis-necessárias-para-executar-a-collection) abaixo) — os dois environments já vêm preenchidos por padrão, só ajuste se necessário.
5. Execute a collection inteira via **Runner**, ou cada request individualmente.

### Opção 2 — Newman (linha de comando)

Com o `docker compose` já rodando e o repositório clonado, aponte o Newman direto para o environment `local`, sem precisar sobrescrever nenhuma URL:

```bash
npm install -g newman

newman run postman/collections/user-service.postman_collection.json \
  -e postman/environments/local.environment.json
```

---

## 🌎 Ambientes disponíveis

Só existem **dois** arquivos de environment neste repositório — mantidos deliberadamente enxutos, um para cada forma de execução:

| Ambiente  | Quando usar                                                                                                                                                                                                                                                                                                                                                        |
| --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Local** | Rodar a collection manualmente (Postman ou Newman), na sua máquina, contra o ambiente de teste do UserService já rodando localmente via `docker compose` (porta padrão do host — ver seção abaixo). Ideal para depurar um cenário específico sem esperar o CI.                                                                                                     |
| **CI**    | Uso interno, automático: é o environment que o próprio `docker compose` do UserService injeta no container de testes. Os hostnames (`user-service`, `auth-service`, `email-service`) são os *aliases* de rede dos serviços dentro do Compose, não `localhost` — porta interna sempre `8080`. Normalmente você não precisa selecionar esse environment manualmente. |

Os dois já vêm com `useMock="true"` e `adminEmail`/`adminPassword`/`emailDefaultToRecoveryPassword` preenchidos com credenciais fixas de mock — não é preciso configurar nada para rodar contra o ambiente de teste (mockado) do UserService, seja localmente, seja no CI.

---

## ⚙️ Variáveis necessárias para executar a Collection
Para que esta collection funcione corretamente no Postman, configure as seguintes variáveis no **Environment**:


### 🌐 URLs dos serviços obrigatórios

| Variável            | Descrição                                                                                            |
| ------------------- | ---------------------------------------------------------------------------------------------------- |
| `base_url`          | URL base do UserService sendo testado — sempre uma instância real, em qualquer um dos environments   |
| `user_service_url`  | URL base do UserService complementado com o path das rotas de usuários (rotas principais do serviço) |
| `auth_service_url`  | URL base do AuthService, usado para obter tokens de login (`performLogin`)                           |
| `email_service_url` | URL base do EmailService, usado para gerar códigos de recuperação (`sendRecoveryCodeRequest`)        |

### 🎭 Modo de execução (mock x real)

| Variável  | Descrição                                                                                                                                                                                                                                                                                                                                                                                 |
| --------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `useMock` | `"true"`: a criação do usuário de teste continua acontecendo de verdade no UserService (só ganha o header `x-mock-scenario`), mas a remoção ao final é pulada; as chamadas a `auth_service_url`/`email_service_url` enviam `x-mock-scenario` para acionar os respectivos mocks. `"false"`: roda a integração real ponta a ponta, sem headers de mock, e remove o usuário criado ao final. |

### 👤 Credenciais e e-mails fixos

| Variável                         | Descrição                                                                                                                                                                                        |
| -------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `adminEmail`                     | Email de um usuário com permissão de Admin, usado nos testes de atualização de permissão                                                                                                         |
| `adminPassword`                  | Senha desse administrador                                                                                                                                                                        |
| `emailDefaultToRecoveryPassword` | E-mail usado pelos testes da pasta **Atualizar senha** para gerar o código de recuperação via EmailService — parametrizado via environment (deixou de ser um e-mail fixo no corpo dos requests). |

### 🧪 Variáveis de fluxo (preenchidas automaticamente)

Criadas e atualizadas pelos scripts da collection a partir das funções auxiliares. Não é necessário preenchê-las manualmente.

| Variável                     | Descrição                                                                       |
| ---------------------------- | ------------------------------------------------------------------------------- |
| `userRecentCadastreId`       | ID do usuário de teste criado                                                   |
| `userRecentCadastreEmail`    | Email do usuário de teste criado                                                |
| `userRecentCadastrePassword` | Senha usada na criação                                                          |
| `userRecentCadastreToken`    | Token JWT do usuário de teste criado                                            |
| `adminToken`                 | Token JWT do admin, obtido via `performLogin`                                   |
| `conflictEmail`              | E-mail de um segundo usuário criado só para testar conflito de e-mail duplicado |
| `invalidToken`               | Token inválido fixo (`"invalid"`), usado no teste de token inválido             |
| `emailToRecoveryCode`        | E-mail usado no fluxo de recuperação de senha                                   |
| `recoveryCode`               | Código de recuperação mais recente gerado pelo EmailService                     |
| `recoveryCodeToExpired`      | Código de recuperação antigo, mantido de propósito para simular expiração       |
| `newPassword`                | Nova senha usada nos testes de atualização de senha                             |

---

## 🧪 Testes da collection

Todos os requests validam o status HTTP retornado e o conteúdo da resposta (corpo, `message` de erro ou dados do usuário).

### 📂 Cadastro Success (`POST {{user_service_url}}`)
| Cenário                                     | Retorno esperado |
| ------------------------------------------- | ---------------- |
| Cadastro de usuário com todos os campos     | `201`            |
| Cadastro de usuário com campos obrigatórios | `201`            |

### 📂 Cadastro campo: nome / sobrenome
6 cenários cada, cobrindo: vazio, ausente, caracteres inválidos, tamanho menor que o esperado, tipo inválido e preenchido só com espaços. Todos retornam `400`.

### 📂 Cadastro campo: email
| Cenário                                                           | Retorno esperado |
| ----------------------------------------------------------------- | ---------------- |
| Sem passar o email / email vazio / tipo inválido / apenas espaços | `400`            |
| Email já cadastrado anteriormente                                 | `409`            |

### 📂 Cadastro campo: telefone
| Cenário                                                   | Retorno esperado              |
| --------------------------------------------------------- | ----------------------------- |
| Tipo inválido / menor que o esperado / maior que o limite | `400`                         |
| Sem passar o telefone                                     | `201` *(telefone é opcional)* |

### 📂 Cadastro campo: senha
6 cenários: vazia, ausente, menor que o esperado, sem letra maiúscula, sem número, sem caractere especial. Todos retornam `400`.

### 📂 Atualizar senha (`PATCH {{user_service_url}}/recuperacao/atualizar-senha`)
| Cenário                                           | O que valida                                                                  | Retorno esperado |
| ------------------------------------------------- | ----------------------------------------------------------------------------- | ---------------- |
| Alterar senha de um usuário válido                | Gera um código de recuperação real via EmailService e usa para trocar a senha | `200`            |
| Alterar senha com senha fraca                     | Nova senha não atende aos requisitos                                          | `400`            |
| Alterar senha com código expirado                 | Usa um código de recuperação antigo, já substituído por um mais novo          | `400`            |
| Alterar senha com código de recuperação incorreto | Código aleatório inválido                                                     | `400`            |
| Alterar senha com email inválido                  | Formato de e-mail inválido                                                    | `400`            |
| Alterar senha sem passar `recoveryCode`           | Campo obrigatório ausente                                                     | `400`            |

### 📂 Editar usuário (`PUT {{user_service_url}}?userId=...`)
| Cenário                                           | Retorno esperado |
| ------------------------------------------------- | ---------------- |
| Editar dados básicos do usuário válido            | `200`            |
| Editar com e-mail já cadastrado por outro usuário | `409`            |
| Editar sem informar id                            | `400`            |
| Editar com id inválido                            | `400`            |

### 📂 Buscar usuário (`GET {{user_service_url}}`)
| Cenário                                                   | Retorno esperado |
| --------------------------------------------------------- | ---------------- |
| Buscar por ID válido                                      | `200`            |
| Buscar por Email válido                                   | `200`            |
| Buscar por ID inválido (formato)                          | `400`            |
| Buscar usuário inexistente (ID válido mas não cadastrado) | `404`            |
| Buscar sem informar ID                                    | `400`            |

### 📂 Validação das credenciais de usuário (`POST {{user_service_url}}/validate-credentials`)
| Cenário                               | Retorno esperado |
| ------------------------------------- | ---------------- |
| Email inválido                        | `400`            |
| Credenciais corretas                  | `200`            |
| Email não cadastrado                  | `404`            |
| Credenciais incorretas (senha errada) | `401`            |

### 📂 Atualizar permissão de usuário (`PUT {{user_service_url}}/update-role-user?userId=...`)
| Cenário                                               | Retorno esperado |
| ----------------------------------------------------- | ---------------- |
| Atualizar permissão (admin autenticado)               | `200`            |
| Atualizar para a mesma permissão que o usuário já tem | `409`            |
| Token inválido                                        | `401`            |
| Token sem permissão de administrador                  | `403`            |
| Permissão de destino inválida                         | `400`            |

### 📂 Deletar usuário (`DELETE {{user_service_url}}?userId=...`)
| Cenário                                            | Retorno esperado |
| -------------------------------------------------- | ---------------- |
| Deletar usuário existente (usando o próprio token) | `200`            |
| Deletar com id diferente do usuário logado         | `403`            |
| Deletar sem informar ID                            | `400`            |

### 📂 Favoritos (`{{base_url}}/favorites`)
Antes de cada request da pasta, um pre-request cria um usuário de teste e um favorito para ele (preenchendo `favoriteId` e `favoritedEventId`). Os requests enviam o token do usuário e os headers `X-Mock-Auth-Scenario` / `X-Mock-Event-Scenario` para acionar os mocks de autenticação e de eventos.

| Cenário                              | Requisição                                    | O que valida                                                                                          | Retorno esperado |
| ------------------------------------ | --------------------------------------------- | ----------------------------------------------------------------------------------------------------- | ---------------- |
| Buscar todos os favoritos do usuário | `GET /favorites/list?userId=...`              | Resposta é um array e cada favorito possui `favoriteId`, `createdAt` e `eventFavorite`                | `200`            |
| Buscar favorito específico           | `GET /favorites?userId=...&favoriteId=...`    | Objeto com `favoriteId`, `createdAt`, `userFavoriteId` e `eventFavorite`, com IDs iguais aos enviados | `200`            |
| Buscar favorito inexistente          | `GET /favorites?userId=...&favoriteId=...`    | `message` "Favorito não foi encontrado"                                                               | `404`            |
| Criar favorito                       | `POST /favorites?userId=...&eventId=...`      | Retorna `favoriteId`, `userFavoriteId` e `eventFavoriteId`, com IDs iguais aos enviados               | `200`            |
| Criar favorito já adicionado à lista | `POST /favorites?userId=...&eventId=...`      | Operação idempotente: retorna o favorito existente, sem gerar um novo `favoriteId`                    | `200`            |
| Remover favorito                     | `DELETE /favorites?userId=...&favoriteId=...` | `message` "Favorito excluído com sucesso"                                                             | `200`            |
| Remover favorito inexistente         | `DELETE /favorites?userId=...&favoriteId=...` | `message` "Favorito não foi encontrado"                                                               | `404`            |

---

## 🛠️ Funções auxiliares (scripts de collection)

Definidas no pre-request script da collection, serializadas via `.toString()` e recuperadas em cada request pelo helper `use(fnName)` (que faz `eval` da função salva como variável de collection).

- **`use(fnName)`** — resolve e recria uma função auxiliar salva como variável de collection.
- **`createSimpleUserRequest(attempt = 1, scenario = "SUCCESS_LOGIN_USER", emailToCreate)`** — cria um usuário no `user_service_url` (senha fixa, nome aleatório, e-mail informado ou sorteado). Em mock, só adiciona o header `x-mock-scenario` — a chamada real não é pulada. Repete até 5 tentativas. Ao ter sucesso, preenche `userRecentCadastreId/Email/Password/Token`.
- **`deleteSimpleUserRequest(token, userId)`** — remove o usuário de teste e limpa as variáveis `userRecentCadastre*`. Pulado em modo mock.
- **`performLogin(email, password, scenario, tokenVarName = "adminToken")`** — loga em `{{auth_service_url}}/login` e salva o token na variável de **environment** indicada. Envia `x-mock-scenario` em modo mock.
- **`sendRecoveryCodeRequest(email, scenario)`** — dispara `{{email_service_url}}/send-recovery-code`, salvando `emailToRecoveryCode` e, quando exposto, `recoveryCode`.

---

## 🧩 Mocks do UserService (WireMock)

Além da collection própria, este repositório mantém os mappings do WireMock consumidos pelos pipelines de CI do `auth-service` e do `email-service`, para simular o UserService como dependência externa sem depender do serviço real.

| Arquivo                             | Endpoint simulado                  | `X-Mock-Scenario`                     | Resposta                                                                                                 |
| ----------------------------------- | ---------------------------------- | ------------------------------------- | -------------------------------------------------------------------------------------------------------- |
| `00-fallback.json`                  | `POST /users/validate-credentials` | *(qualquer, sem cenário reconhecido)* | `500` — erro genérico avisando que o cenário não foi definido                                            |
| `01-validate-success-user.json`     | `POST /users/validate-credentials` | `SUCCESS_VALIDATE_USER`               | `200` — retorna `userId`, `userEmail` e `roleName: "User"`                                               |
| `02-validate-success-admin.json`    | `POST /users/validate-credentials` | `SUCCESS_VALIDATE_ADMIN`              | `200` — retorna `userId`, `userEmail` e `roleName: "Admin"`                                              |
| `03-invalid-credentials.json`       | `POST /users/validate-credentials` | `INVALID_CREDENTIALS`                 | `401` — "Credenciais inválidas"                                                                          |
| `04-user-not-found.json`            | `ANY /users*`                      | `USER_NOT_FOUND`                      | `404` — "Usuário não encontrado"                                                                         |
| `05-invalid-email.json`             | `ANY /users/*`                     | `INVALID_EMAIL`                       | `400` — "email deve ser um email válido"                                                                 |
| `06-get-to-email-success-user.json` | `GET /users?userEmail=...`         | `SUCCESS_GET_USER`                    | `200` — retorna um usuário mockado, ecoando o `userEmail` da query na resposta (via `response-template`) |
| `07-create-success-user.json`       | `POST /users`                      | `SUCCESS_CREATE_USER`                 | `201` — retorna um usuário mockado, com os dados enviados na requisição e `userToken` fixo               |

A prioridade (`priority`) de cada mapping evita conflito entre regras mais genéricas (`ANY /users*`) e as mais específicas por método/endpoint — o fallback (`00`) tem a prioridade mais baixa e só responde quando nenhum outro mapping casa com a requisição.

### Subindo o mock isoladamente

Use a imagem buildada a partir do `docker/mock.Dockerfile` deste repositório — os mappings já ficam embutidos na imagem (`COPY wiremock /home/wiremock`), sem precisar de bind-mount:

```bash
docker build -f docker/mock.Dockerfile -t user-service-mock .

docker run -d --name wiremock-user-service -p 8081:8080 user-service-mock
```

Isso expõe o mock em `http://localhost:8081` (porta padrão utilizada no projeto para o UserService, já configurada no environment `local`). Depois, basta apontar a variável de URL do UserService do serviço/collection sendo testado para esse endereço e enviar o header `X-Mock-Scenario` desejado.