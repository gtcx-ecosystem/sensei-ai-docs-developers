# Connections API

## Manage Database Connections

The Connections API lets you securely configure and manage connections to source and target databases. Credentials are encrypted at rest and never exposed in API responses.

---

### Base URL

```text
https://api.sensei.ai/v1/connections
```

---

## Endpoints

### Create Connection

Register a new database connection.

```text
POST /connections
```

**Request Body:**

```json
{
  "name": "Oracle Production",
  "description": "Main Oracle database - HR and Finance schemas",
  "type": "oracle",
  "role": "source",
  "config": {
    "host": "oracle-prod.company.internal",
    "port": 1521,
    "service_name": "ORCLPDB1",
    "username": "sensei_reader",
    "password": "secure_password_here",
    "schemas": ["HR", "FINANCE"]
  },
  "options": {
    "ssl_mode": "verify-full",
    "connection_timeout_seconds": 30,
    "query_timeout_seconds": 300,
    "max_connections": 10
  },
  "tags": ["production", "oracle", "hr-data"]
}
```

**Supported Connection Types:**

| Type         | Description                    |
| ------------ | ------------------------------ |
| `postgresql` | PostgreSQL 11+                 |
| `mysql`      | MySQL 5.7+ / MariaDB 10.3+     |
| `oracle`     | Oracle 12c+                    |
| `sqlserver`  | SQL Server 2016+               |
| `mongodb`    | MongoDB 4.4+                   |
| `snowflake`  | Snowflake Data Cloud           |
| `bigquery`   | Google BigQuery                |
| `redshift`   | Amazon Redshift                |
| `databricks` | Databricks SQL                 |
| `s3`         | Amazon S3 (CSV, Parquet, JSON) |

**Response:**

```json
{
  "id": "conn_abc123def456",
  "name": "Oracle Production",
  "type": "oracle",
  "role": "source",
  "status": "testing",
  "created_at": "2026-02-02T15:00:00Z"
}
```

---

### Get Connection

```text
GET /connections/{id}
```

### List Connections

```text
GET /connections
```

### Update Connection

```text
PATCH /connections/{id}
```

### Delete Connection

```sql
DELETE /connections/{id}
```

### Test Connection

```text
POST /connections/{id}/test
```

### Get Connection Schema

```text
GET /connections/{id}/schema
```

→ [Migrations API](migrations.md)
→ [Validation API](validation.md)
