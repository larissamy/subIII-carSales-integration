# Auth Buyers Service

## Sobre

API REST responsável pelo **cadastro e autenticação de compradores** da plataforma de revenda de veículos do Tech Challenge SOAT – Fase 3.

Este serviço foi criado de forma **separada** da API transacional de veículos/vendas para atender ao requisito de isolamento dos dados de compradores e do processo de autorização.

A API permite:
- Cadastro de comprador
- Login com e-mail e senha
- Geração de JWT
- Consulta do comprador autenticado
- Listagem de compradores cadastrados (apenas para apoio em desenvolvimento/demonstração)

## Arquitetura

O projeto segue uma separação inspirada em **Clean Architecture**:

```text
src/main/java
├── domain          # Entidades e regras de negócio
├── application     # Casos de uso, serviços e DTOs
├── infrastructure  # Persistência, segurança, JWT e configuração
└── presentation    # Controllers REST e tratamento de erros
```

## Tecnologias

- Java 17
- Spring Boot 3
- Maven
- Spring Security
- JWT (JJWT)
- SQLite (arquivo local)
- Swagger / OpenAPI
- Docker
- Kubernetes
- GitHub Actions

## Como foi implementado

A solução foi dividida em um serviço dedicado de compradores/autenticação.

### Fluxo principal
1. O comprador se cadastra via `POST /auth/register`
2. O comprador faz login via `POST /auth/login`
3. O serviço retorna um token JWT
4. O token pode ser usado pela API de veículos/vendas para autorizar a compra do veículo

### Dados persistidos
Cada comprador possui:
- `id`
- `name`
- `email`
- `passwordHash`
- `createdAt`
- `updatedAt`

A senha é persistida com hash BCrypt.

## Como rodar localmente

### Pré-requisitos
- Java 17
- Maven
- Docker (opcional)
- Kubernetes (opcional)

### Rodando localmente

```bash
mvn clean package -DskipTests
mvn spring-boot:run
```

API disponível em:
- `http://localhost:8081`

Swagger:
- `http://localhost:8081/swagger-ui/index.html`

## Como testar

### 1. Registrar comprador
`POST /auth/register`

Exemplo de body:

```json
{
  "name": "Larissa Yamaguchi",
  "email": "larissa@example.com",
  "password": "Senha@123"
}
```

### 2. Fazer login
`POST /auth/login`

```json
{
  "email": "larissa@example.com",
  "password": "Senha@123"
}
```

A resposta terá um `token` JWT.

### 3. Consultar comprador autenticado
`GET /auth/me`

Use o token no header:

```text
Authorization: Bearer <token>
```

### 4. Listar compradores
`GET /buyers`

## Docker

```bash
mvn clean package -DskipTests
docker build -t auth-buyers-service:local .
docker run --rm -p 8081:8081 auth-buyers-service:local
```

Ou com Docker Compose:

```bash
docker compose up
```

## Kubernetes

```bash
kubectl apply -f k8s/
kubectl get pods
kubectl port-forward svc/auth-buyers-service 8081:80
```

## Deploy automatizado

O projeto inclui pipelines GitHub Actions para:
- build e testes em pull requests
- validação básica dos manifests Kubernetes
- build e publicação da imagem Docker no GHCR em pushes para `main`

## Endpoints

### Auth
- `POST /auth/register`
- `POST /auth/login`
- `GET /auth/me`

### Buyers
- `GET /buyers`
- `GET /buyers/{id}`

## Observações

- Este serviço foi pensado para ser consumido pela API transacional de veículos/vendas.
- Em um cenário real, a API de vendas deve validar o JWT e extrair o `buyerId` do token para efetuar a compra.
