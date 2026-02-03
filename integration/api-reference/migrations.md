# Migrations API

## Create, Manage, and Monitor Migrations

The Migrations API is the core of Sensei's functionality. Use it to create migrations, monitor progress, and control execution.

---

### Base URL

```text
https://api.sensei.ai/v1/migrations
```

---

## Endpoints

### Create Migration

Create a new migration from a source to a target system.

```text
POST /migrations
```

**Request Body:**

```json
{
  "name": "Q1 Database Modernization",
  "description": "Migrate legacy Oracle to PostgreSQL",
  "source": {
    "connection_id": "conn_oracle_prod",
    "schemas": ["HR", "FINANCE"],
    "exclude_tables": ["AUDIT_LOG", "TEMP_*"]
  },
  "target": {
    "connection_id": "conn_postgres_cloud",
    "schema": "public"
  },
  "options": {
    "mode": "full",
    "parallelism": 8,
    "batch_size": 10000,
    "auto_approve_above": 0.95,
    "generate_rollback": true
  },
  "schedule": {
    "start_at": "2026-02-15T02:00:00Z",
    "maintenance_window": {
      "start": "02:00",
      "end": "06:00",
      "timezone": "America/New_York"
    }
  },
  "notifications": {
    "email": ["team@company.com"],
    "slack_channel": "#data-ops",
    "on_events": ["completed", "failed", "approval_required"]
  }
}
```

**Response:**

```json
{
  "id": "mig_abc123def456",
  "name": "Q1 Database Modernization",
  "status": "pending",
  "created_at": "2026-02-02T15:30:00Z",
  "created_by": "user_xyz789",
  "source": {
    "connection_id": "conn_oracle_prod",
    "connection_name": "Oracle Production",
    "schemas": ["HR", "FINANCE"],
    "tables_count": 47
  },
  "target": {
    "connection_id": "conn_postgres_cloud",
    "connection_name": "PostgreSQL Cloud"
  },
  "plan": {
    "status": "generating",
    "estimated_completion": "2026-02-02T15:45:00Z"
  },
  "links": {
    "self": "/v1/migrations/mig_abc123def456",
    "plan": "/v1/migrations/mig_abc123def456/plan",
    "start": "/v1/migrations/mig_abc123def456/start",
    "logs": "/v1/migrations/mig_abc123def456/logs"
  }
}
```

---

### Get Migration

Retrieve details of a specific migration.

```text
GET /migrations/{id}
```

**Response:**

```json
{
  "id": "mig_abc123def456",
  "name": "Q1 Database Modernization",
  "status": "running",
  "phase": "data_transfer",
  "progress": {
    "overall_percent": 67,
    "current_phase": "data_transfer",
    "phases": [
      { "name": "planning", "status": "completed", "duration_seconds": 847 },
      { "name": "schema_migration", "status": "completed", "duration_seconds": 124 },
      { "name": "data_transfer", "status": "running", "progress_percent": 78 },
      { "name": "validation", "status": "pending" },
      { "name": "cutover", "status": "pending" }
    ]
  },
  "metrics": {
    "records_processed": 12453891,
    "records_total": 18500000,
    "bytes_transferred": "47.2 GB",
    "throughput_records_per_second": 8234,
    "errors_recovered": 127,
    "errors_escalated": 0
  },
  "timing": {
    "created_at": "2026-02-02T15:30:00Z",
    "started_at": "2026-02-02T16:00:00Z",
    "estimated_completion": "2026-02-02T18:30:00Z",
    "elapsed_seconds": 5400
  }
}
```

---

### List Migrations

List all migrations with optional filtering and pagination.

```text
GET /migrations
```

**Query Parameters:**

| Parameter        | Type     | Description                                                                                      |
| ---------------- | -------- | ------------------------------------------------------------------------------------------------ |
| `status`         | string   | Filter by status: `pending`, `planning`, `running`, `paused`, `completed`, `failed`, `cancelled` |
| `source_id`      | string   | Filter by source connection                                                                      |
| `target_id`      | string   | Filter by target connection                                                                      |
| `created_after`  | datetime | Migrations created after this time                                                               |
| `created_before` | datetime | Migrations created before this time                                                              |
| `limit`          | integer  | Results per page (default: 20, max: 100)                                                         |
| `offset`         | integer  | Pagination offset                                                                                |
| `sort`           | string   | Sort field: `created_at`, `started_at`, `name`                                                   |
| `order`          | string   | Sort order: `asc`, `desc`                                                                        |

