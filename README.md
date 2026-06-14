# Bot de WhatsApp com IA

Projeto de automação para WhatsApp usando **WAHA**, **n8n**, **Redis** e **Postgres** via Docker Compose.

## Tecnologias

- [WAHA](https://waha.devlike.pro/) — API HTTP para WhatsApp
- [n8n](https://n8n.io/) — automação e workflows
- Redis — memória/contexto das conversas
- Postgres — banco de dados do n8n
- Docker Compose — orquestração dos serviços

## Portas utilizadas

| Serviço | Porta | URL |
|---|---:|---|
| WAHA | 3000 | http://localhost:3000 |
| WAHA Dashboard | 3000 | http://localhost:3000/dashboard |
| n8n | 5678 | http://localhost:5678 |
| Redis | 6379 | localhost:6379 |
| Postgres | 5432 | localhost:5432 |

## Como executar

Na pasta do projeto, rode:

```bash
docker compose up -d
```

Para verificar os containers:

```bash
docker compose ps
```

Para parar tudo:

```bash
docker compose down
```

## Acessando o WAHA

Acesse:

```text
http://localhost:3000/dashboard
```

Se o WAHA pedir usuário e senha, veja as credenciais nos logs:

```bash
docker logs waha
```

Procure por:

```text
WAHA_DASHBOARD_USERNAME
WAHA_DASHBOARD_PASSWORD
WAHA_API_KEY
```

## Configurando o Dashboard do WAHA

No dashboard, conecte ao servidor usando:

```text
Server URL: http://localhost:3000
API Key: valor_da_WAHA_API_KEY_dos_logs
```

Depois crie ou inicie uma sessão e conecte o WhatsApp pelo QR Code.

## Webhook com n8n

O WAHA envia eventos de mensagens para o n8n.

No `docker-compose.yml`, o webhook configurado é:

```text
http://host.docker.internal:5678/webhook/webhook
```

Esse endpoint é a URL de produção do n8n.

### Para usar em produção

No n8n:

1. Crie um workflow com um nó **Webhook**
2. Configure o path como:

```text
webhook
```

3. Ative o workflow no canto superior direito
4. Envie uma mensagem para o WhatsApp conectado

A execução aparecerá em **Executions**.

### Para testar no editor do n8n

Se estiver usando **Listen for test event**, use a URL de teste:

```text
http://host.docker.internal:5678/webhook-test/webhook
```

Importante: dentro do container WAHA, não use `localhost` para acessar o n8n. Use `host.docker.internal`.

## Credenciais padrão dos serviços

### Postgres

```text
Usuário: n8n
Senha: n8n
Banco: n8n
```

### Redis

```text
Senha: default
```

## Logs úteis

Ver logs do WAHA:

```bash
docker logs -f waha
```

Ver logs do n8n:

```bash
docker logs -f n8n
```

Ver logs de todos os serviços:

```bash
docker compose logs -f
```

## Problemas comuns

### Erro 401 Unauthorized no WAHA

O dashboard ou a API estão sem a API Key correta.

Verifique a chave com:

```bash
docker logs waha
```

E use o valor de `WAHA_API_KEY` no dashboard.

### n8n não recebe mensagens

Verifique se o workflow está ativo.

Se o log mostrar:

```text
The requested webhook "POST webhook" is not registered
```

significa que o endpoint de produção não está ativo no n8n.

Soluções:

- Ativar o workflow no n8n
- Ou usar a URL de teste com `webhook-test`

### ECONNREFUSED 127.0.0.1:5678

Isso acontece quando o WAHA tenta acessar `localhost`, mas `localhost` dentro do container é o próprio container WAHA.

Use:

```text
host.docker.internal
```

em vez de:

```text
localhost
```

## Estrutura do projeto

```text
.
└── docker-compose.yml
```

## Observações

Este projeto é voltado para estudos de integração entre WhatsApp, automação com n8n e APIs.

Não exponha o WAHA publicamente sem autenticação, HTTPS e proteção adequada.
