---
description: Latency, throughput, and capacity
---

# Performance

## Latency, Throughput, and Capacity

AMANI's performance varies by channel, language, and operation type. This page details what to expect and how the system scales.

---

### Channel Response Latencies

| Channel         | Target Latency | Components                          | Typical Achieved |
| --------------- | -------------- | ----------------------------------- | ---------------- |
| **Web**         | <500ms         | Server processing + WebSocket       | 200-400ms        |
| **WhatsApp**    | <2s            | Webhook → Processing → API response | 1.2-1.8s         |
| **SMS**         | <5s            | Gateway → Processing → Gateway      | 2-4s             |
| **USSD**        | <3s            | Session → Processing → Response     | 1.5-2.5s         |
| **Voice (STT)** | <1s            | Audio → Speech-to-text              | 600-900ms        |
| **Voice (TTS)** | <500ms         | Text → Audio generation             | 300-450ms        |

Latencies measured at p95 (95th percentile). P99 latencies are typically 1.5-2× higher.

---

### Language Processing Performance

| Language Tier          | Detection Latency | Translation Latency | NLU Accuracy |
| ---------------------- | ----------------- | ------------------- | ------------ |
| Tier 1 (50 languages)  | <50ms             | <200ms              | >98%         |
| Tier 2 (80 languages)  | <100ms            | <400ms              | 95-98%       |
| Tier 3 (70+ languages) | <200ms            | <800ms              | 90-95%       |

Translation latency applies only when the user's language differs from the system language. Same-language interactions skip translation.

---

### Agent Coordination

When AMANI routes to Sensei agents for complex queries:

| Query Type              | Typical Agents Involved | Added Latency |
| ----------------------- | ----------------------- | ------------- |
| Status check            | Meta Agent              | +100-200ms    |
| Error explanation       | Scout + Worker          | +300-500ms    |
| Strategy recommendation | Architect               | +500-800ms    |
| Validation query        | KORA                    | +200-400ms    |
| Complex analysis        | Multiple agents         | +1-3s         |

Complex multi-agent queries may take several seconds. AMANI provides immediate acknowledgment ("Let me check...") for queries expected to exceed 2 seconds.

---

### Concurrent Users

AMANI scales horizontally to support concurrent users:

| Deployment Size | Concurrent Users | Conversations/Hour | Infrastructure        |
| --------------- | ---------------- | ------------------ | --------------------- |
| Small           | 50               | 500                | 2 pods, 4 CPU cores   |
| Medium          | 500              | 5,000              | 8 pods, 16 CPU cores  |
| Large           | 5,000            | 50,000             | 32 pods, 64 CPU cores |
| Enterprise      | 50,000+          | 500,000+           | Auto-scaling cluster  |

A "conversation" is a message-response pair. Complex conversations with multiple agent interactions count as multiple conversation units.

---

### Notification Throughput

For broadcast notifications (e.g., migration completion alerts to all stakeholders):

| Channel    | Throughput    | Limiting Factor          |
| ---------- | ------------- | ------------------------ |
| Web (push) | 10,000/second | WebSocket connections    |
| WhatsApp   | 80/second     | Meta API rate limits     |
| SMS        | 100/second    | Carrier gateway limits   |
| USSD       | N/A           | Session-based, not push  |
| Voice      | 10/second     | Concurrent call capacity |

For large recipient lists, notifications are queued and delivered over time. Critical notifications (e.g., migration failures) use SMS as primary channel for guaranteed delivery.

---

### Offline Sync Performance

| Metric                                 | Value         |
| -------------------------------------- | ------------- |
| Sync detection after connectivity      | <1 second     |
| Typical sync duration (1-day offline)  | 2-5 seconds   |
| Typical sync duration (1-week offline) | 5-15 seconds  |
| Maximum sync duration (30-day offline) | 30-60 seconds |
| Bandwidth (typical sync)               | 10-100 KB     |
| Bandwidth (30-day sync)                | 1-10 MB       |

Sync is incremental — only changes since last sync are exchanged. CRDT merge operations are computationally lightweight.

---

### Resource Requirements

#### Server-Side

| Component          | CPU        | Memory   | Storage   | Purpose                 |
| ------------------ | ---------- | -------- | --------- | ----------------------- |
| AMANI API          | 2-8 cores  | 4-16 GB  | Minimal   | Request handling        |
| Conversation state | 1-2 cores  | 8-32 GB  | 100GB-1TB | Session storage (Redis) |
| Language models    | 4-16 cores | 16-64 GB | 20-100 GB | NLU, translation        |
| WebSocket server   | 2-4 cores  | 2-8 GB   | Minimal   | Real-time connections   |

#### Client-Side (Edge)

| Capability Level         | RAM    | Storage | Network         |
| ------------------------ | ------ | ------- | --------------- |
| Full (with local LLM)    | 8+ GB  | 8 GB    | Optional        |
| Standard (no local LLM)  | 2 GB   | 2 GB    | Intermittent OK |
| Minimal (templates only) | 512 MB | 500 MB  | Optional        |

---

### Performance Monitoring

AMANI exposes metrics for monitoring:

- **Latency percentiles:** p50, p90, p95, p99 by channel and operation
- **Error rates:** By channel, language, and error type
- **Throughput:** Messages/second, conversations/hour
- **Sync metrics:** Sync duration, conflict rate, failure rate
- **User satisfaction:** Response helpfulness ratings (when provided)

Metrics are available via Prometheus/Grafana integration and the AMANI admin dashboard.

→ [Multi-Channel Communication](../../../product/platform/amani/multi-channel.md)
→ [Offline-First Architecture](../../../product/platform/amani/offline-first.md)
→ [AMANI Overview](../../../product/platform/amani/README.md)
