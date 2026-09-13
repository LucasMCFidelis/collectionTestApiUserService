# 👤 collectionTestApiUserService

Repositório de suporte a testes do **UserService** do projeto *Catálogo de Eventos*.

> ⚠️ **Estado atual**: apesar do nome, este repositório **ainda não contém uma collection do Postman própria para o UserService**. Hoje ele reúne apenas os **mappings do WireMock** que simulam as respostas do UserService, reaproveitados como dependência mockada pelas suítes de teste do [`auth-service`](https://github.com/LucasMCFidelis/auth-service-eventsCatalog-) e do [`email-service`](https://github.com/LucasMCFidelis/email-service-eventsCatalog-). Uma collection de testes de integração do UserService em si (criar usuário, buscar por e-mail, etc.) ainda está pendente.

---

## 🎯 Para que serve

Outros serviços do *Catálogo de Eventos* dependem do UserService para validar credenciais de login (`POST /users/validate-credentials`) e para buscar dados de um usuário por e-mail (`GET /users?userEmail=...`). Para não depender do UserService real (nem do seu estado de dados) durante os testes de CI desses serviços, essa dependência é substituída por um container **WireMock**, configurado com os mappings deste repositório.

Cada mapping representa um cenário de resposta, selecionado pelo header `X-Mock-Scenario` enviado na requisição.

---

## 📁 Estrutura

```
collectionTestApiUserService/
└── postman/
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

> A pasta se chama `postman/wiremock` por convenção do projeto (agrupando tudo relacionado a testes de API), mas o conteúdo hoje é exclusivamente de configuração do WireMock — não há arquivos de collection do Postman.

---

## 🧩 Mappings disponíveis

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

---

## ▶️ Como usar

Suba um container WireMock apontando para a pasta de mappings deste repositório:

```bash
docker run -d --name wiremock-user-service -p 8089:8080 \
  -v "$(pwd)/postman/wiremock:/home/wiremock" \
  wiremock/wiremock
```

Isso expõe o mock em `http://localhost:8089`. 

Depois, basta apontar a variável de URL do UserService do serviço sendo testado (`USER_SERVICE_URL_DEV`, `user_service_url`, etc.) para `http://localhost:8089` e enviar o header `X-Mock-Scenario` desejado nas requisições de teste.

Esse é exatamente o uso feito hoje pelos pipelines de CI do `auth-service` e do `email-service` — veja os respectivos READMEs para o passo a passo completo de integração.

---

## 🚧 Próximos passos

- [ ] Adicionar uma collection do Postman própria, com testes de integração do UserService (cadastro, consulta por e-mail, remoção do usuário de teste, etc.), no padrão das demais collections do projeto (`collectionTestApiAuthService`, `collectionTestApiEmailService`).
- [ ] Adicionar *environments* (`local-mock`, `local-real`, `ci`, `production-like`) equivalentes aos das outras collections.
- [ ] Documentar, quando a collection existir, as variáveis necessárias e os cenários de teste cobertos.