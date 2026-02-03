---
description: Environment variable reference
---

# Environment Variables

## Environment Variable Reference

All environment variables supported by Sensei.

---

### Authentication

| Variable         | Description                | Default                 |
| ---------------- | -------------------------- | ----------------------- |
| `SENSEI_API_KEY` | API key for authentication | (required)              |
| `SENSEI_API_URL` | API endpoint URL           | `https://api.sensei.ai` |
| `SENSEI_ORG_ID`  | Organization ID            | (from API key)          |

---

### Database Connections

| Variable             | Description                | Default     |
| -------------------- | -------------------------- | ----------- |
| `SENSEI_DB_HOST`     | Internal database host     | `localhost` |
| `SENSEI_DB_PORT`     | Internal database port     | `5432`      |
| `SENSEI_DB_NAME`     | Internal database name     | `sensei`    |
| `SENSEI_DB_USER`     | Internal database user     | `sensei`    |
| `SENSEI_DB_PASSWORD` | Internal database password | (required)  |
| `SENSEI_DB_SSL_MODE` | SSL mode                   | `require`   |

---

### Redis

| Variable                | Description          | Default                  |
| ----------------------- | -------------------- | ------------------------ |
| `SENSEI_REDIS_URL`      | Redis connection URL | `redis://localhost:6379` |
| `SENSEI_REDIS_PASSWORD` | Redis password       | (none)                   |
| `SENSEI_REDIS_TLS`      | Enable Redis TLS     | `false`                  |

---

### Storage

| Variable                | Description                     | Default              |
| ----------------------- | ------------------------------- | -------------------- |
| `SENSEI_STORAGE_TYPE`   | Storage backend                 | `s3`                 |
| `SENSEI_S3_BUCKET`      | S3 bucket name                  | (required)           |
| `SENSEI_S3_REGION`      | AWS region                      | `us-east-1`          |
| `SENSEI_S3_ENDPOINT`    | S3 endpoint (for S3-compatible) | (AWS default)        |
| `AWS_ACCESS_KEY_ID`     | AWS access key                  | (from instance role) |
| `AWS_SECRET_ACCESS_KEY` | AWS secret key                  | (from instance role) |

---

### Encryption

| Variable                | Description                | Default    |
| ----------------------- | -------------------------- | ---------- |
| `SENSEI_ENCRYPTION_KEY` | Data encryption key        | (required) |
| `SENSEI_KMS_KEY_ID`     | AWS KMS key ID             | (optional) |
| `SENSEI_VAULT_ADDR`     | HashiCorp Vault address    | (optional) |
| `SENSEI_VAULT_TOKEN`    | Vault authentication token | (optional) |

---

### AI/LLM

| Variable              | Description         | Default               |
| --------------------- | ------------------- | --------------------- |
| `SENSEI_LLM_PROVIDER` | LLM provider        | `anthropic`           |
| `ANTHROPIC_API_KEY`   | Anthropic API key   | (required for Anthropic) |
| `OPENAI_API_KEY`      | OpenAI API key      | (required for OpenAI) |
| `SENSEI_LLM_MODEL`    | Model to use        | `claude-sonnet-4-20250514` |
| `SENSEI_LLM_ENDPOINT` | Custom LLM endpoint | (provider default)    |

---

### Logging

| Variable            | Description            | Default       |
| ------------------- | ---------------------- | ------------- |
| `SENSEI_LOG_LEVEL`  | Log level              | `info`        |
| `SENSEI_LOG_FORMAT` | Log format (json/text) | `json`        |
| `SENSEI_LOG_FILE`   | Log file path          | (stdout only) |
| `SENSEI_DEBUG`      | Enable debug mode      | `false`       |

---

### Monitoring

| Variable                  | Description     | Default    |
| ------------------------- | --------------- | ---------- |
| `SENSEI_METRICS_ENABLED`  | Enable metrics  | `true`     |
| `SENSEI_METRICS_PORT`     | Metrics port    | `9090`     |
| `SENSEI_TRACING_ENABLED`  | Enable tracing  | `false`    |
| `SENSEI_TRACING_ENDPOINT` | OTLP endpoint   | (none)     |
| `DATADOG_API_KEY`         | Datadog API key | (optional) |

---

### Workers

| Variable                | Description       | Default |
| ----------------------- | ----------------- | ------- |
| `SENSEI_WORKER_COUNT`   | Number of workers | `4`     |
| `SENSEI_WORKER_MEMORY`  | Memory per worker | `2GB`   |
| `SENSEI_WORKER_TIMEOUT` | Worker timeout    | `3600`  |
| `SENSEI_MAX_WORKERS`    | Maximum workers   | `32`    |

---

### Networking

| Variable           | Description          | Default   |
| ------------------ | -------------------- | --------- |
| `SENSEI_HOST`      | Listen host          | `0.0.0.0` |
| `SENSEI_PORT`      | Listen port          | `8080`    |
| `SENSEI_TLS_CERT`  | TLS certificate path | (none)    |
| `SENSEI_TLS_KEY`   | TLS key path         | (none)    |
| `SENSEI_PROXY_URL` | HTTP proxy URL       | (none)    |

---

### Feature Flags

| Variable                     | Description                  | Default |
| ---------------------------- | ---------------------------- | ------- |
| `SENSEI_FEATURE_CDC`         | Enable CDC                   | `true`  |
| `SENSEI_FEATURE_TIME_TRAVEL` | Enable Time-Travel Testing   | `true`  |
| `SENSEI_FEATURE_BEHAVIORAL`  | Enable behavioral validation | `true`  |
| `SENSEI_FEATURE_LOCAL_LLM`   | Enable local LLM             | `false` |

---

### Self-Hosted Only

| Variable                   | Description       | Default              |
| -------------------------- | ----------------- | -------------------- |
| `SENSEI_LICENSE_KEY`       | License key       | (required)           |
| `SENSEI_LICENSE_FILE`      | License file path | (alternative to key) |
| `SENSEI_TELEMETRY_ENABLED` | Send telemetry    | `true`               |
| `SENSEI_UPDATE_CHECK`      | Check for updates | `true`               |

---

### Example .env File

```bash
# Authentication
SENSEI_API_KEY=sk_live_abc123def456

# Database
SENSEI_DB_HOST=postgres.internal
SENSEI_DB_PASSWORD=secure_password

# Storage
SENSEI_S3_BUCKET=sensei-data
SENSEI_S3_REGION=us-west-2

# AI
OPENAI_API_KEY=sk-...

# Encryption
SENSEI_ENCRYPTION_KEY=base64_encoded_key

# Logging
SENSEI_LOG_LEVEL=info

# Workers
SENSEI_WORKER_COUNT=8
SENSEI_MAX_WORKERS=64
```

---

### Loading Environment Variables

**Docker:**

```bash
docker run --env-file .env sensei/sensei
```

**Kubernetes:**

```yaml
envFrom:
  - secretRef:
      name: sensei-secrets
  - configMapRef:
      name: sensei-config
```

**Shell:**

```bash
export SENSEI_API_KEY=sk_live_abc123
sensei migrate start
```

→ [Configuration Reference](configuration-reference.md)
→ [Self-Hosted Deployment](../../operations/deployment/self-hosted.md)