**Example:**

```bash
curl -X GET "https://api.sensei.ai/v1/migrations?status=running&limit=10" \
  -H "Authorization: Bearer sk_live_abc123"
```

**Response:**

```json
{
  "data": [
    {
      "id": "mig_abc123",
      "name": "Q1 Database Modernization",
      "status": "running",
      "progress_percent": 67,
      "created_at": "2026-02-02T15:30:00Z"
    },
    {
      "id": "mig_def456",
      "name": "Analytics Warehouse Migration",
      "status": "running",
      "progress_percent": 23,
      "created_at": "2026-02-01T10:00:00Z"
    }
  ],
  "pagination": {
    "total": 42,
    "limit": 10,
    "offset": 0,
    "has_more": true
  }
}
```

---

### Start Migration

Begin execution of a planned migration.

```text
POST /migrations/{id}/start
```

**Request Body (optional):**

```json
{
  "skip_approval": false,
  "start_from_phase": "data_transfer",
  "dry_run": false
}
```

**Response:**

```json
{
  "id": "mig_abc123def456",
  "status": "running",
  "phase": "schema_migration",
  "started_at": "2026-02-02T16:00:00Z",
  "message": "Migration started successfully"
}
```

---

### Pause Migration

Pause a running migration at the next safe checkpoint.

```text
POST /migrations/{id}/pause
```

**Request Body (optional):**

```json
{
  "reason": "Maintenance window ending",
  "immediate": false
}
```

**Response:**

```json
{
  "id": "mig_abc123def456",
  "status": "pausing",
  "message": "Migration will pause at next checkpoint",
  "checkpoint_eta": "2026-02-02T17:15:00Z"
}
```

---

### Resume Migration

Resume a paused migration from its last checkpoint.

```text
POST /migrations/{id}/resume
```

**Response:**

```json
{
  "id": "mig_abc123def456",
  "status": "running",
  "resumed_at": "2026-02-02T18:00:00Z",
  "resumed_from_checkpoint": "ckpt_789xyz"
}
```

---

### Cancel Migration

Cancel a migration. For running migrations, initiates rollback if configured.

```text
POST /migrations/{id}/cancel
```

**Request Body:**

```json
{
  "reason": "Requirements changed",
  "rollback": true,
  "force": false
}
```

**Response:**

```json
{
  "id": "mig_abc123def456",
  "status": "cancelling",
  "rollback_status": "initiated",
  "message": "Migration cancellation in progress"
}
```

---

### Get Migration Plan

Retrieve the generated migration plan with mappings and transformations.

```text
GET /migrations/{id}/plan
```

**Response:**

```json
{
  "migration_id": "mig_abc123def456",
  "status": "approved",
  "generated_at": "2026-02-02T15:45:00Z",
  "summary": {
    "tables": 47,
    "columns": 892,
    "relationships": 63,
    "transformations": 234,
    "estimated_duration_seconds": 9000,
    "estimated_records": 18500000
  },
  "mappings": [
    {
      "source": { "schema": "HR", "table": "EMPLOYEES", "column": "EMP_ID" },
      "target": { "schema": "public", "table": "employees", "column": "employee_id" },
      "transformation": "CAST(EMP_ID AS BIGINT)",
      "confidence": 0.98,
      "status": "approved"
    }
  ],
  "phases": [
    {
      "name": "schema_migration",
      "order": 1,
      "tables": ["employees", "departments", "locations"],
      "parallel": true,
      "estimated_duration_seconds": 300
    }
  ],
  "risks": [
    {
      "severity": "medium",
      "type": "data_truncation",
      "description": "3 VARCHAR columns may be truncated",
      "affected": ["products.description"],
      "mitigation": "Increase target column size"
    }
  ]
}
```

---

### Approve Mapping

Approve or reject individual mappings that require human review.

```text
POST /migrations/{id}/plan/mappings/{mapping_id}/approve
```

**Request Body:**

```json
{
  "approved": true,
  "override_transformation": null,
  "comment": "Verified with business team"
}
```

