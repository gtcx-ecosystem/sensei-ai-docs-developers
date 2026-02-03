---
description: Cross-component coordination
---

# Component Orchestration

## How MABA, KORA, and AMANI Work Together

The three core components — MABA (transformation), KORA (verification), and AMANI (human interface) — operate as a coordinated system, not independent tools. This page explains how they orchestrate to deliver autonomous migration intelligence.

---

### The Orchestration Model

Components communicate through the **Agent Layer**, which serves as the coordination fabric:

```text
                    ┌─────────────────┐
                    │     AMANI       │
                    │ (Human Intent)  │
                    └────────┬────────┘
                             │
                             ↓
┌────────────────────────────────────────────────────────┐
│                    AGENT LAYER                          │
│                                                         │
│  Scout ←→ Architect ←→ Worker ←→ Validator ←→ Meta    │
│                                                         │
│         Redis Pub/Sub    │    Temporal Workflows       │
└────────────────────────────────────────────────────────┘
                    ↙            ↘
           ┌───────────┐    ┌───────────┐
           │   MABA    │    │   KORA    │
           │ Transform │ ←→ │  Verify   │
           └───────────┘    └───────────┘
```

**Key principle:** Components don't call each other directly. They communicate through agents, which provides:

- Loose coupling (components can evolve independently)
- Observability (all interactions are logged)
- Learning (Meta Agents observe and improve)

---

### Orchestration Phases

#### Phase 1: Discovery

**Lead:** Scout Agents + MABA

1. Scout Agents connect to the source database
2. MABA's ingestion layer extracts schema metadata
3. Scout Agents analyze the schema using LLM inference
4. MABA computes distributional fingerprints for every column
5. Results are broadcast to all agents via Redis pub/sub

**AMANI role:** Reports discovery progress, surfaces questions about ambiguous structures

**KORA role:** Idle (waiting for data to verify)

---

#### Phase 2: Planning

**Lead:** Architect Agents + MABA

1. Architect Agents receive Scout discoveries
2. MABA's semantic pattern recognition suggests mappings
3. Architect Agents generate the migration DAG
4. MABA generates transformation code for each mapping
5. Plan is presented for approval

**AMANI role:** Presents plan to stakeholders, collects approval, explains recommendations

**KORA role:** Validates plan feasibility (can these transformations be verified?)

---

#### Phase 3: Execution

**Lead:** Worker Agents + MABA + KORA

1. Worker Agents pull tasks from the execution queue
2. MABA executes transformations (read → transform → write)
3. KORA's Validator Agents check each record inline
4. Errors trigger Worker Agent recovery attempts
5. Successful patterns broadcast to swarm

**AMANI role:** Reports progress, alerts on errors, escalates decisions

**KORA role:** Continuous inline validation, error classification

---

#### Phase 4: Verification

**Lead:** KORA + Validator Agents

1. KORA runs post-migration structural/statistical/referential validation
2. KORA executes Time-Travel Testing (behavioral verification)
3. Validator Agents reconcile any discrepancies
4. KORA generates Behavioral Equivalence Certificate

**AMANI role:** Reports validation progress, presents results to stakeholders

**MABA role:** Provides transformation logs for state reconstruction

---

#### Phase 5: Learning

**Lead:** Meta Agents + Learning Agents

1. Meta Agents extract patterns from the completed migration
2. Learning Agents update the pattern library
3. Evolutionary selection updates strategy populations
4. Cross-customer patterns are shared (privacy-preserving)

**AMANI role:** Privacy-preserving pattern transmission via federated learning

**MABA role:** Contributes validated mappings to pattern library

**KORA role:** Contributes validated error-recovery pairs

---

### Real-Time Coordination Example

**Scenario:** Worker Agent encounters an encoding error

```text
T+0ms    Worker Agent: Encounters UTF-8 decoding error on row 47,291
T+5ms    Worker Agent: Broadcasts error to swarm via Redis
T+10ms   Validator Agent: Classifies error type (encoding_mismatch)
T+15ms   Worker Agent: Queries pattern library for solution
T+25ms   MABA: Returns known fix (Latin-1 → UTF-8 conversion)
T+30ms   Worker Agent: Applies fix, reprocesses row
T+35ms   Validator Agent: Confirms row passes validation
T+40ms   Worker Agent: Broadcasts successful recovery to swarm
T+45ms   AMANI: Updates error counter in dashboard (no alert - auto-resolved)
T+50ms   Learning Agent: Records recovery pattern for future use
```

Total time: 50ms. No human intervention. All components participated.

---

### Failure Handling

When a component fails, the system degrades gracefully:

| Component Failure  | Impact                                    | Mitigation                                                       |
| ------------------ | ----------------------------------------- | ---------------------------------------------------------------- |
| **MABA instance**  | Transformation paused for affected tables | Other MABA instances continue; Ray auto-scales replacement       |
| **KORA instance**  | Inline validation paused                  | Data continues flowing; validation catches up when KORA recovers |
| **AMANI instance** | Human communication interrupted           | Migration continues; notifications queue for delivery            |
| **Agent instance** | Specific agent capability reduced         | Other agents compensate; Ray auto-scales replacement             |
| **Redis**          | Swarm communication disrupted             | Agents fall back to local operation; sync on recovery            |
| **Temporal**       | Workflow orchestration paused             | In-flight tasks complete; new tasks queue for recovery           |

No single component failure stops the migration. The system is designed for partial degradation.

---

### Observability

All component interactions are observable:

- **Distributed tracing:** OpenTelemetry traces span component boundaries
- **Metrics:** Prometheus metrics for each component and interaction
- **Logs:** Structured logs with correlation IDs
- **Events:** Event stream for audit and debugging

```text
Trace: migration_mig_abc123_phase_3
├── Scout.analyze_table (table=customers, duration=2.3s)
├── MABA.compute_fingerprints (columns=47, duration=0.8s)
├── Architect.generate_mapping (confidence=0.94, duration=1.2s)
├── Worker.transform_batch (rows=10000, duration=4.5s)
│   └── KORA.validate_batch (passed=9987, failed=13, duration=0.6s)
│       └── Worker.recover_errors (recovered=13, duration=0.3s)
└── Meta.extract_patterns (patterns=3, duration=0.2s)
```

→ [Data Flow](data-flow.md)
→ [Agent Communication](agent-communication.md)
→ [Integration Overview](README.md)
