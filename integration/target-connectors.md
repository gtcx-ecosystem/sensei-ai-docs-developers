# Target Connectors

## Connecting to Your Target Databases

Target connectors are optimized for write performance — bulk loading, batch inserts, and efficient index management. This page covers supported targets, configuration, and write optimization.

---

### Supported Targets

#### Relational Databases

| Database       | Versions | Bulk Load Method | Write Optimization                 |
| -------------- | -------- | ---------------- | ---------------------------------- |
| **PostgreSQL** | 12+      | COPY             | Parallel workers, deferred indexes |
| **MySQL**      | 8.0+     | LOAD DATA        | Bulk insert, disabled keys         |
| **Oracle**     | 19c+     | SQL\*Loader      | Direct path, parallel DML          |
| **SQL Server** | 2019+    | BULK INSERT      | Minimal logging, tablock           |
| **SQLite**     | 3.x      | Batch INSERT     | Deferred transactions              |
| **MariaDB**    | 10.5+    | LOAD DATA        | Bulk insert                        |

#### Cloud Data Warehouses

| Platform          | Bulk Load Method | Notes                       |
| ----------------- | ---------------- | --------------------------- |
| **Snowflake**     | COPY INTO        | Stage files to S3/Azure/GCS |
| **BigQuery**      | Load jobs        | Streaming or batch mode     |
| **Redshift**      | COPY             | From S3 with manifest       |
| **Azure Synapse** | COPY INTO        | From ADLS or Blob           |
| **Databricks**    | COPY INTO        | Delta Lake format           |

#### Document Stores

| Database          | Write Method                  |
| ----------------- | ----------------------------- |
| **MongoDB**       | Bulk write with ordered=false |
| **Elasticsearch** | Bulk API                      |
| **CouchDB**       | Bulk docs API                 |

---

### Connection Configuration

#### Standard Configuration

```yaml
target:
  type: postgresql
  connection_string: 'postgresql://user:password@host:5432/database'
  schema: migrated_data
  options:
    create_schema_if_missing: true
    drop_existing_tables: false
```

#### Cloud Data Warehouse Configuration

**Snowflake:**

```yaml
target:
  type: snowflake
  account: xy12345.us-east-1
  warehouse: MIGRATION_WH
  database: PROD_DB
  schema: MIGRATED
  credentials: vault://secrets/snowflake
  options:
    stage: '@MIGRATION_STAGE'
    file_format: 'MIGRATION_CSV'
    warehouse_size: MEDIUM
    auto_suspend: 300
```

**BigQuery:**

```yaml
target:
  type: bigquery
  project: my-project
  dataset: migrated_data
  credentials: vault://secrets/bigquery-sa
  options:
    location: US
    write_disposition: WRITE_TRUNCATE
    create_disposition: CREATE_IF_NEEDED
```

---

### Write Optimization

#### PostgreSQL Optimization

```yaml
target:
  type: postgresql
  # ... connection details ...
  options:
    # Bulk loading
    use_copy: true
    copy_buffer_size: 65536

    # Index management
    defer_indexes: true # Create indexes after data load
    defer_constraints: true # Enable constraints after load

    # Parallelism
    parallel_workers: 4

    # Logging
    unlogged_tables: false # Set true for temporary speed boost
    synchronous_commit: false # Faster writes, slight durability risk
```

**Performance impact:**

- `use_copy`: 5-10x faster than INSERT
- `defer_indexes`: 2-3x faster overall
- `parallel_workers`: Near-linear scaling to ~8 workers

#### Snowflake Optimization

```yaml
target:
  type: snowflake
  # ... connection details ...
  options:
    # Staging
    stage: '@MIGRATION_STAGE'
    stage_region: us-east-1 # Match data location
    compression: GZIP

    # Loading
    file_format: 'MIGRATION_PARQUET'
    on_error: CONTINUE # Don't fail entire load on bad records
    size_limit: 5368709120 # 5GB per file

    # Warehouse
    warehouse_size: XLARGE # Scale up during migration
    auto_resume: true

    # Clustering
    auto_clustering: false # Enable after load
```

#### BigQuery Optimization

```yaml
target:
  type: bigquery
  # ... connection details ...
  options:
    # Loading method
    write_method: load_job # vs streaming (cheaper, higher latency)
    source_format: PARQUET # More efficient than CSV

    # Partitioning
    time_partitioning:
      field: created_at
      type: DAY

    # Clustering
    clustering_fields: ['customer_id', 'region']

    # Job config
    max_bad_records: 100
    allow_jagged_rows: true
```

---

### Schema Management

#### Schema Creation Options

| Option                           | Behavior                          |
| -------------------------------- | --------------------------------- |
| `create_schema_if_missing: true` | Auto-create target schema         |
| `drop_existing_tables: true`     | Drop and recreate tables (DANGER) |
| `truncate_existing_tables: true` | Truncate but keep structure       |
| `append_to_existing: true`       | Add rows to existing tables       |

#### Index Strategy

| Strategy                  | When to Use                                    |
| ------------------------- | ---------------------------------------------- |
| `create_before_load`      | Small datasets, need immediate queryability    |
| `defer_indexes` (default) | Large datasets, 2-3x faster load               |
| `create_after_load`       | Same as defer, explicit control                |
| `skip_indexes`            | Ultra-fast load, create indexes manually later |

---

### Network and Security

#### VPC Configuration

For private cloud databases:

```yaml
target:
  type: snowflake
  account: xy12345.privatelink.us-east-1
  private_link: true
  # ... other config ...
```

#### Encryption

All data in transit uses TLS 1.2+. For additional encryption:

```yaml
target:
  type: postgresql
  # ... connection details ...
  ssl:
    mode: verify-full
    root_cert: vault://certs/postgres-ca
    client_cert: vault://certs/client-cert
    client_key: vault://certs/client-key
```

---

### Error Handling

Configure how write errors are handled:

```yaml
target:
  # ... connection details ...
  error_handling:
    on_constraint_violation: skip # skip, fail, or queue
    on_type_error: convert # convert, skip, or fail
    on_null_violation: default # default, skip, or fail
    max_errors: 1000 # Fail migration if exceeded
    error_log: s3://bucket/errors/ # Store rejected records
```

---

### Testing Connections

```bash
curl -X POST https://api.sensei.ai/v1/connections/test \
  -H "Authorization: Bearer sk_live_abc123" \
  -d '{
    "type": "snowflake",
    "account": "xy12345.us-east-1",
    "warehouse": "MIGRATION_WH",
    "database": "PROD_DB"
  }'
```

Response:

```json
{
  "status": "success",
  "latency_ms": 120,
  "warehouse_state": "RUNNING",
  "warehouse_size": "MEDIUM",
  "write_test": "success",
  "available_schemas": ["PUBLIC", "STAGING", "PROD"]
}
```

→ [Source Connectors](source-connectors.md)
→ [API Reference](api-reference.md)
→ [Integration Overview](README.md)
