# Infra do DivulgaIF - Uma vitrine dos projetos dos alunos!

Feito por Artur Zanqueta, Eduardo Ceciliato, Igor Felipe de Souza, SM: Mateus Rakoski e PO: Andrey Jodar

### Ele está acessível em:

[https://divulgaif.com.br/](https://divulgaif.com.br/)

## Arquitetura

- **Backend**: Java Spring Boot com suporte SSL
- **Frontend**: React (Create React App)
- **Banco de Dados**: PostgreSQL 15 com replicação primário/secundário
- **Containerização**: Docker & Docker Compose

## Pré-requisitos

- Docker e Docker Compose
- Java 17+ (para desenvolvimento local do backend)
- Node.js 18+ e npm (para desenvolvimento local do frontend)
- Certificado SSL (keystore.p12) para HTTPS

## Variáveis de Ambiente

Você pode utilizar tanto um arquivo `.env` quanto definir as variáveis de ambiente em seu sistema operacional.

Crie um arquivo `.env` na raiz do seu projeto com as seguintes variáveis:

```env
# Configuração do Banco de Dados
DIVULGAIF_DB=divulgaif_prod                   # Nome do banco primário
DIVULGAIF_DB_USER=divulgaif_user              # Usuário do banco primário
DIVULGAIF_TEST_DB=divulgaif_test              # Nome do banco secundário/teste
DIVULGAIF_DB_TEST_USER=divulgaif_test_user    # Usuário do banco secundário
POSTGRES_PASSWORD=sua_senha_segura            # Senha do banco (compartilhada)
POSTGRES_HOST_AUTH_METHOD=md5                 # Método de autenticação PostgreSQL

# Autenticação JWT
DIVULGAIF_JWT_TOKEN=sua_chave_secreta_jwt     # Chave secreta para assinatura JWT (mín 256 bits)

# Configuração SSL
SSL_PASSWORD=senha_do_keystore                # Senha do arquivo keystore SSL
```

### Explicação das Variáveis de Ambiente

| Variável | Descrição | Exemplo |
|----------|-----------|---------|
| `DIVULGAIF_DB` | Nome do banco PostgreSQL primário | `divulgaif_prod` |
| `DIVULGAIF_DB_USER` | Usuário do banco primário | `divulgaif_user` |
| `DIVULGAIF_TEST_DB` | Banco secundário para testes/replicação | `divulgaif_test` |
| `DIVULGAIF_DB_TEST_USER` | Usuário do banco de teste | `divulgaif_test_user` |
| `POSTGRES_PASSWORD` | Senha para ambos os bancos | `minhaSenhaSegura123!` |
| `POSTGRES_HOST_AUTH_METHOD` | Método de autenticação PostgreSQL | `md5` ou `trust` |
| `DIVULGAIF_JWT_TOKEN` | Chave secreta JWT para assinatura de tokens | `minhaChaveJWT256Bits...` |
| `SSL_PASSWORD` | Senha do arquivo keystore SSL | `senhaDoKeystore` |

## Como Executar

### 1. Executar com Docker Compose (Recomendado)

```bash
# Clone o repositório
git clone <url-do-repositorio>
cd divulgaif

# Configure as variáveis de ambiente
cp .env.example .env
# Edite o arquivo .env com suas configurações

# Construa e execute todos os serviços
docker-compose up --build

# Para executar em background
docker-compose up -d --build
```

**Serviços disponíveis após execução:**
- Backend: https://localhost:8081/api/v1
- Banco Primário: localhost:5432
- Banco Secundário: localhost:5433

### 2. Executar Backend Localmente (Desenvolvimento)

```bash
# Navegue até o diretório do backend
cd divulgaif-back

# Execute apenas os bancos com Docker
docker-compose up divulgaif-primary divulgaif-secondary -d

# Configure as variáveis de ambiente locais
export DIVULGAIF_DB_HOST=localhost
export DIVULGAIF_DB=seu_banco
export DIVULGAIF_DB_USER=seu_usuario
export POSTGRES_PASSWORD=sua_senha
export DIVULGAIF_JWT_TOKEN=sua_chave_jwt
export SSL_PASSWORD=senha_keystore

# Execute o Spring Boot
./mvnw spring-boot:run

# Ou compile e execute o JAR
./mvnw clean package
java -jar target/divulgaif-back-*.jar
```

### 3. Executar Frontend Localmente (Desenvolvimento)

```bash
# Navegue até o diretório do frontend
cd divulgaif-front

# Instale as dependências
npm install

# Execute em modo de desenvolvimento
npm start

# Para build de produção
npm run build

# Para servir o build
npm install -g serve
serve -s build
```

**Frontend estará disponível em:** http://localhost:3000

## Configurações Importantes

### SSL/HTTPS
O backend está configurado para usar HTTPS. Certifique-se de ter o arquivo `keystore.p12` no diretório correto do backend e configure a senha corretamente.

### CORS
O backend está configurado para aceitar requisições de:
- https://localhost:36643
- https://frontend:4200  
- http://localhost:3000
- https://divulgaif.com.br
- https://desenvolvimento.divulgaif.com.br

### JWT
Os tokens JWT têm os seguintes tempos de expiração:
- Access Token: 2 horas
- Refresh Token: 4280 horas (~6 meses)

### Banco de Dados
- **Flyway**: Gerenciamento de migrações habilitado
- **Hibernate**: Modo `validate` (não altera estrutura automaticamente)
- **Dialect**: PostgreSQL
- **Timezone**: UTC

## Comandos Úteis

```bash
# Parar todos os serviços
docker-compose down

# Remover volumes (CUIDADO: apaga dados do banco)
docker-compose down -v

# Ver logs
docker-compose logs -f

# Ver logs de um serviço específico
docker-compose logs -f backend

# Acessar shell do container
docker-compose exec backend bash
docker-compose exec divulgaif-primary psql -U $DIVULGAIF_DB_USER -d $DIVULGAIF_DB

# Rebuild apenas um serviço
docker-compose up --build backend
```

## Estrutura do Projeto

```
divulgaif/
├── docker-compose.yml          # Configuração dos containers
├── .env                        # Variáveis de ambiente
├── divulgaif-back/            # Backend Spring Boot
│   ├── src/
│   ├── Dockerfile
│   ├── pom.xml
│   └── keystore.p12           # Certificado SSL
└── divulgaif-front/           # Frontend React
    ├── src/
    ├── public/
    ├── package.json
    └── Dockerfile (opcional)
```

## Troubleshooting

### Problemas Comuns

1. **Erro de conexão com banco**: Verifique se as variáveis de ambiente estão corretas
2. **Erro SSL**: Certifique-se de que o keystore.p12 existe e a senha está correta
3. **CORS Error**: Verifique se a URL do frontend está na lista de origins permitidas
4. **Porta em uso**: Altere as portas no docker-compose.yml se necessário

### Logs e Debug

```bash
# Backend logs
docker-compose logs backend

# Database logs
docker-compose logs divulgaif-primary

# Verificar saúde dos containers
docker-compose ps
```