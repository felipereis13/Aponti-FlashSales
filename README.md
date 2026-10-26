# Flash Sales API

API para vendas relâmpago de ingressos, desenvolvida com Node.js, TypeScript e Express. Utiliza Redis com script Lua para controle atômico de estoque, PostgreSQL com TypeORM para persistência de pedidos e Prometheus com Grafana para observabilidade.

## Funcionalidades

- Compra de ingressos com validação de disponibilidade.
- Controle atômico de estoque no Redis.
- Persistência de pedidos em PostgreSQL.
- Endpoint para simulação de relatórios pesados e avaliação sob carga.
- Exposição de métricas para o Prometheus e visualização no Grafana.
- Testes automatizados e pipeline de integração e publicação de imagens Docker.

## Tecnologias

| Categoria | Tecnologias |
| --- | --- |
| Linguagem e execução | TypeScript e Node.js 22 |
| Framework HTTP | Express |
| Banco de dados e ORM | PostgreSQL 16 e TypeORM |
| Controle de estoque | Redis 7 e Lua |
| Observabilidade | Prometheus e Grafana |
| Testes | Jest |
| Ambiente | Docker e Docker Compose |
| Automação | GitHub Actions e Docker Hub |

## Estrutura do projeto

```text
.
├── .github/
│   └── workflows/       # Workflows de CI/CD
├── src/
│   ├── config/         # Configuração de conexões com PostgreSQL e Redis
│   ├── controllers/    # Recebimento de requisições e envio de respostas
│   ├── entities/       # Entidades mapeadas pelo TypeORM
│   ├── middlewares/    # Coleta de métricas e tratamento de erros
│   ├── routes/         # Definição das rotas da API
│   └── services/       # Regras de negócio e controle de estoque
├── tests/              # Testes automatizados
├── Dockerfile          # Definição da imagem da aplicação
├── docker-compose.yml  # Orquestração do ambiente local
└── prometheus.yml      # Configuração de coleta de métricas
```

## Infraestrutura

O ambiente é executado em contêineres. O Docker Compose inicializa cinco serviços:

| Serviço | Nome no Compose | Porta | Responsabilidade |
| --- | --- | --- | --- |
| API | `app` | `3000` | Processar requisições e executar as regras de negócio |
| PostgreSQL | `postgres` | `5432` | Persistir pedidos |
| Redis | `redis` | `6379` | Controlar o estoque de ingressos |
| Prometheus | `prometheus` | `9090` | Coletar métricas da API a cada 15 segundos |
| Grafana | `grafana` | `3001` | Disponibilizar painéis de monitoramento |

As credenciais definidas no `docker-compose.yml` destinam-se exclusivamente ao ambiente de desenvolvimento e testes locais.

## Execução local

### Pré-requisitos

- Git.
- Docker com Docker Compose disponível pelo comando `docker compose`.
- Node.js 22 e npm, caso deseje executar os testes fora dos contêineres.

### 1. Clonar o repositório

```bash
git clone https://github.com/joaov877/flash-sales.git
cd flash-sales
```

### 2. Inicializar o ambiente

```bash
docker compose up --build
```

Para executar os serviços em segundo plano:

```bash
docker compose up --build -d
```

### 3. Acessar os serviços

| Recurso | Endereço |
| --- | --- |
| Endereço base da API | http://localhost:3000 |
| Métricas da API | http://localhost:3000/metrics |
| Prometheus | http://localhost:9090 |
| Grafana | http://localhost:3001 |

Credenciais locais do Grafana: usuário `admin` e senha `admin`.

### 4. Encerrar o ambiente

```bash
docker compose down
```

## Rotas da API

| Método | Rota | Descrição |
| --- | --- | --- |
| `POST` | `/checkout` | Valida o estoque no Redis e persiste o pedido no PostgreSQL |
| `POST` | `/tickets/export` | Simula o processamento de relatórios pesados para testes sob carga |
| `GET` | `/metrics` | Expõe métricas no formato compatível com o Prometheus |

### Compra de ingressos

Antes de realizar uma compra de teste, inicialize o estoque do evento no Redis. Execute o comando na raiz do projeto, com o ambiente em execução:

```bash
docker compose exec redis redis-cli SET event:show-123:tickets 100
```

Esse comando define o estoque do evento `show-123` em 100 ingressos. Executá-lo novamente substitui o saldo existente.

Envie uma requisição para `POST /checkout`:

```bash
curl -i -X POST http://localhost:3000/checkout \
  -H "Content-Type: application/json" \
  -d '{ "eventId": "show-123", "userId": "user-01", "quantity": 2 }'
```

O exemplo utiliza a sintaxe de Bash.

#### Corpo da requisição

| Campo | Tipo | Descrição |
| --- | --- | --- |
| `eventId` | `string` | Identificador do evento |
| `userId` | `string` | Identificador do comprador |
| `quantity` | `number` | Quantidade de ingressos solicitada |

#### Códigos de resposta

| Código HTTP | Significado |
| --- | --- |
| `201 Created` | Compra realizada com sucesso |
| `400 Bad Request` | Ausência de dados obrigatórios |
| `409 Conflict` | Ingressos esgotados |

## Testes

Os testes automatizados verificam as regras de negócio, incluindo a validação de estoque e a criação de pedidos, sem depender de conexões com bancos de dados reais.

Para instalar as dependências e executar os testes, utilize:

```bash
npm ci
npm test
```

## Integração e entrega contínuas

O workflow definido em `.github/workflows/ci-cd.yml` automatiza a execução dos testes e a publicação da imagem Docker.

| Etapa | Comportamento |
| --- | --- |
| Testes | Executados a cada push ou pull request direcionado às branches `main` ou `master` |
| Publicação | Após a aprovação dos testes, alterações enviadas à branch principal acionam a construção e a publicação da imagem no Docker Hub |
| Versionamento da imagem | Publicação com a tag `latest` e uma tag associada ao identificador do commit |

### Configuração dos secrets

No repositório do GitHub, acesse **Settings → Secrets and variables → Actions** e configure:

| Secret | Descrição |
| --- | --- |
| `DOCKERHUB_USERNAME` | Nome de usuário da conta no Docker Hub |
| `DOCKERHUB_TOKEN` | Token de acesso do Docker Hub utilizado pela pipeline para publicar imagens |
