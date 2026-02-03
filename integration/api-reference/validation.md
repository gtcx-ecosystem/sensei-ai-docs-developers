# Validation API

## Verify Migration Correctness

The Validation API triggers KORA verification processes, retrieves validation results, and generates Behavioral Equivalence Certificates.

---

### Base URL

```text
https://api.sensei.ai/v1/validations
```

---

## Endpoints

### Run Validation

Trigger validation on a completed migration.

```text
POST /validations
```

**Request Body:**

```json
{
  "migration_id": "mig_abc123def456",
  "type": "full",
  "options": {
    "structural": true,
    "statistical": true,
    "referential": true,
    "semantic": true,
    "behavioral": true,
    "time_travel_checkpoints": 5
  }
}
```

**Validation Types:**

| Type              | Description                               | Duration   |
| ----------------- | ----------------------------------------- | ---------- |
| `quick`           | Row counts and spot checks                | Minutes    |
| `standard`        | Structural + statistical + referential    | Hours      |
| `full`            | All validation types including behavioral | Hours-Days |
| `behavioral_only` | Time-Travel Testing only                  | Hours      |

**Response:**

```json
{
  "id": "val_xyz789",
  "migration_id": "mig_abc123def456",
  "status": "running",
  "type": "full",
  "started_at": "2026-02-02T18:00:00Z",
  "estimated_completion": "2026-02-02T22:00:00Z"
}
```

---

### Get Validation Results

```text
GET /validations/{id}
```

**Response:**

```json
{
  "id": "val_xyz789",
  "migration_id": "mig_abc123def456",
  "status": "completed",
  "overall_result": "passed",
  "confidence_score": 0.987,
  "results": {
    "structural": {
      "status": "passed",
      "tables_checked": 47,
      "columns_checked": 892,
      "issues": []
    },
    "statistical": {
      "status": "passed",
      "distributions_compared": 892,
      "max_deviation": 0.002,
      "issues": []
    },
    "referential": {
      "status": "passed",
      "relationships_verified": 63,
      "orphan_records": 0
    },
    "semantic": {
      "status": "passed",
      "rules_evaluated": 156,
      "violations": 0
    },
    "behavioral": {
      "status": "passed",
      "checkpoints_replayed": 5,
      "queries_replayed": 12456,
      "exact_matches": 12398,
      "semantic_matches": 58,
      "mismatches": 0
    }
  },
  "certificate_id": "cert_abc123",
  "completed_at": "2026-02-02T21:45:00Z"
}
```

---

### List Validations

```text
GET /validations
```

**Query Parameters:**

| Parameter      | Type   | Description                                 |
| -------------- | ------ | ------------------------------------------- |
| `migration_id` | string | Filter by migration                         |
| `status`       | string | `pending`, `running`, `completed`, `failed` |
| `result`       | string | `passed`, `failed`, `partial`               |

---

### Get Validation Report

Generate a detailed validation report.

```text
GET /validations/{id}/report
```

**Query Parameters:**

| Parameter         | Type    | Description               |
| ----------------- | ------- | ------------------------- |
| `format`          | string  | `json`, `pdf`, `html`     |
| `include_details` | boolean | Include row-level details |

---

### Get Certificate

Retrieve Behavioral Equivalence Certificate.

```text
GET /validations/{id}/certificate
```

**Response:**

```json
{
  "id": "cert_abc123",
  "validation_id": "val_xyz789",
  "migration_id": "mig_abc123def456",
  "status": "valid",
  "issued_at": "2026-02-02T21:45:00Z",
  "valid_until": "2027-02-02T21:45:00Z",
  "signature": "sha256:abc123...",
  "attestations": {
    "data_integrity": true,
    "schema_equivalence": true,
    "behavioral_equivalence": true,
    "audit_trail_complete": true
  },
  "download_url": "https://api.sensei.ai/v1/certificates/cert_abc123/download"
}
```

---

### Custom Validation Rules

```text
POST /validations/rules
```

**Request Body:**

```json
{
  "name": "salary_range_check",
  "description": "Verify salaries are within expected range",
  "type": "sql",
  "rule": "SELECT COUNT(*) FROM employees WHERE salary < 0 OR salary > 10000000",
  "expected": 0,
  "severity": "error"
}
```

---

## Validation Statuses

| Status      | Description                  |
| ----------- | ---------------------------- |
| `pending`   | Validation queued            |
| `running`   | Validation in progress       |
| `completed` | Validation finished          |
| `failed`    | Validation encountered error |
| `cancelled` | Validation cancelled         |

## Result Codes

| Result                 | Description              |
| ---------------------- | ------------------------ |
| `passed`               | All checks passed        |
| `passed_with_warnings` | Passed with minor issues |
| `failed`               | Critical issues found    |
| `partial`              | Some checks incomplete   |

→ [KORA Documentation](../../../product/platform/kora/README.md)
→ [Time-Travel Testing](../../../product/platform/kora/time-travel-testing.md)
→ [Migrations API](migrations.md)
