# Authentication

## Securing Your Sensei Integration

Sensei uses industry-standard authentication methods to secure API access. All requests must be authenticated, and all endpoints require HTTPS.

---

### Authentication Methods

#### API Keys

API keys are the simplest way to authenticate. They're ideal for server-to-server integrations where you can securely store the key.

```bash
curl -X GET https://api.sensei.ai/v1/migrations \
  -H "Authorization: Bearer sk_live_abc123def456"
```

**Key Types:**

| Key Prefix | Environment | Use Case                       |
| ---------- | ----------- | ------------------------------ |
| `sk_live_` | Production  | Live migrations with real data |
| `sk_test_` | Sandbox     | Development and testing        |

**Creating API Keys:**

1. Navigate to **Settings → API Keys** in the dashboard
2. Click **Create New Key**
3. Select key type (Live or Test)
4. Set permissions (see scopes below)
5. Copy the key immediately — it won't be shown again

```bash
# Create key via API (requires admin privileges)
curl -X POST https://api.sensei.ai/v1/api-keys \
  -H "Authorization: Bearer sk_live_admin_key" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "CI/CD Pipeline Key",
    "scopes": ["migrations:read", "migrations:write"],
    "expires_at": "2027-01-01T00:00:00Z"
  }'
```

---

#### OAuth 2.0

OAuth 2.0 is recommended for applications where users authenticate with their own Sensei credentials. Supports Authorization Code and Client Credentials flows.

**Authorization Code Flow (User Authentication):**

```text
1. Redirect user to authorization URL
   https://auth.sensei.ai/oauth/authorize?
     client_id=YOUR_CLIENT_ID&
     redirect_uri=https://your-app.com/callback&
     response_type=code&
     scope=migrations:read+migrations:write&
     state=RANDOM_STATE

2. User authenticates and approves scopes

3. Sensei redirects to your callback with code
   https://your-app.com/callback?code=AUTH_CODE&state=RANDOM_STATE

4. Exchange code for tokens
   POST https://auth.sensei.ai/oauth/token
   {
     "grant_type": "authorization_code",
     "code": "AUTH_CODE",
     "client_id": "YOUR_CLIENT_ID",
     "client_secret": "YOUR_CLIENT_SECRET",
     "redirect_uri": "https://your-app.com/callback"
   }

5. Receive access token and refresh token
   {
     "access_token": "eyJ...",
     "token_type": "Bearer",
     "expires_in": 3600,
     "refresh_token": "rt_abc123",
     "scope": "migrations:read migrations:write"
   }
```

**Client Credentials Flow (Machine-to-Machine):**

```bash
curl -X POST https://auth.sensei.ai/oauth/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=client_credentials" \
  -d "client_id=YOUR_CLIENT_ID" \
  -d "client_secret=YOUR_CLIENT_SECRET" \
  -d "scope=migrations:read migrations:write"
```

---

#### Service Accounts

Service accounts are special API identities for automated systems. They have their own permissions and audit trails, separate from user accounts.

```yaml
# service-account.yaml
name: ci-pipeline
description: 'GitHub Actions deployment pipeline'
scopes:
  - migrations:read
  - migrations:write
  - connections:read
ip_whitelist:
  - 192.30.252.0/22 # GitHub Actions
  - 185.199.108.0/22
```

```bash
# Create service account
curl -X POST https://api.sensei.ai/v1/service-accounts \
  -H "Authorization: Bearer sk_live_admin_key" \
  -H "Content-Type: application/json" \
  -d @service-account.yaml
```

---

### Permission Scopes

| Scope               | Description                              |
| ------------------- | ---------------------------------------- |
| `migrations:read`   | View migrations and their status         |
| `migrations:write`  | Create, start, pause, cancel migrations  |
| `connections:read`  | View database connections                |
| `connections:write` | Create and modify connections            |
| `validation:read`   | View validation results and certificates |
| `validation:write`  | Run validations, configure rules         |
| `webhooks:read`     | View webhook configurations              |
| `webhooks:write`    | Create and modify webhooks               |
| `admin:read`        | View organization settings               |
| `admin:write`       | Modify organization settings             |
| `billing:read`      | View usage and invoices                  |
| `billing:write`     | Modify billing settings                  |

**Wildcard Scopes:**

- `migrations:*` — All migration permissions
- `*:read` — Read-only access to all resources
- `*` — Full access (admin only)

---

### Token Management

