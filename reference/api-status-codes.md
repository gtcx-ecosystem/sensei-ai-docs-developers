# API Status Codes

## HTTP Response Codes

Reference for HTTP status codes returned by the Sensei API.

---

### Success Codes (2xx)

| Code | Status     | Description                     | Common Use                    |
| ---- | ---------- | ------------------------------- | ----------------------------- |
| 200  | OK         | Request successful              | GET requests, updates         |
| 201  | Created    | Resource created                | POST creating new resource    |
| 202  | Accepted   | Request accepted for processing | Async operations (migrations) |
| 204  | No Content | Success, no body                | DELETE operations             |

---

### Client Error Codes (4xx)

| Code | Status                 | Description                | Solution                       |
| ---- | ---------------------- | -------------------------- | ------------------------------ |
| 400  | Bad Request            | Invalid request format     | Check request body, parameters |
| 401  | Unauthorized           | Invalid or missing API key | Check API key                  |
| 403  | Forbidden              | Insufficient permissions   | Check API key permissions      |
| 404  | Not Found              | Resource doesn't exist     | Check resource ID              |
| 405  | Method Not Allowed     | HTTP method not supported  | Check allowed methods          |
| 409  | Conflict               | Resource state conflict    | Refresh and retry              |
| 413  | Payload Too Large      | Request body too large     | Reduce payload size            |
| 415  | Unsupported Media Type | Wrong Content-Type         | Use application/json           |
| 422  | Unprocessable Entity   | Validation failed          | Check input values             |
| 429  | Too Many Requests      | Rate limit exceeded        | Slow down, check limits        |

---

### Server Error Codes (5xx)

| Code | Status                | Description             | Solution                             |
| ---- | --------------------- | ----------------------- | ------------------------------------ |
| 500  | Internal Server Error | Unexpected error        | Retry, contact support if persistent |
| 502  | Bad Gateway           | Upstream error          | Retry after brief wait               |
| 503  | Service Unavailable   | Maintenance or overload | Retry with backoff                   |
| 504  | Gateway Timeout       | Upstream timeout        | Retry, check status page             |

---

### Response Body Format

All error responses include a standard body:

```json
{
  "error": {
    "code": "E7003",
    "message": "Invalid request: missing required field 'name'",
    "details": {
      "field": "name",
      "reason": "required"
    },
    "request_id": "req_abc123def456"
  }
}
```

| Field        | Description                                           |
| ------------ | ----------------------------------------------------- |
| `code`       | Sensei error code (see [Error Codes](error-codes.md)) |
| `message`    | Human-readable description                            |
| `details`    | Additional context (varies by error)                  |
| `request_id` | Unique ID for support reference                       |

---

### Rate Limiting

When rate limited (429), response includes headers:

```text
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1677849600
Retry-After: 60
```

| Header                  | Description                      |
| ----------------------- | -------------------------------- |
| `X-RateLimit-Limit`     | Requests allowed per window      |
| `X-RateLimit-Remaining` | Requests remaining               |
| `X-RateLimit-Reset`     | Unix timestamp when limit resets |
| `Retry-After`           | Seconds to wait before retry     |

---

### Pagination

List endpoints return pagination info:

```json
{
  "data": [...],
  "pagination": {
    "total": 150,
    "page": 1,
    "per_page": 50,
    "total_pages": 3,
    "has_next": true,
    "has_prev": false
  }
}
```

---

### Async Operations

Long-running operations return 202 Accepted:

```json
{
  "id": "mig_abc123",
  "status": "pending",
  "status_url": "/v1/migrations/mig_abc123"
}
```

Poll `status_url` to check progress.

→ [API Reference](../integration/api-reference/README.md)
→ [Error Codes](error-codes.md)
