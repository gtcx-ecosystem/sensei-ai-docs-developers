---
description: Code examples and common patterns
---

# Integration Examples

## Complete Working Examples

This page provides end-to-end examples for common integration patterns. Each example is complete and can be adapted for your specific use case.

---

### Example 1: CI/CD Pipeline Integration

Trigger a migration as part of your deployment pipeline and gate deployment on successful completion.

**GitHub Actions:**

```yaml
# .github/workflows/migrate-database.yml
name: Database Migration

on:
  push:
    branches: [main]
    paths:
      - 'migrations/**'

jobs:
  migrate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Install Sensei CLI
        run: pip install sensei-sdk

      - name: Run Migration
        env:
          SENSEI_API_KEY: ${{ secrets.SENSEI_API_KEY }}
        run: |
          python scripts/migrate.py

      - name: Verify Migration
        env:
          SENSEI_API_KEY: ${{ secrets.SENSEI_API_KEY }}
        run: |
          python scripts/verify.py

# scripts/migrate.py
import os
import sys
from sensei import Sensei

client = Sensei(api_key=os.environ["SENSEI_API_KEY"])

migration = client.migrations.create(
    name=f"Deploy {os.environ.get('GITHUB_SHA', 'manual')[:8]}",
    source={
        "type": "postgresql",
        "connection_string": os.environ["SOURCE_DB_URL"]
    },
    target={
        "type": "postgresql",
        "connection_string": os.environ["TARGET_DB_URL"]
    },
    strategy="safe"
)

migration.start()

# Wait for completion with timeout
try:
    result = migration.wait(timeout=3600)  # 1 hour timeout
    if result.status == "completed":
        print(f"Migration completed: {result.records_processed} records")
        sys.exit(0)
    else:
        print(f"Migration failed: {result.error}")
        sys.exit(1)
except TimeoutError:
    print("Migration timed out")
    migration.cancel()
    sys.exit(1)
```

---

### Example 2: Slack Notifications

Route migration events to Slack channels based on severity.

```python
# webhook_handler.py
from flask import Flask, request, jsonify
from slack_sdk import WebClient
from sensei.webhooks import verify_signature

app = Flask(__name__)
slack = WebClient(token=os.environ["SLACK_BOT_TOKEN"])

CHANNELS = {
    "critical": "#migration-alerts",
    "info": "#migration-updates",
    "success": "#migration-wins"
}

@app.post("/webhooks/sensei")
def handle_sensei_webhook():
    # Verify signature
    if not verify_signature(
        payload=request.data,
        signature=request.headers.get("X-Sensei-Signature"),
        secret=os.environ["SENSEI_WEBHOOK_SECRET"]
    ):
        return jsonify({"error": "Invalid signature"}), 401

    event = request.json

    # Route based on event type
    if event["type"] == "migration.failed":
        notify_slack("critical", format_failure(event))
    elif event["type"] == "error.escalated":
        notify_slack("critical", format_error(event))
    elif event["type"] == "migration.completed":
        notify_slack("success", format_success(event))
    elif event["type"] in ["migration.started", "migration.phase.completed"]:
        notify_slack("info", format_progress(event))

    return jsonify({"status": "ok"})

def notify_slack(severity, message):
    slack.chat_postMessage(
        channel=CHANNELS[severity],
        blocks=message["blocks"],
        text=message["fallback"]
    )

def format_failure(event):
    return {
        "fallback": f"ALERT: Migration failed: {event['data']['migration_name']}",
        "blocks": [
            {
                "type": "header",
                "text": {"type": "plain_text", "text": "ALERT: Migration Failed"}
            },
            {
                "type": "section",
                "fields": [
                    {"type": "mrkdwn", "text": f"*Migration:*\n{event['data']['migration_name']}"},
                    {"type": "mrkdwn", "text": f"*Error:*\n{event['data']['error_message']}"}
                ]
            },
            {
                "type": "actions",
                "elements": [
                    {
                        "type": "button",
                        "text": {"type": "plain_text", "text": "View Details"},
                        "url": f"https://app.sensei.ai/migrations/{event['data']['migration_id']}"
                    }
                ]
            }
        ]
    }
```

---

### Example 3: Custom Dashboard

Build a real-time migration dashboard using Server-Sent Events.

