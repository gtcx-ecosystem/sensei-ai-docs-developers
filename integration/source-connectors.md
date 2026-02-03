# Source Connectors

## Connecting to Your Source Databases

Sensei connects to 50+ source formats through native connectors optimized for migration workloads. This page covers supported sources, connection configuration, and best practices.

---

### Supported Sources

#### Relational Databases

| Database       | Versions | Connector Type | Catalog Extraction | CDC Support           |
| -------------- | -------- | -------------- | ------------------ | --------------------- |
| **PostgreSQL** | 10+      | Native (libpq) | Full               | Logical replication   |
| **MySQL**      | 5.7+     | Native         | Full               | Binary log            |
| **Oracle**     | 11g+     | Native (OCI)   | Full               | LogMiner / GoldenGate |
| **SQL Server** | 2016+    | Native (TDS)   | Full               | Change tracking       |
| **SQLite**     | 3.x      | Native         | Full               | N/A                   |
| **IBM DB2**    | 11.1+    | JDBC           | Full               | InfoSphere CDC        |
| **SAP Sybase** | 15.7+    | JDBC           | Full               | Replication Server    |
| **MariaDB**    | 10.2+    | Native         | Full               | Binary log            |

#### Cloud Data Warehouses

| Platform          | Connector Type | Notes                     |
| ----------------- | -------------- | ------------------------- |
| **Snowflake**     | Native REST    | Time travel for snapshots |
| **BigQuery**      | Native REST    | Dataset-level access      |
| **Redshift**      | Native (libpq) | PostgreSQL-compatible     |
| **Azure Synapse** | Native (TDS)   | SQL Server compatible     |
| **Databricks**    | Native REST    | Unity Catalog support     |

#### Document Stores

| Database          | Versions | Notes                         |
| ----------------- | -------- | ----------------------------- |
| **MongoDB**       | 4.0+     | Change streams for CDC        |
| **CouchDB**       | 2.x+     | Changes feed                  |
| **Elasticsearch** | 7.x+     | Scroll API for large datasets |

#### File Formats

| Format      | Variations                       |
| ----------- | -------------------------------- |
| **CSV**     | Standard, TSV, custom delimiters |
| **JSON**    | Single-object, NDJSON, nested    |
| **Parquet** | Standard, partitioned            |
| **Avro**    | With/without schema registry     |
| **ORC**     | Standard                         |
| **Excel**   | XLSX, XLS, XLSM                  |
| **XML**     | Generic, domain-specific schemas |

#### Legacy Formats

| Format                | Notes                                |
| --------------------- | ------------------------------------ |
| **COBOL Copybooks**   | COMP-3, EBCDIC support               |
| **Mainframe dumps**   | RDW headers, variable-length records |
| **Fixed-width files** | Layout specification required        |

---

### Connection Configuration

#### Standard Connection String

```yaml
source:
  type: postgresql
  connection_string: 'postgresql://user:password@host:5432/database'
  schema: public
  ssl_mode: require
```

#### Secure Credential Management

**Option 1: Vault Integration**

```yaml
source:
  type: oracle
  host: oracle.example.com
  port: 1521
  service_name: ORCL
  credentials: vault://secrets/oracle-prod
  ssl_mode: require
```

**Option 2: Environment Variables**

```yaml
source:
  type: postgresql
  connection_string: '${POSTGRES_CONNECTION_STRING}'
```

**Option 3: AWS Secrets Manager**

```yaml
source:
  type: mysql
  host: mysql.example.com
  credentials: aws-secrets://prod/mysql-credentials
```

---

### Database-Specific Configuration

#### Oracle

```yaml
source:
  type: oracle
  host: oracle.example.com
  port: 1521
  service_name: ORCL # or sid: ORCL
  username: migration_user
  credentials: vault://secrets/oracle
  options:
    fetch_size: 10000
    array_size: 1000
    use_flashback: true # For point-in-time snapshots
    flashback_time: '2026-02-01T00:00:00Z'
```

#### SQL Server

```yaml
source:
  type: sqlserver
  host: sqlserver.example.com
  port: 1433
  database: ProductionDB
  username: migration_user
  credentials: vault://secrets/sqlserver
  options:
    encrypt: true
    trust_server_certificate: false
    application_intent: ReadOnly
    multi_subnet_failover: true
```

#### MongoDB

```yaml
source:
  type: mongodb
  connection_string: 'mongodb+srv://user:pass@cluster.mongodb.net/database'
  options:
    read_preference: secondaryPreferred
    read_concern: majority
    use_change_streams: true # For CDC
```

---

### Network Access

#### Direct Connection

Sensei connects directly to your database. Ensure:

- Database is network-accessible from Sensei (public IP or VPC peering)
- Firewall rules allow inbound connections on database port
- User has SELECT permissions on source tables

#### VPC Peering

For databases in private VPCs:

1. Create VPC peering connection to Sensei's VPC
2. Update route tables to allow traffic
3. Configure security groups to allow Sensei's IP range

#### SSH Tunnel

For databases behind a bastion host:

```yaml
source:
  type: postgresql
  host: localhost
  port: 5432
  database: production
  tunnel:
    type: ssh
    host: bastion.example.com
    port: 22
    username: tunnel_user
    private_key: vault://secrets/bastion-key
    local_port: 5432
    remote_host: internal-db.example.com
    remote_port: 5432
```

#### Private Link (AWS/Azure/GCP)

Enterprise plans support private endpoints for cloud databases without public internet exposure.

---

### Performance Tuning

| Parameter              | Default | Description              |
| ---------------------- | ------- | ------------------------ |
| `batch_size`           | 10,000  | Rows per read batch      |
| `parallel_readers`     | 4       | Concurrent table readers |
| `fetch_size`           | 10,000  | JDBC fetch size          |
| `query_timeout`        | 3600s   | Maximum query duration   |
| `connection_pool_size` | 10      | Database connections     |

**High-volume sources (>100M rows):**

```yaml
source:
  # ... connection details ...
  options:
    batch_size: 50000
    parallel_readers: 8
    connection_pool_size: 20
```

**Sensitive production sources:**

```yaml
source:
  # ... connection details ...
  options:
    batch_size: 5000
    parallel_readers: 2
    connection_pool_size: 5
    query_timeout: 600
```

---

### Testing Connections

Always test before starting a migration:

```bash
curl -X POST https://api.sensei.ai/v1/connections/test \
  -H "Authorization: Bearer sk_live_abc123" \
  -d '{
    "type": "postgresql",
    "connection_string": "postgresql://user:pass@host:5432/db"
  }'
```

Response:

```json
{
  "status": "success",
  "latency_ms": 45,
  "server_version": "PostgreSQL 15.2",
  "accessible_schemas": ["public", "analytics"],
  "table_count": 247,
  "estimated_rows": 52000000
}
```

→ [Target Connectors](target-connectors.md)
→ [API Reference](api-reference.md)
→ [Integration Overview](README.md)