**Refreshing Access Tokens:**

```bash
curl -X POST https://auth.sensei.ai/oauth/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=refresh_token" \
  -d "refresh_token=rt_abc123" \
  -d "client_id=YOUR_CLIENT_ID" \
  -d "client_secret=YOUR_CLIENT_SECRET"
```

**Revoking Tokens:**

```bash
curl -X POST https://auth.sensei.ai/oauth/revoke \
  -H "Authorization: Bearer sk_live_abc123" \
  -d "token=ACCESS_TOKEN_TO_REVOKE"
```

**Token Introspection:**

```bash
curl -X POST https://auth.sensei.ai/oauth/introspect \
  -H "Authorization: Bearer sk_live_abc123" \
  -d "token=TOKEN_TO_CHECK"
```

---

### Security Best Practices

#### Key Rotation

Rotate API keys regularly and immediately if compromised:

```bash
# Create new key
NEW_KEY=$(curl -X POST https://api.sensei.ai/v1/api-keys \
  -H "Authorization: Bearer sk_live_current_key" \
  -d '{"name": "Rotated Key", "scopes": ["migrations:*"]}' \
  | jq -r '.key')

# Update your application to use NEW_KEY

# Delete old key
curl -X DELETE https://api.sensei.ai/v1/api-keys/old_key_id \
  -H "Authorization: Bearer sk_live_admin_key"
```

#### IP Whitelisting

Restrict API access to known IP addresses:

```bash
curl -X PATCH https://api.sensei.ai/v1/api-keys/key_id \
  -H "Authorization: Bearer sk_live_admin_key" \
  -H "Content-Type: application/json" \
  -d '{
    "ip_whitelist": ["203.0.113.0/24", "198.51.100.0/24"]
  }'
```

#### Storing Secrets

**Never:**

- Commit API keys to version control
- Log API keys in application logs
- Expose keys in client-side code
- Share keys via email or chat

**Do:**

- Use environment variables or secret managers
- Use different keys for development and production
- Set minimal required scopes
- Enable audit logging

```python
import os
from sensei import Sensei

# Load from environment
client = Sensei(api_key=os.environ["SENSEI_API_KEY"])
```

---

### Rate Limiting

API requests are rate-limited to ensure fair usage:

| Tier         | Requests/Second | Requests/Day |
| ------------ | --------------- | ------------ |
| Free         | 10              | 1,000        |
| Starter      | 50              | 50,000       |
| Professional | 200             | 500,000      |
| Enterprise   | Custom          | Custom       |

**Rate Limit Headers:**

```text
X-RateLimit-Limit: 200
X-RateLimit-Remaining: 187
X-RateLimit-Reset: 1706889600
```

**Handling Rate Limits:**

```python
import time

def api_request_with_retry(url, headers, max_retries=3):
    for attempt in range(max_retries):
        response = requests.get(url, headers=headers)

        if response.status_code == 429:
            retry_after = int(response.headers.get('Retry-After', 60))
            time.sleep(retry_after)
            continue

        return response

    raise Exception("Rate limit exceeded after retries")
```

---

### Error Responses

Authentication errors return standard HTTP status codes:

| Status | Error          | Description                             |
| ------ | -------------- | --------------------------------------- |
| 401    | `unauthorized` | Missing or invalid authentication       |
| 403    | `forbidden`    | Valid auth but insufficient permissions |
| 429    | `rate_limited` | Too many requests                       |

```json
{
  "error": {
    "code": "unauthorized",
    "message": "Invalid API key provided",
    "documentation_url": "https://docs.sensei.ai/authentication"
  }
}
```

---

### Audit Logging

All API requests are logged for security and compliance:

```bash
# View recent API activity
curl -X GET "https://api.sensei.ai/v1/audit-logs?limit=50" \
  -H "Authorization: Bearer sk_live_admin_key"
```

```json
{
  "data": [
    {
      "id": "log_abc123",
      "timestamp": "2026-02-02T14:30:00Z",
      "actor": {
        "type": "api_key",
        "id": "key_xyz789",
        "name": "CI Pipeline Key"
      },
      "action": "migrations.create",
      "resource": "mig_def456",
      "ip_address": "203.0.113.42",
      "user_agent": "sensei-sdk/1.0.0",
      "result": "success"
    }
  ]
}
```

→ [API Reference Overview](README.md)
→ [Migrations API](migrations.md)
→ [SDKs](../sdks.md)