```html
<!-- dashboard.html -->
<!DOCTYPE html>
<html>
  <head>
    <title>Migration Dashboard</title>
    <script src="https://cdn.tailwindcss.com"></script>
  </head>
  <body class="bg-gray-100 p-8">
    <div class="mx-auto max-w-4xl">
      <h1 class="mb-6 text-2xl font-bold">Active Migrations</h1>
      <div id="migrations" class="space-y-4"></div>
    </div>

    <script>
      const API_KEY = 'sk_live_abc123'; // In production, use a backend proxy

      async function loadMigrations() {
        const response = await fetch('https://api.sensei.ai/v1/migrations?status=running', {
          headers: { Authorization: `Bearer ${API_KEY}` },
        });
        const { data } = await response.json();

        const container = document.getElementById('migrations');
        container.innerHTML = data.map((m) => migrationCard(m)).join('');

        // Subscribe to events for each migration
        data.forEach((m) => subscribeToEvents(m.id));
      }

      function migrationCard(migration) {
        return `
                <div id="migration-${migration.id}" class="bg-white rounded-lg shadow p-6">
                    <div class="flex justify-between items-center mb-4">
                        <h2 class="text-lg font-semibold">${migration.name}</h2>
                        <span class="px-3 py-1 rounded-full text-sm ${statusColor(migration.status)}">
                            ${migration.status}
                        </span>
                    </div>
                    <div class="mb-4">
                        <div class="flex justify-between text-sm text-gray-600 mb-1">
                            <span>Progress</span>
                            <span id="progress-${migration.id}">${migration.progress || 0}%</span>
                        </div>
                        <div class="w-full bg-gray-200 rounded-full h-2">
                            <div id="bar-${migration.id}" 
                                 class="bg-blue-600 h-2 rounded-full transition-all duration-500"
                                 style="width: ${migration.progress || 0}%"></div>
                        </div>
                    </div>
                    <div class="grid grid-cols-3 gap-4 text-center">
                        <div>
                            <div id="records-${migration.id}" class="text-xl font-bold">
                                ${formatNumber(migration.records_processed || 0)}
                            </div>
                            <div class="text-sm text-gray-500">Records</div>
                        </div>
                        <div>
                            <div id="errors-${migration.id}" class="text-xl font-bold text-red-600">
                                ${migration.errors || 0}
                            </div>
                            <div class="text-sm text-gray-500">Errors</div>
                        </div>
                        <div>
                            <div id="throughput-${migration.id}" class="text-xl font-bold">
                                ${formatNumber(migration.throughput || 0)}
                            </div>
                            <div class="text-sm text-gray-500">Records/hr</div>
                        </div>
                    </div>
                </div>
            `;
      }

      function subscribeToEvents(migrationId) {
        const eventSource = new EventSource(
          `https://api.sensei.ai/v1/migrations/${migrationId}/events?token=${API_KEY}`
        );

        eventSource.onmessage = (event) => {
          const data = JSON.parse(event.data);
          updateMigrationUI(migrationId, data);
        };
      }

      function updateMigrationUI(id, data) {
        if (data.progress) {
          document.getElementById(`progress-${id}`).textContent = `${data.progress}%`;
          document.getElementById(`bar-${id}`).style.width = `${data.progress}%`;
        }
        if (data.records_processed) {
          document.getElementById(`records-${id}`).textContent = formatNumber(
            data.records_processed
          );
        }
        if (data.errors !== undefined) {
          document.getElementById(`errors-${id}`).textContent = data.errors;
        }
        if (data.throughput) {
          document.getElementById(`throughput-${id}`).textContent = formatNumber(data.throughput);
        }
      }

      function formatNumber(n) {
        return n.toLocaleString();
      }

      function statusColor(status) {
        const colors = {
          running: 'bg-blue-100 text-blue-800',
          completed: 'bg-green-100 text-green-800',
          failed: 'bg-red-100 text-red-800',
          paused: 'bg-yellow-100 text-yellow-800',
        };
        return colors[status] || 'bg-gray-100 text-gray-800';
      }

      loadMigrations();
    </script>
  </body>
</html>
```

---

### Example 4: Programmatic Mapping Approval

Review and approve mappings programmatically based on custom rules.

```python
# auto_approve.py
from sensei import Sensei

client = Sensei(api_key="sk_live_abc123")

def auto_approve_mappings(migration_id):
    """
    Automatically approve high-confidence mappings,
    flag uncertain ones for review.
    """
    migration = client.migrations.get(migration_id)
    mappings = client.mappings.list(migration_id)

    to_approve = []
    to_review = []

    for mapping in mappings:
        # Auto-approve criteria
        if should_auto_approve(mapping):
            to_approve.append(mapping.id)
        else:
            to_review.append(mapping)

    # Bulk approve
    if to_approve:
        client.mappings.bulk_update(
            migration_id=migration_id,
            updates=[{"id": mid, "approved": True} for mid in to_approve]
        )
        print(f"Auto-approved {len(to_approve)} mappings")

    # Report on remaining
    if to_review:
        print(f"\n{len(to_review)} mappings need review:")
        for m in to_review:
            print(f"  - {m.source_column} → {m.target_column} ({m.confidence:.0%})")
            print(f"    Reason: {get_review_reason(m)}")

    return len(to_approve), len(to_review)

