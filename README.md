# LiteLLM Bedrock

A Docker Compose setup for [LiteLLM](https://github.com/BerryAI/litellm) proxy backed by AWS Bedrock, with PostgreSQL and Redis.

## Models

| Alias | Bedrock Model |
|---|---|
| claude-opus-5 | `global.anthropic.claude-opus-5` |
| claude-opus-4-6 | `global.anthropic.claude-opus-4-6-v1` |
| claude-haiku-4-5 | `us.anthropic.claude-haiku-4-5-20251001-v1:0` |
| gpt-oss-120b | `openai.gpt-oss-120b-1:0` |

## Prerequisites

- Docker and Docker Compose
- AWS credentials configured in `~/.aws` with a profile that has Bedrock access

## Setup

1. Create the external PostgreSQL volume:

```bash
docker volume create litellm-bedrock-pg
```

2. Set environment variables in `docker-compose.yml`:

```yaml
environment:
  AWS_REGION:  <your-aws-region>
  AWS_PROFILE: <your-aws-profile>
  DATABASE_URL: postgresql://<db-user>:<db-password>@db:5432/litellm
  LITELLM_MASTER_KEY: <your-master-key>
```

3. Start the services:

```bash
docker compose up -d
```

The LiteLLM proxy will be available at `http://localhost:4000`.

## Services

| Service | Port |
|---|---|
| LiteLLM Proxy | 4000 |
| PostgreSQL | 5433 |
| Redis | 6380 |

## Usage

### Test a model

```bash
curl http://localhost:4000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer ${LITELLM_MASTER_KEY}" \
  -d '{
    "model": "claude-opus-4-6",
    "messages": [{"role": "user", "content": "Say hello in one sentence"}]
  }'
```

### Check spend

```bash
curl http://localhost:4000/global/spend \
  -H "Authorization: Bearer ${LITELLM_MASTER_KEY}"
```

### Check spend per model

```bash
curl http://localhost:4000/global/spend/models \
  -H "Authorization: Bearer ${LITELLM_MASTER_KEY}"
```

## License

[MIT](LICENSE)