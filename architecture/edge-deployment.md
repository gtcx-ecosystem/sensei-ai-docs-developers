---
description: Offline and limited-connectivity deployments
---

# Edge & Offline Deployment

## Migration Intelligence Without the Cloud

Not every migration happens in a well-connected data center. Government agencies in developing countries, defense installations, financial institutions with air-gapped networks, and field offices on intermittent connectivity all need migration intelligence. Sensei's edge deployment model brings the platform to them.

---

### AMANI Edge Client

The primary edge deployment is the AMANI client — a lightweight application that runs on local devices and provides migration guidance without server connectivity.

#### What Runs Locally

| Component                                        | Size                 | Purpose                                              |
| ------------------------------------------------ | -------------------- | ---------------------------------------------------- |
| Compressed language model (Llama-3-8B quantized) | ~4 GB                | Basic NLU, error classification, guidance generation |
| Pattern library cache                            | ~500 MB              | Most common patterns for the migration profile       |
| Decision trees                                   | ~50 MB               | Pre-computed guidance for standard workflow steps    |
| Language packs                                   | ~100 MB per language | Offline multilingual support                         |
| Migration state                                  | Variable             | Current migration progress, configuration, logs      |

**Total local footprint:** ~5 GB for full capability, ~1 GB for minimal configuration.

#### Supported Devices

| Device                | Capability                              | Duration                     |
| --------------------- | --------------------------------------- | ---------------------------- |
| Modern laptop         | Full AMANI + local LLM inference        | Unlimited (power permitting) |
| Tablet (iPad/Android) | AMANI guidance, no local LLM            | 30 days offline              |
| Smartphone            | AMANI via WhatsApp/SMS cached responses | 30 days offline              |
| Feature phone         | USSD menus, SMS templates               | N/A (carrier-based)          |

---

### Offline Operation

When connectivity is unavailable, the AMANI edge client operates autonomously:

**What works offline:**

- Migration progress tracking and status updates
- Guidance for common workflow steps (pre-computed decision trees)
- Error classification and resolution for patterns in the local cache
- Stakeholder communication via locally cached response templates
- Data validation using locally stored validation rules
- Basic natural language interaction using the compressed local model

**What requires connectivity:**

- Cross-migration pattern library updates
- Frontier model inference for novel situations
- Real-time collaboration with Cognitive Agents
- Pattern library contribution (uploading local discoveries)
- Software updates and model refreshes

**What happens during extended offline periods:**
Decisions and actions taken offline are logged locally with timestamps. When connectivity is restored, the CRDT-based synchronization protocol reconciles local state with server state without conflicts or data loss.

---

### CRDT-Based Synchronization

When the edge client reconnects, bidirectional sync resolves the divergence between local and server state:

**Upload (edge → server):**

- Migration progress updates
- Decisions made during offline period
- Errors encountered and resolutions applied
- New patterns discovered locally

**Download (server → edge):**

- Updated pattern library excerpts
- New guidance models and decision trees
- Platform notifications and configuration changes
- Patterns discovered by other migrations during the offline period

**Conflict resolution:**
Conflict-free Replicated Data Types (CRDTs) ensure that concurrent modifications merge deterministically without coordination. If a migration parameter was changed both locally and on the server during the offline period, the CRDT merge strategy produces a consistent result without requiring human intervention.

---

### Air-Gapped Deployment

For environments with no external connectivity at all, Sensei deploys as a self-contained installation:

**All-in-one appliance:**

- Full Kubernetes cluster on bare metal or VMs
- Local LLM inference (Llama-3-70B for Cognitive, Llama-3-8B for Execution)
- Self-hosted databases (Neo4j, MongoDB, Redis, Weaviate)
- Pre-loaded pattern library from anonymized cross-customer intelligence
- AMANI web interface served locally

**Limitations of air-gapped deployment:**

- No cross-customer learning (patterns don't leave the installation)
- Pattern library only grows from the organization's own migrations
- No automatic model updates (requires manual software update cycle)
- LLM capability limited to locally-hosted models (no GPT-4/Claude API)

**Mitigations:**

- Pre-loaded pattern library contains thousands of anonymized patterns from the public knowledge base
- Manual update packages can be brought in on physical media for periodic model and pattern library updates
- Local learning still compounds — the system improves from the organization's own migrations even in complete isolation

---

### Frontier Market Considerations

Edge deployment is designed for real conditions in developing economies:

**Power reliability:** The AMANI client handles ungraceful shutdown — migration state is checkpointed to durable storage continuously, enabling recovery from power loss without data corruption.

**Network variability:** Sync operations are designed for intermittent, low-bandwidth connections. Compressed delta transfers minimize data volume. Sync can complete over 2G connections, though slowly.

**Device diversity:** The five-channel architecture (web, WhatsApp, SMS, USSD, voice) ensures that even the most basic device can access migration guidance. USSD menus work on any mobile phone with cellular service, with zero data consumption.

**Local language support:** Offline language packs support the most common languages for each deployment region. Organizations deploying in West Africa receive pre-loaded Swahili, Hausa, Yoruba, French, and English packs. Southeast Asian deployments receive Bahasa, Thai, Vietnamese, and English.

→ [AMANI: Offline-First Architecture](../../product/platform/amani/offline-first.md)
→ [Frontier Markets](../../product/why-sensei/frontier-markets.md)
→ [Deployment Architecture Overview](deployment.md)
