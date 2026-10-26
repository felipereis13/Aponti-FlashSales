Flash Sales API 


API de vendas relâmpago de ingressos, construída com Node.js + TypeScript + Express, com controle de estoque atômico via Redis (script Lua), persistência de pedidos em PostgreSQL (TypeORM) e observabilidade com Prometheus + Grafana.


Tecnologias
Linguagem & Framework: Node.js 22, TypeScript, Express

Banco de Dados: PostgreSQL 16 (TypeORM)

Cache & Estoque: Redis 7

Observabilidade: Prometheus e Grafana

Ambiente & Testes: Docker, Docker Compose, Jest

Automação (CI/CD): GitHub Actions e Docker Hub


🗂️ Estrutura do Projeto
Plaintext
.
├── .github/workflows/    # Configuração da pipeline de CI/CD
├── src/                  # Código-fonte da aplicação
│   ├── config/           # Conexões com banco de dados e cache
│   ├── controllers/      # Entrada das requisições e respostas
│   ├── entities/         # Modelos de dados (PostgreSQL)
│   ├── middlewares/      # Coleta de métricas e tratamento de erros
│   ├── routes/           # Rotas disponíveis na API
│   └── services/         # Regras de negócio (ex: validação de estoque)
├── tests/                # Testes automatizados
├── Dockerfile            # Receita para criar a imagem da API
├── docker-compose.yml    # Sobe todo o ambiente localmente
└── prometheus.yml        # Configuração da coleta de métricas





🐳 Como Funciona a Infraestrutura
Docker & Docker Compose
O projeto está 100% containerizado. Com um único comando, o Docker Compose inicializa 5 serviços interligados:

API (app): A aplicação Node.js rodando na porta 3000.

PostgreSQL (postgres): Armazena os pedidos salvos na porta 5432.

Redis (redis): Controla a contagem rápida e segura do estoque na porta 6379.

Prometheus (prometheus): Coleta métricas de desempenho da API a cada 15 segundos na porta 9090.

Grafana (grafana): Painel visual para monitorar a saúde da API na porta 3001.

 Aviso: As senhas do docker-compose.yml são apenas para testes locais.
 



 Como Rodar o Projeto
 

Pré-requisitos
Ter o Docker e o Docker Compose instalados na sua máquina.

Passo a Passo

1/Clone o repositório:
Bash
git clone https://github.com/joaov877/flash-sales.git
cd flash-sales

2/Suba todo o ambiente:
Bash
docker compose up --build

3/Acesse os serviços:
API: http://localhost:3000

Métricas brutas: http://localhost:3000/metrics

Painel do Prometheus: http://localhost:9090

Painel do Grafana: http://localhost:3001 (Login: admin / Senha: admin)

4/Para parar o ambiente:
Bash
docker compose down


Rotas da API


Método	Rota	Descrição
POST	/checkout	Realiza a compra do ingresso, valida o estoque no Redis e salva o pedido no Banco.
POST	/tickets/export	Rota para simular relatórios pesados e testar a capacidade da API sob carga.
GET	/metrics	Retorna as métricas de desempenho formatadas para o Prometheus.
Exemplo de Checkout (POST /checkout)
Bash
curl -X POST http://localhost:3000/checkout \
  -H "Content-Type: application/json" \
  -d '{
    "eventId": "show-123",
    "userId": "user-01",
    "quantity": 2
  }'
201 Created: Compra realizada com sucesso.

400 Bad Request: Dados obrigatórios faltando.

409 Conflict: Ingressos esgotados.

Dica de teste: Antes de comprar, defina o estoque do evento no Redis executando:

docker exec -it flash-sales-redis redis-cli SET event:show-123:tickets 100


   Testes

   
Os testes verificam se as regras de negócio (como validação de estoque e criação de pedidos) estão funcionando sem precisar conectar aos bancos de dados reais.

Para rodar os testes localmente:

Bash
npm ci
npm test



🔄 CI/CD e Automação


O projeto conta com uma esteira automatizada no GitHub Actions (.github/workflows/ci-cd.yml):

Testes Automáticos: A cada push ou pull request nas branches main ou master, os testes unitários são executados em um banco de dados temporário.

Publicação da Imagem: Se os testes passarem e a alteração for enviada para a branch principal, a pipeline constrói a imagem Docker e publica no Docker Hub sob a tag latest e com o identificador único do commit.

Variaveis Secretas (Secrets)
Para a publicação automática funcionar, configure as seguintes variáveis nas configurações do repositório (Settings -> Secrets and variables -> Actions):

DOCKERHUB_USERNAME: Seu usuário no Docker Hub.

DOCKERHUB_TOKEN: Token de acesso gerado no Docker Hub.
