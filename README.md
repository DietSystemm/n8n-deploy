# N8N Deploy - Coolify + Docker

Deploy do n8n em VPS usando Coolify com Docker Compose.

## Stack

- **n8n**: Plataforma de automacao
- **n8n-worker**: Worker para processamento em fila
- **PostgreSQL 15**: Banco de dados
- **Redis 7**: Cache e filas

## Pre-requisitos

- VPS com Coolify instalado
- Dominio apontando para o IP da VPS
- Portas 80 e 443 liberadas

## Deploy via Coolify

### 1. Criar novo servico

1. Acesse o painel do Coolify
2. Clique em **New Resource** > **Docker Compose**
3. Selecione **GitHub** como fonte
4. Conecte ao repositorio `DietSystemm/n8n-deploy`

### 2. Configurar variaveis de ambiente

No Coolify, va em **Environment Variables** e adicione:

```env
# Obrigatorias
N8N_HOST=n8n.seudominio.com
WEBHOOK_URL=https://n8n.seudominio.com/
N8N_ENCRYPTION_KEY=<gerar com: openssl rand -hex 32>
N8N_BASIC_AUTH_USER=admin
N8N_BASIC_AUTH_PASSWORD=<senha-segura>
POSTGRES_PASSWORD=<senha-postgres>
REDIS_PASSWORD=<senha-redis>

# Opcionais (ja tem valores padrao)
GENERIC_TIMEZONE=America/Sao_Paulo
N8N_CONCURRENCY=10
N8N_WORKER_CONCURRENCY=5
```

### 3. Configurar dominio

1. Va em **Domains**
2. Adicione seu dominio (ex: `n8n.seudominio.com`)
3. Ative **HTTPS** (Let's Encrypt)
4. Configure a porta `5678`

### 4. Deploy

Clique em **Deploy** e aguarde o build completar.

## Gerando chaves seguras

```bash
# Encryption key
openssl rand -hex 32

# Senhas
openssl rand -base64 24
```

## Backup

### Backup do banco de dados

```bash
docker exec n8n-postgres pg_dump -U n8n n8n > backup_$(date +%Y%m%d).sql
```

### Restaurar backup

```bash
docker exec -i n8n-postgres psql -U n8n n8n < backup.sql
```

### Backup dos workflows

Os workflows ficam em `/home/node/.n8n` dentro do container, mas com PostgreSQL configurado, os dados ficam no banco.

## Monitoramento

### Logs

```bash
# Todos os servicos
docker compose logs -f

# Apenas n8n
docker compose logs -f n8n

# Apenas worker
docker compose logs -f n8n-worker
```

### Health check

- n8n: `https://n8n.seudominio.com/healthz`

## Atualizacao

No Coolify, va em **Deployments** e clique em **Redeploy** para puxar a versao mais recente do n8n.

Para fixar uma versao especifica, altere no `docker-compose.yml`:

```yaml
image: n8nio/n8n:1.70.0
```

## Solucao de problemas

### Container nao inicia

```bash
docker compose logs n8n
```

### Erro de conexao com banco

Verifique se o PostgreSQL esta healthy:

```bash
docker compose ps
```

### Webhooks nao funcionam

Verifique se `WEBHOOK_URL` esta configurado corretamente com HTTPS.

## Estrutura

```
n8n-deploy/
├── docker-compose.yml          # Stack completa (teste local)
├── docker-compose.coolify.yml  # Otimizado para Coolify (producao)
├── docker-compose.simple.yml   # Versao sem workers
├── .env.example                # Template de variaveis
├── .gitignore                  # Arquivos ignorados
└── README.md                   # Documentacao
```

## Qual docker-compose usar?

| Arquivo | Uso |
|---------|-----|
| `docker-compose.yml` | Teste local com porta exposta |
| `docker-compose.coolify.yml` | **Producao no Coolify** (recomendado) |
| `docker-compose.simple.yml` | VPS pequenas sem workers |

No Coolify, selecione `docker-compose.coolify.yml` como arquivo de configuracao.