def should_auto_approve(mapping):
    """Define your auto-approval criteria"""

    # High confidence
    if mapping.confidence >= 0.95:
        return True

    # Known safe transformations
    safe_transforms = [
        "identity",          # No change
        "type_widening",     # INT → BIGINT
        "encoding_normalize" # UTF-8 normalization
    ]
    if mapping.transformation_type in safe_transforms:
        return True

    # Exact name match with compatible types
    if (mapping.source_column.lower() == mapping.target_column.lower() and
        mapping.type_compatible):
        return True

    return False

def get_review_reason(mapping):
    """Explain why mapping needs review"""
    reasons = []

    if mapping.confidence < 0.80:
        reasons.append(f"Low confidence ({mapping.confidence:.0%})")

    if mapping.has_data_loss_risk:
        reasons.append("Potential data loss (precision/truncation)")

    if mapping.is_pii:
        reasons.append("PII field - verify handling")

    if mapping.transformation_type == "custom":
        reasons.append("Custom transformation - verify logic")

    return "; ".join(reasons) or "Manual review requested"

# Usage
if __name__ == "__main__":
    import sys
    migration_id = sys.argv[1]
    approved, pending = auto_approve_mappings(migration_id)

    if pending == 0:
        print("\nAll mappings approved - ready to start migration")
    else:
        print(f"\nWARNING: {pending} mappings need manual review")
        print("   Review at: https://app.sensei.ai/migrations/{migration_id}/mappings")
```

---

### Example 5: Error Recovery Automation

Automatically handle common errors during migration.

```python
# error_handler.py
from sensei import Sensei
from sensei.webhooks import verify_signature
from flask import Flask, request

app = Flask(__name__)
client = Sensei(api_key="sk_live_abc123")

@app.post("/webhooks/errors")
def handle_error_webhook():
    event = request.json

    if event["type"] == "error.occurred":
        handle_error(event["data"])

    return {"status": "ok"}

def handle_error(error_data):
    """Attempt automatic recovery based on error type"""

    error_type = error_data["error_type"]
    migration_id = error_data["migration_id"]
    error_id = error_data["error_id"]

    handler = ERROR_HANDLERS.get(error_type)
    if handler:
        try:
            handler(client, migration_id, error_id, error_data)
            print(f"Auto-recovered: {error_type}")
        except Exception as e:
            print(f"Auto-recovery failed: {e}")
            escalate_to_human(error_data)
    else:
        escalate_to_human(error_data)

def handle_encoding_error(client, migration_id, error_id, data):
    """Fix encoding issues by applying UTF-8 normalization"""
    client.errors.resolve(
        migration_id=migration_id,
        error_id=error_id,
        resolution={
            "type": "apply_transformation",
            "transformation": "force_utf8",
            "retry": True
        }
    )

def handle_truncation_warning(client, migration_id, error_id, data):
    """Accept truncation for non-critical fields"""
    non_critical = ["description", "notes", "comments", "remarks"]
    column = data["column_name"].lower()

    if any(nc in column for nc in non_critical):
        client.errors.resolve(
            migration_id=migration_id,
            error_id=error_id,
            resolution={
                "type": "accept_truncation",
                "reason": "Non-critical descriptive field"
            }
        )
    else:
        raise Exception("Truncation on critical field - needs review")

def handle_null_violation(client, migration_id, error_id, data):
    """Apply default values for known nullable fields"""
    defaults = {
        "status": "'UNKNOWN'",
        "created_at": "CURRENT_TIMESTAMP",
        "updated_at": "CURRENT_TIMESTAMP",
        "is_active": "true"
    }

    column = data["column_name"].lower()
    if column in defaults:
        client.errors.resolve(
            migration_id=migration_id,
            error_id=error_id,
            resolution={
                "type": "apply_default",
                "default_value": defaults[column],
                "retry": True
            }
        )
    else:
        raise Exception(f"No default defined for {column}")

def escalate_to_human(error_data):
    """Send to Slack/PagerDuty for human review"""
    # Implementation depends on your alerting system
    pass

ERROR_HANDLERS = {
    "encoding_mismatch": handle_encoding_error,
    "truncation_warning": handle_truncation_warning,
    "null_constraint_violation": handle_null_violation,
}
```

→ [API Reference](api-reference.md)
→ [Webhooks](webhooks.md)
→ [SDKs](sdks.md)
