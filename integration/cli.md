# CLI

## Command-Line Interface

The Sensei CLI provides full platform access from your terminal. Ideal for scripting, CI/CD pipelines, and operators who prefer command-line workflows.

---

## Installation

### macOS (Homebrew)

```bash
brew tap sensei-ai/tap
brew install sensei
```

### Linux

```bash
curl -fsSL https://get.sensei.ai/cli | bash
```

### Windows

```powershell
winget install sensei-ai.sensei
# or
scoop bucket add sensei https://github.com/sensei-ai/scoop-bucket
scoop install sensei
```

### Docker

```bash
docker pull sensei/cli:latest
docker run -it sensei/cli:latest sensei --help
```

### Verify Installation

```bash
sensei --version
# sensei v1.5.0 (build abc123)
```

---

## Configuration

### Authentication

```bash
# Interactive login (opens browser)
sensei login

# Or set API key directly
sensei config set api-key sk_live_abc123

# Use environment variable
export SENSEI_API_KEY=sk_live_abc123
```

### Configuration File

Located at `~/.sensei/config.yaml`:

```yaml
api_key: sk_live_abc123
default_organization: org_xyz
output_format: table # table, json, yaml
color: true
```

### Multiple Profiles

```bash
# Create profile
sensei config profile create staging --api-key sk_test_abc123

# Switch profiles
sensei config profile use staging

# List profiles
sensei config profile list
```

---

## Commands

### Migrations

```bash
# List migrations
sensei migrations list
sensei migrations list --status running

# Create migration
sensei migrations create \
  --name "Q1 Migration" \
  --source conn_oracle \
  --target conn_postgres \
  --schemas HR,FINANCE

# Create from config file
sensei migrations create -f migration.yaml

# Get migration details
sensei migrations get mig_abc123

# Start migration
sensei migrations start mig_abc123

# Monitor migration (live updates)
sensei migrations watch mig_abc123

# Pause/Resume
sensei migrations pause mig_abc123
sensei migrations resume mig_abc123

# Cancel with rollback
sensei migrations cancel mig_abc123 --rollback

# View logs
sensei migrations logs mig_abc123
sensei migrations logs mig_abc123 --follow
sensei migrations logs mig_abc123 --level error
```

### Connections

```bash
# List connections
sensei connections list

# Create connection (interactive)
sensei connections create

# Create from config
sensei connections create -f connection.yaml

# Test connection
sensei connections test conn_abc123

# View schema
sensei connections schema conn_abc123
sensei connections schema conn_abc123 --schema HR --tables

# Preview data
sensei connections preview conn_abc123 \
  --schema HR \
  --table EMPLOYEES \
  --limit 10

# Delete connection
sensei connections delete conn_abc123
```

### Validations

```bash
# Run validation
sensei validations run mig_abc123 --type full

# Check status
sensei validations status val_xyz789

# View report
sensei validations report val_xyz789
sensei validations report val_xyz789 --format pdf > report.pdf

# Download certificate
sensei validations certificate val_xyz789 > certificate.json
```

### Plans

```bash
# View migration plan
sensei plans get mig_abc123

# Review mappings
sensei plans mappings mig_abc123

# Approve all high-confidence mappings
sensei plans approve mig_abc123 --above 0.95

# Approve specific mapping
sensei plans approve mig_abc123 --mapping map_xyz

# Export plan
sensei plans export mig_abc123 -o plan.yaml
```

---

## Configuration Files

### Migration Config

```yaml
# migration.yaml
name: Q1 Database Modernization
description: Migrate legacy Oracle to PostgreSQL

source:
  connection: conn_oracle_prod
  schemas:
    - HR
    - FINANCE
  exclude_tables:
    - AUDIT_LOG
    - TEMP_*

target:
  connection: conn_postgres_cloud
  schema: public

options:
  mode: full
  parallelism: 8
  batch_size: 10000
  auto_approve_above: 0.95
  generate_rollback: true

schedule:
  start_at: '2026-02-15T02:00:00Z'
  maintenance_window:
    start: '02:00'
    end: '06:00'
    timezone: America/New_York

notifications:
  email:
    - team@company.com
  slack: '#data-ops'
  events:
    - completed
    - failed
    - approval_required
```

### Connection Config

```yaml
# connection.yaml
name: Oracle Production
type: oracle
role: source

config:
  host: oracle-prod.company.internal
  port: 1521
  service_name: ORCLPDB1
  username: sensei_reader
  password: ${ORACLE_PASSWORD} # Environment variable
  schemas:
    - HR
    - FINANCE

options:
  ssl_mode: verify-full
  connection_timeout: 30
  query_timeout: 300
  max_connections: 10

tags:
  - production
  - oracle
```

---

## Output Formats

```bash
# Table (default)
sensei migrations list

# JSON
sensei migrations list --output json

# YAML
sensei migrations list --output yaml

# Quiet (IDs only)
sensei migrations list --quiet

# Set default
sensei config set output-format json
```

---

## Scripting

### Bash Example

```bash
#!/bin/bash
set -e

# Create migration and capture ID
MIG_ID=$(sensei migrations create \
  --name "Nightly Sync" \
  --source conn_source \
  --target conn_target \
  --quiet)

echo "Created migration: $MIG_ID"

# Wait for plan
sensei migrations wait $MIG_ID --for plan_ready

# Auto-approve high confidence mappings
sensei plans approve $MIG_ID --above 0.95

# Start and wait for completion
sensei migrations start $MIG_ID
sensei migrations wait $MIG_ID --for completed

# Run validation
VAL_ID=$(sensei validations run $MIG_ID --type full --quiet)
sensei validations wait $VAL_ID

# Export certificate
sensei validations certificate $VAL_ID > "cert_$(date +%Y%m%d).json"

echo "Migration complete!"
```

### CI/CD Integration

```yaml
# .github/workflows/migrate.yaml
name: Database Migration
on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Target environment'
        required: true
        type: choice
        options:
          - staging
          - production

jobs:
  migrate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Install Sensei CLI
        run: curl -fsSL https://get.sensei.ai/cli | bash

      - name: Run Migration
        env:
          SENSEI_API_KEY: ${{ secrets.SENSEI_API_KEY }}
        run: |
          sensei migrations create -f migrations/${{ inputs.environment }}.yaml
          sensei migrations start --wait
          sensei validations run --type full --wait
```

---

## Useful Aliases

Add to your shell config:

```bash
# ~/.bashrc or ~/.zshrc
alias sm='sensei migrations'
alias sc='sensei connections'
alias sv='sensei validations'

# Quick status
alias smw='sensei migrations watch'
alias sml='sensei migrations logs --follow'
```

---

## Troubleshooting

### Debug Mode

```bash
sensei --debug migrations list
```

### Check Configuration

```bash
sensei config show
sensei config validate
```

### Test API Connectivity

```bash
sensei ping
```

### Common Issues

| Issue                | Solution                                          |
| -------------------- | ------------------------------------------------- |
| `unauthorized`       | Check API key: `sensei config show`               |
| `connection refused` | Check network/firewall settings                   |
| `rate limited`       | Wait and retry, or upgrade plan                   |
| `timeout`            | Increase timeout: `sensei config set timeout 120` |

→ [API Reference](api-reference/README.md)
→ [SDKs](sdks.md)
→ [Tutorials](../../enterprise/resources/best-practices/tutorials/README.md)
