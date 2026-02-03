# Authentication

## Securing Access to the Sensei Platform

All API requests require authentication. Sensei supports multiple authentication methods for different use cases.

---

### Authentication Methods

| Method               | Use Case                     | Lifespan                   |
| -------------------- | ---------------------------- | -------------------------- |
| **API Keys**         | Server-to-server integration | Long-lived (until revoked) |
| **OAuth 2.0**        | User-context applications    | Short-lived (1 hour)       |
| **Service Accounts** | CI/CD and automation         | Long-lived with rotation   |

---

### API Keys

API keys are the simplest authentication method for server-side integrations.

#### Creating an API Key

1. Navigate to **Settings → API Keys** in the dashboard
2. Click **Create API Key**
3. Provide a descriptive name (e.g., "Production CI/CD")
4. Select permissions scope
5. Copy the key immediately — it won't be shown again

#### Using API Keys

Include the key in the `Authorization` header:

```bash
curl -H "Authorization: Bearer sk_live_abc123def456" \
     https://api.sensei.ai/v1/migrations
```

Or with SDKs:

```python
from sensei import Sensei
client = Sensei(api_key="sk_live_abc123def456")
```

#### Key Types

| Prefix     | Environment | Access                              |
| ---------- | ----------- | ----------------------------------- |
| `sk_live_` | Production  | Full access to production resources |
| `sk_test_` | Sandbox     | Sandbox resources only, no billing  |

#### Key Permissions

Configure granular permissions when creating keys:

| Permission          | Grants Access To                   |
| ------------------- | ---------------------------------- |
| `migrations:read`   | View migrations and status         |
| `migrations:write`  | Create, update, delete migrations  |
| `connections:read`  | View saved connections             |
| `connections:write` | Create, update, delete connections |
| `webhooks:read`     | View webhook configurations        |
| `webhooks:write`    | Create, update, delete webhooks    |
| `billing:read`      | View usage and invoices            |
| `admin:*`           | Full administrative access         |

---

### OAuth 2.0

For applications acting on behalf of users (e.g., custom dashboards).

#### Authorization Flow

1. **Redirect user to authorization:**

```text
https://auth.sensei.ai/authorize?
  client_id=YOUR_CLIENT_ID&
  redirect_uri=https://yourapp.com/callback&
  response_type=code&
  scope=migrations:read+migrations:write&
  state=random_state_value
```

2. **User authorizes your application**

3. **Receive authorization code:**

```text
https://yourapp.com/callback?code=AUTH_CODE&state=random_state_value
```

4. **Exchange code for tokens:**

```bash
curl -X POST https://auth.sensei.ai/token \
  -d "grant_type=authorization_code" \
  -d "code=AUTH_CODE" \
  -d "client_id=YOUR_CLIENT_ID" \
  -d "client_secret=YOUR_CLIENT_SECRET" \
  -d "redirect_uri=https://yourapp.com/callback"
```

5. **Response:**

```json
{
  "access_token": "eyJhbG...",
  "token_type": "Bearer",
  "expires_in": 3600,
  "refresh_token": "rf_xyz...",
  "scope": "migrations:read migrations:write"
}
```

#### Refreshing Tokens

```bash
curl -X POST https://auth.sensei.ai/token \
  -d "grant_type=refresh_token" \
  -d "refresh_token=rf_xyz..." \
  -d "client_id=YOUR_CLIENT_ID" \
  -d "client_secret=YOUR_CLIENT_SECRET"
```

---

### Service Accounts

For automated systems that need long-lived credentials with automatic rotation.

#### Creating a Service Account

```bash
curl -X POST https://api.sensei.ai/v1/service-accounts \
  -H "Authorization: Bearer sk_live_admin_key" \
  -d '{
    "name": "GitHub Actions",
    "description": "CI/CD pipeline integration",
    "permissions": ["migrations:read", "migrations:write"]
  }'
```

Response:

```json
{
  "id": "sa_abc123",
  "name": "GitHub Actions",
  "credentials": {
    "client_id": "sa_abc123",
    "client_secret": "sas_xyz789...",
    "token_endpoint": "https://auth.sensei.ai/token"
  }
}
```

#### Using Service Accounts

Exchange credentials for an access token:

```bash
curl -X POST https://auth.sensei.ai/token \
  -d "grant_type=client_credentials" \
  -d "client_id=sa_abc123" \
  -d "client_secret=sas_xyz789..."
```

#### Automatic Key Rotation

Service accounts support automatic key rotation:

```bash
curl -X POST https://api.sensei.ai/v1/service-accounts/sa_abc123/rotate \
  -H "Authorization: Bearer sk_live_admin_key"
```

Old credentials remain valid for 24 hours after rotation to allow graceful transition.

---

### Security Best Practices

1. **Never expose keys client-side** — API keys should only be used in server-side code
2. **Use environment variables** — Don't hardcode keys in source code
3. **Rotate regularly** — Rotate production keys quarterly
4. **Use least privilege** — Grant only the permissions needed
5. **Monitor usage** — Set up alerts for unusual API activity
6. **Enable IP allowlisting** — Restrict key usage to known IPs (Enterprise)

---

### IP Allowlisting (Enterprise)

Restrict API key usage to specific IP addresses:

```bash
curl -X PATCH https://api.sensei.ai/v1/api-keys/sk_live_abc123 \
  -H "Authorization: Bearer sk_live_admin_key" \
  -d '{
    "allowed_ips": ["203.0.113.0/24", "198.51.100.42"]
  }'
```

Requests from non-allowed IPs receive `403 Forbidden`.

---

### Audit Logging

All authentication events are logged:

- Key creation and deletion
- Successful and failed authentication attempts
- Permission changes
- Key rotation events

Access audit logs in **Settings → Security → Audit Log** or via API:

```bash
curl https://api.sensei.ai/v1/audit-logs?type=authentication \
  -H "Authorization: Bearer sk_live_admin_key"
```

→ [Rate Limits](rate-limits.md)
→ [API Reference](api-reference.md)
→ [Integration Overview](README.md)