---

### Get Migration Logs

Stream or retrieve migration logs.

```text
GET /migrations/{id}/logs
```

**Query Parameters:**

| Parameter | Type     | Description                                 |
| --------- | -------- | ------------------------------------------- |
| `level`   | string   | Filter: `debug`, `info`, `warning`, `error` |
| `phase`   | string   | Filter by phase name                        |
| `since`   | datetime | Logs after this timestamp                   |
| `follow`  | boolean  | Stream logs in real-time (SSE)              |
| `limit`   | integer  | Max log entries (default: 100)              |

**Example (streaming):**

```bash
curl -N "https://api.sensei.ai/v1/migrations/mig_abc123/logs?follow=true" \
  -H "Authorization: Bearer sk_live_abc123" \
  -H "Accept: text/event-stream"
```

**Response (SSE):**

```text
event: log
data: {"timestamp":"2026-02-02T16:30:00Z","level":"info","message":"Processing table: employees","phase":"data_transfer","table":"employees"}

event: log
data: {"timestamp":"2026-02-02T16:30:01Z","level":"info","message":"Batch 1/500 complete","phase":"data_transfer","records":10000}

event: progress
data: {"phase":"data_transfer","percent":45,"records_processed":4500000}
```

---

### Get Migration Metrics

Retrieve detailed performance metrics for a migration.

```text
GET /migrations/{id}/metrics
```

**Response:**

```json
{
  "migration_id": "mig_abc123def456",
  "snapshot_at": "2026-02-02T17:00:00Z",
  "throughput": {
    "current_records_per_second": 8234,
    "average_records_per_second": 7891,
    "peak_records_per_second": 12456,
    "current_bytes_per_second": "32.4 MB"
  },
  "resources": {
    "active_workers": 8,
    "cpu_utilization_percent": 67,
    "memory_utilization_percent": 45,
    "network_utilization_percent": 78
  },
  "quality": {
    "error_rate_percent": 0.001,
    "recovery_success_rate_percent": 99.2,
    "validation_pass_rate_percent": 99.8
  },
  "by_table": [
    {
      "table": "orders",
      "records_total": 5000000,
      "records_processed": 3750000,
      "throughput_records_per_second": 4500,
      "errors": 12
    }
  ]
}
```

---

### Delete Migration

Delete a completed, failed, or cancelled migration and its associated data.

```sql
DELETE /migrations/{id}
```

**Query Parameters:**

| Parameter      | Type    | Description                            |
| -------------- | ------- | -------------------------------------- |
| `keep_logs`    | boolean | Retain logs for audit (default: true)  |
| `keep_metrics` | boolean | Retain metrics history (default: true) |

**Response:**

```json
{
  "id": "mig_abc123def456",
  "deleted": true,
  "logs_retained": true,
  "metrics_retained": true
}
```

---

## Migration Statuses

| Status         | Description                                    |
| -------------- | ---------------------------------------------- |
| `pending`      | Migration created, awaiting plan generation    |
| `planning`     | Plan being generated by Scout/Architect agents |
| `plan_ready`   | Plan generated, awaiting review/approval       |
| `approved`     | Plan approved, ready to start                  |
| `running`      | Migration actively executing                   |
| `pausing`      | Migration stopping at next checkpoint          |
| `paused`       | Migration paused, can be resumed               |
| `validating`   | Post-migration validation in progress          |
| `completed`    | Migration finished successfully                |
| `failed`       | Migration failed with unrecoverable error      |
| `cancelled`    | Migration cancelled by user                    |
| `rolling_back` | Rollback in progress                           |
| `rolled_back`  | Rollback completed                             |

---

## Error Codes

| Code                  | Description                                  |
| --------------------- | -------------------------------------------- |
| `migration_not_found` | Migration ID does not exist                  |
| `invalid_state`       | Operation not allowed in current state       |
| `plan_not_ready`      | Cannot start migration without approved plan |
| `source_unreachable`  | Cannot connect to source system              |
| `target_unreachable`  | Cannot connect to target system              |
| `approval_required`   | Mappings require manual approval             |
| `rollback_failed`     | Rollback operation failed                    |

→ [Connections API](connections.md)
→ [Validation API](validation.md)
→ [Webhooks](../webhooks.md)
