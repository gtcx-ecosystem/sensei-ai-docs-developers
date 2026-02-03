# API Reference

## Sensei REST API v1

The Sensei API provides programmatic access to all platform capabilities: database connections, migration execution, plan review, validation, and verification certificates.

---

### Base URL

```text
https://api.sensei.ai/v1
```

For self-hosted deployments, replace the hostname with your instance address.

---

### Authentication

All requests must include a valid credential:

| Method          | Header                              | Use Case                |
| --------------- | ----------------------------------- | ----------------------- |
| API Key         | `Authorization: Bearer sk_live_...` | Server-to-server        |
| OAuth 2.0       | `Authorization: Bearer eyJ...`      | User-authenticated apps |
| Service Account | `Authorization: Bearer sa_...`      | CI/CD pipelines         |

```bash
curl -X GET https://api.sensei.ai/v1/migrations \
  -H "Authorization: Bearer sk_live_abc123def456"
```

Keys use `sk_live_` for production and `sk_test_` for sandbox.

> [Authentication Reference](authentication.md) -- key management, OAuth flows, scopes, token lifecycle

---

### Common Headers

| Header              | Required       | Description                                         |
| ------------------- | -------------- | --------------------------------------------------- |
| `Authorization`     | Yes            | Bearer token                                        |
| `Content-Type`      | POST/PUT/PATCH | `application/json`                                  |
| `Accept`            | No             | `application/json` (default) or `text/event-stream` |
| `X-Request-Id`      | No             | Client UUID for tracing                             |
| `X-Idempotency-Key` | No             | At-most-once POST semantics                         |

---

### Rate Limits

| Tier         | Requests/Second | Requests/Day |
| ------------ | --------------- | ------------ |
| Free         | 10              | 1,000        |
| Starter      | 50              | 50,000       |
| Professional | 200             | 500,000      |
| Enterprise   | Custom          | Custom       |

```text
X-RateLimit-Limit: 200
X-RateLimit-Remaining: 187
X-RateLimit-Reset: 1706889600
```

HTTP 429 includes a `Retry-After` header.

---

### Pagination

| Parameter | Type    | Default | Description                 |
| --------- | ------- | ------- | --------------------------- |
| `limit`   | integer | 20      | Results per page (max: 100) |
| `offset`  | integer | 0       | Results to skip             |

```json
{
  "data": [],
  "pagination": { "total": 142, "limit": 20, "offset": 0, "has_more": true }
}
```

---

### Error Format

```json
{
  "error": {
    "code": "migration_not_found",
    "message": "No migration exists with ID mig_invalid",
    "documentation_url": "https://docs.sensei.ai/integration/api-reference/migrations",
    "request_id": "req_abc123"
  }
}
```

| Status | Meaning                  |
| ------ | ------------------------ |
| 400    | Malformed request        |
| 401    | Invalid authentication   |
| 403    | Insufficient permissions |
| 404    | Resource not found       |
| 409    | Conflict                 |
| 422    | Validation error         |
| 429    | Rate limit exceeded      |
| 500    | Internal server error    |

---

### Versioning

Versioned via URL path (`/v1`). Breaking changes only in new major versions. Current version supported for 24 months after any successor. Non-breaking additions (new fields, optional parameters) are added without version bump.

---

### SDKs

| Language   | Package          | Installation                       |
| ---------- | ---------------- | ---------------------------------- |
| Python     | `sensei-sdk`     | `pip install sensei-sdk`           |
| TypeScript | `@sensei-ai/sdk` | `npm install @sensei-ai/sdk`       |
| Go         | `sensei-go`      | `go get github.com/gtcx/sensei-go` |
| Rust       | `sensei-rs`      | `cargo add sensei-rs`              |

---

### Endpoints

| Resource                            | Path                       | Description                |
| ----------------------------------- | -------------------------- | -------------------------- |
| [Authentication](authentication.md) | `/v1/api-keys`, `/oauth/*` | Keys, OAuth, tokens        |
| [Connections](connections.md)       | `/v1/connections`          | Database registration      |
| [Migrations](migrations.md)         | `/v1/migrations`           | Execution, monitoring      |
| [Validation](validation.md)         | `/v1/validations`          | Verification, certificates |
| [Webhooks](../webhooks.md)          | `/v1/webhooks`             | Event subscriptions        |
