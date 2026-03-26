# CrowVault Setup — GitHub Action

Install and authenticate the [CrowVault](https://crowvault.ai) CLI in your GitHub Actions workflows. Generate schemas, Dockerfiles, K8s configs, OpenAPI specs, Terraform modules, and run multi-step workflows — all in CI/CD.

## Usage

```yaml
- uses: rajeswaran140/crowvault-setup@v1
  with:
    api-key: ${{ secrets.CROWVAULT_API_KEY }}

- run: crowvault schema Order --format prisma --output ./prisma/schema.prisma

- run: crowvault dockerfile node --version 20 --output Dockerfile

- run: crowvault workflow run deploy-stack --arg name=my-app --arg runtime=node
```

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `api-key` | Yes | — | CrowVault API key (`cv_...`). Store as a [GitHub secret](https://docs.github.com/en/actions/security-guides/encrypted-secrets). |
| `version` | No | `latest` | CLI version to install |
| `api-url` | No | `https://api.crowvault.ai` | API URL |

## Examples

### Generate database schema on PR

```yaml
name: Generate Schema
on: pull_request

jobs:
  schema:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: rajeswaran140/crowvault-setup@v1
        with:
          api-key: ${{ secrets.CROWVAULT_API_KEY }}
      - run: crowvault schema Order --format prisma --output ./prisma/schema.prisma
      - run: git diff --exit-code ./prisma/ || echo "Schema changed"
```

### Run full workflow in CI

```yaml
name: Scaffold Microservice
on: workflow_dispatch

jobs:
  scaffold:
    runs-on: ubuntu-latest
    steps:
      - uses: rajeswaran140/crowvault-setup@v1
        with:
          api-key: ${{ secrets.CROWVAULT_API_KEY }}
      - run: crowvault workflow run microservice-scaffold --arg name=orders --arg language=node --output result.json
```

### Batch generation

```yaml
- uses: rajeswaran140/crowvault-setup@v1
  with:
    api-key: ${{ secrets.CROWVAULT_API_KEY }}
- run: |
    crowvault schema User --format prisma --output prisma/user.prisma
    crowvault schema Order --format prisma --output prisma/order.prisma
    crowvault dockerfile node --version 20 --output Dockerfile
    crowvault terraform aws --env production --output infra/main.tf
```

## Available Commands

| Command | Description |
|---------|-------------|
| `crowvault servers` | List 9 MCP servers (325 tools) |
| `crowvault tools --search <query>` | Search tools |
| `crowvault call <server> <tool> --arg key=value` | Call any tool |
| `crowvault schema <name>` | Generate database schema |
| `crowvault dockerfile <runtime>` | Generate Dockerfile |
| `crowvault openapi` | Generate OpenAPI spec |
| `crowvault terraform <provider>` | Generate Terraform module |
| `crowvault workflow run <name>` | Run multi-step workflow |
| `crowvault batch <file.json>` | Parallel execution (up to 10) |

## 6 Workflows

- `microservice-scaffold` — Microservice + schema + migration + Docker + K8s (5 steps)
- `api-full-stack` — OpenAPI + endpoint + tests + contracts (4 steps)
- `ddd-complete` — Bounded context + model + aggregate + entity + repo + schema (6 steps)
- `event-driven` — Kafka + event handler + DLQ (3 steps)
- `deploy-stack` — Dockerfile + Compose + K8s + Helm (4 steps)
- `database-setup` — Schema + ORM + migration + seed (4 steps)

## Get an API Key

1. Sign up free at [crowvault.ai/register](https://crowvault.ai/register?plan=free) (50 calls/month)
2. Go to Dashboard → API Keys → Create Key
3. Add to your repo: Settings → Secrets → New secret → `CROWVAULT_API_KEY`

## License

MIT. Copyright TechSynergy Corp.
