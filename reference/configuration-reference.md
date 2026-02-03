---
description: All configuration options
---

# Configuration Reference

## Complete Configuration Options

Comprehensive reference for all Sensei configuration options.

---

### Connection Configuration

```yaml
connection:
  # Required
  type: postgresql # Database type
  host: localhost # Hostname or IP
  port: 5432 # Port number
  database: mydb # Database name
  username: user # Username
  password: pass # Password (or use secrets)

  # Optional
  schema: public # Default schema
  ssl_mode: require # disable, allow, prefer, require, verify-ca, verify-full
  ssl_ca: /path/to/ca.pem # CA certificate path
  ssl_cert: /path/to/cert # Client certificate
  ssl_key: /path/to/key # Client key

  # Connection pool
  pool_size: 10 # Connections in pool
  pool_timeout: 30 # Seconds to wait for connection
  idle_timeout: 300 # Close idle connections after
  max_lifetime: 3600 # Max connection lifetime

  # Timeouts
  connect_timeout: 10 # Connection timeout (seconds)
  query_timeout: 300 # Query timeout (seconds)

  # Advanced
  application_name: sensei # Application identifier
  extra_params: {} # Database-specific parameters
```

---

### Migration Configuration

```yaml
migration:
  # Basic
  name: My Migration # Migration name
  source: conn_source # Source connection ID
  target: conn_target # Target connection ID

  # Tables
  tables: # Tables to include (default: all)
    - customers
    - orders
  exclude_tables: # Tables to exclude
    - temp_*
    - log_*

  # Mode
  mode: full # full, incremental

  # Execution
  batch_size: 10000 # Records per batch
  workers: 4 # Parallel workers

  # Checkpointing
  checkpointing:
    enabled: true
    interval: 5m # Checkpoint interval

  # Error handling
  error_handling:
    max_errors: 100 # Max errors before abort
    continue_on_error: false
    retry_count: 3
    retry_delay: 5s

  # Performance
  parallel_extract: true # Parallel source extraction
  parallel_load: true # Parallel target loading
  compression: true # Compress data in transit
```

---

### Transformation Configuration

```yaml
transformations:
  # Global settings
  null_handling: preserve # preserve, default, error
  default_encoding: utf-8

  # Column-level
  columns:
    created_date:
      transform: "TO_TIMESTAMP(source, 'YYYY-MM-DD')"
      null_handling: use_default
      default_value: '1970-01-01'

    email:
      transform: 'LOWER(TRIM(source))'

    phone:
      transform: "REGEXP_REPLACE(source, '[^0-9]', '')"

  # Type mappings
  type_mappings:
    - source_type: 'NUMBER(10)'
      target_type: 'BIGINT'
    - source_type: 'VARCHAR2(4000)'
      target_type: 'TEXT'

  # Custom functions
  custom_functions:
    - name: mask_ssn
      language: python
      code: |
        def mask_ssn(value):
            return 'XXX-XX-' + value[-4:] if value else None
```

---

### Validation Configuration

```yaml
validation:
  # Enable/disable types
  structural: true
  statistical: true
  referential: true
  semantic: true
  behavioral: true          # Time-Travel Testing

  # Thresholds
  thresholds:
    row_count_tolerance: 0  # Exact match
    null_rate_tolerance: 0.01  # 1% variance allowed
    distribution_tolerance: 0.05

  # Statistical validation
  statistical:
    sample_size: 10000      # Records to sample
    confidence_level: 0.95

  # Behavioral validation
  behavioral:
    query_replay: true
    checkpoint_count: 5
    comparison_mode: semantic  # exact, semantic, business

  # Exceptions
  exceptions:
    - table: audit_log
      skip: [statistical]
    - column: modified_at
      skip: [distribution]
```

---

### CDC Configuration

```yaml
cdc:
  enabled: true

  # Method (auto-detected if not specified)
  method: logical_replication # logical_replication, binlog, logminer

  # Polling
  poll_interval: 1s
  batch_size: 1000

  # Lag management
  max_lag: 60s # Alert if lag exceeds
  lag_action: alert # alert, pause, abort

  # Replication slot (PostgreSQL)
  slot_name: sensei_slot
  publication_name: sensei_pub

  # Binary log (MySQL)
  server_id: 12345
  binlog_position: auto # auto or specific position
```

---

### Logging Configuration

```yaml
logging:
  level: info # debug, info, warn, error
  format: json # json, text

  # Output
  stdout: true
  file:
    enabled: true
    path: /var/log/sensei
    max_size: 100MB
    max_files: 10

  # External
  siem:
    enabled: true
    endpoint: https://siem.example.com
    api_key: ${SIEM_API_KEY}
```

---

### Notification Configuration

```yaml
notifications:
  webhooks:
    - url: https://example.com/webhook
      events: [migration.completed, migration.failed]
      secret: ${WEBHOOK_SECRET}
      retry_count: 3

  email:
    enabled: true
    recipients: [team@example.com]
    events: [migration.failed]

  slack:
    enabled: true
    webhook_url: ${SLACK_WEBHOOK}
    channel: '#data-team'
    events: [migration.completed, migration.failed]
```

---

### Security Configuration

```yaml
security:
  # Encryption
  encryption:
    at_rest: true
    key_management: aws_kms # aws_kms, gcp_kms, vault, local
    kms_key_id: ${KMS_KEY_ID}

  # Authentication
  authentication:
    sso:
      enabled: true
      provider: okta
      domain: company.okta.com
      client_id: ${OKTA_CLIENT_ID}

  # Access control
  access_control:
    ip_allowlist:
      - 10.0.0.0/8
      - 192.168.1.0/24
    mfa_required: true
    session_timeout: 8h
```

---

### Resource Limits

```yaml
resources:
  # Memory
  memory:
    worker: 2GB
    max_total: 32GB

  # CPU
  cpu:
    worker: 2
    max_total: 32

  # Storage
  storage:
    temp_dir: /tmp/sensei
    max_temp_size: 100GB

  # Network
  network:
    max_connections: 100
    bandwidth_limit: 1Gbps
```

---

### Configuration Precedence

1. Environment variables (highest)
2. Command-line arguments
3. Configuration file
4. Default values (lowest)

→ [Environment Variables](environment-variables.md)
→ [Limits and Quotas](limits-quotas.md)
