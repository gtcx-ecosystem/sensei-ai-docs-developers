---
description: System limits and quotas
---

# Limits and Quotas

## Platform Limits by Plan

Reference for Sensei platform limits and quotas.

---

### Data Volume

| Limit               | Starter | Professional | Enterprise |
| ------------------- | ------- | ------------ | ---------- |
| Monthly data volume | 100 GB  | 1 TB         | Unlimited  |
| Per-migration limit | 50 GB   | 500 GB       | Unlimited  |
| Overage rate        | $15/GB  | $10/GB       | Negotiated |

---

### Migrations

| Limit                 | Starter | Professional | Enterprise |
| --------------------- | ------- | ------------ | ---------- |
| Concurrent migrations | 2       | 10           | Unlimited  |
| Migrations per month  | 20      | 100          | Unlimited  |
| Tables per migration  | 500     | 2,000        | Unlimited  |
| Columns per table     | 1,000   | 1,000        | 1,000      |

---

### Connections

| Limit                    | Starter | Professional | Enterprise |
| ------------------------ | ------- | ------------ | ---------- |
| Saved connections        | 10      | 50           | Unlimited  |
| Connection pool size     | 5       | 20           | Custom     |
| Simultaneous connections | 10      | 100          | Custom     |

---

### Workers

| Limit                    | Starter | Professional | Enterprise |
| ------------------------ | ------- | ------------ | ---------- |
| Workers per migration    | 4       | 16           | 64         |
| Total concurrent workers | 8       | 64           | Custom     |
| Worker memory            | 2 GB    | 4 GB         | Custom     |

---

### API

| Limit                   | Starter | Professional | Enterprise |
| ----------------------- | ------- | ------------ | ---------- |
| API requests/day        | 10,000  | 100,000      | Custom     |
| API requests/minute     | 100     | 500          | Custom     |
| Webhook endpoints       | 5       | 100          | Unlimited  |
| Webhook deliveries/hour | 1,000   | 10,000       | Unlimited  |

---

### Validation

| Limit                   | Starter | Professional | Enterprise |
| ----------------------- | ------- | ------------ | ---------- |
| Validation types        | Basic   | All          | All        |
| Time-Travel checkpoints | N/A     | 10           | 100        |
| Custom validation rules | N/A     | 50           | Unlimited  |
| Validation history      | 30 days | 90 days      | Custom     |

---

### Storage & Retention

| Limit                 | Starter | Professional | Enterprise |
| --------------------- | ------- | ------------ | ---------- |
| Checkpoint retention  | 7 days  | 30 days      | Custom     |
| Log retention         | 7 days  | 30 days      | 1 year+    |
| Certificate retention | 90 days | 1 year       | 7 years    |
| Metadata retention    | 90 days | 1 year       | Custom     |

---

### Users & Access

| Limit    | Starter   | Professional | Enterprise |
| -------- | --------- | ------------ | ---------- |
| Users    | 5         | 25           | Unlimited  |
| API keys | 5         | 25           | Unlimited  |
| Roles    | 3 (fixed) | 10           | Custom     |
| SSO      | No        | Yes          | Yes        |

---

### Technical Limits

| Limit                      | Value     | Notes             |
| -------------------------- | --------- | ----------------- |
| Maximum row size           | 100 MB    | Including LOBs    |
| Maximum column count       | 1,000     | Per table         |
| Maximum table name length  | 128 chars |                   |
| Maximum column name length | 128 chars |                   |
| Maximum batch size         | 100,000   | Records per batch |
| Maximum query timeout      | 3,600 s   | 1 hour            |
| Maximum file upload        | 5 GB      | For configuration |
| Maximum webhook payload    | 1 MB      |                   |

---

### Rate Limits

| Endpoint             | Limit    | Window     |
| -------------------- | -------- | ---------- |
| All API endpoints    | Per plan | Per day    |
| Authentication       | 100      | Per minute |
| Migration create     | 10       | Per minute |
| Migration start      | 20       | Per minute |
| Status polling       | 60       | Per minute |
| Webhook registration | 10       | Per minute |

---

### Soft vs Hard Limits

**Soft limits** (can be temporarily exceeded):

- Monthly data volume (with overage charges)
- API requests (with rate limiting)
- Concurrent workers (may queue)

**Hard limits** (cannot be exceeded):

- Maximum row size
- Maximum column count
- Tables per migration (per plan)

---

### Requesting Limit Increases

**Professional plan:** Contact support for temporary increases.

**Enterprise plan:** Custom limits negotiated in contract.

**Request process:**

1. Email support@sensei.ai
2. Describe use case
3. Specify limits needed
4. Timeline required

---

### Monitoring Usage

**Dashboard:** Settings → Usage shows current consumption.

**API:**

```bash
curl -H "Authorization: Bearer $API_KEY" \
  https://api.sensei.ai/v1/usage
```

**Alerts:** Configure alerts at 80% and 100% of limits.

→ [Pricing](../../enterprise/business/pricing/) → [Enterprise](../../enterprise/business/compliance/enterprise.md)
