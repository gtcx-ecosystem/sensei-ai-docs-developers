# Change Management and User Adoption

## Guiding Users Through Change

AMANI employs progressive disclosure that adapts the complexity of its guidance to each user's demonstrated expertise. First-time users receive step-by-step walkthroughs. Experts receive concise, command-driven interaction. The system learns and adapts automatically.

---

### Progressive Disclosure Model

#### Level 1: Guided Mode (New Users)

For users new to the platform or to data migration:

- **Step-by-step walkthroughs** with explanations of each concept
- **Contextual definitions** for technical terms ("A foreign key is a relationship between tables...")
- **Confirmation prompts** before any significant action
- **Visual guides** showing what will happen and why
- **Proactive explanations** of what AMANI is doing and why

**Example interaction:**

> **User:** Start the migration
>
> **AMANI:** Before we begin, let me walk you through what will happen:
>
> 1. **Analysis phase** (~40 minutes): I'll examine your source database to understand its structure, find relationships between tables, and identify any potential issues.
> 2. **Planning phase** (~10 minutes): Based on the analysis, I'll create a migration plan showing the order tables will be migrated and any transformations needed.
> 3. **Review checkpoint**: You'll have a chance to review and approve the plan before any data moves.
>
> Ready to start the analysis phase? [Start Analysis] [Learn More First]

---

#### Level 2: Standard Mode (Familiar Users)

For users who demonstrate familiarity through faster responses, technical terminology, or explicit preference:

- **Concise guidance** omitting basic explanations
- **Grouped confirmations** for related decisions
- **Technical details available on request** but not forced
- **Fewer interruptions** for routine checkpoints

**Example interaction:**

> **User:** Start the migration
>
> **AMANI:** Starting migration: HR_PROD (Oracle 19c) → hr_production (PostgreSQL 16). Analysis will take ~40 minutes. I'll notify you when the migration plan is ready for review. Proceed? [Start] [Configure Options]

---

#### Level 3: Expert Mode (Power Users)

For users who demonstrate deep expertise or request minimal guidance:

- **Command-driven interaction** with minimal confirmation
- **Only decision points and anomalies** interrupt the workflow
- **Batch approvals** for routine decisions
- **Direct access** to logs, metrics, and configuration
- **Abbreviated status updates** unless detail requested

**Example interaction:**

> **User:** migrate HR_PROD to hr_production --strategy=balanced --notify=completion
>
> **AMANI:** Migration queued. ETA: 24h. Will notify on completion. [View Status]

---

### Automatic Adaptation

AMANI infers user expertise from multiple signals:

| Signal                  | Interpretation                                                               |
| ----------------------- | ---------------------------------------------------------------------------- |
| Response time           | Fast responses suggest familiarity                                           |
| Vocabulary              | Technical terms indicate expertise                                           |
| Question patterns       | Clarification questions suggest learning; direct commands suggest expertise  |
| Decision confidence     | Users who approve quickly without reviewing details may prefer less guidance |
| Historical interactions | Past sessions inform current expectations                                    |

Adaptation happens automatically and continuously. A user who starts in Guided mode might progress to Standard mode within a single session as they demonstrate competence.

---

### Manual Override

Users can override the automatic detection at any time:

- **"Explain more"** — Temporarily increase detail level
- **"Skip the explanations"** — Temporarily reduce detail level
- **Settings → Guidance Level** — Set permanent preference
- **"Treat me as a beginner"** — Reset to Guided mode for unfamiliar areas

The override applies immediately and AMANI acknowledges the preference change.

---

### Contextual Expertise

Users may be experts in some areas and novices in others. AMANI tracks expertise **per domain**:

| Domain             | Example Topics                             | Expertise Tracked Separately |
| ------------------ | ------------------------------------------ | ---------------------------- |
| Source systems     | Oracle, SQL Server, PostgreSQL specifics   | Yes                          |
| Target systems     | Snowflake, PostgreSQL, cloud databases     | Yes                          |
| Migration concepts | Schema mapping, transformation, validation | Yes                          |
| Compliance         | GDPR, HIPAA, SOX requirements              | Yes                          |
| Platform operation | AMANI commands, configuration, dashboards  | Yes                          |

A user might be an Oracle expert (Expert mode for Oracle topics) but new to Snowflake (Guided mode for Snowflake topics). AMANI adapts accordingly.

---

### Onboarding Flow

First-time users receive an optional onboarding experience:

1. **Welcome** — Introduction to AMANI and the Sensei platform
2. **Role selection** — Technical operator, business stakeholder, compliance, executive
3. **Experience assessment** — Self-reported familiarity with migration concepts
4. **Channel preferences** — Which channels for which notification types
5. **Quick tour** — Interactive walkthrough of key features
6. **First task** — Guided completion of a simple task to build confidence

Onboarding takes ~5 minutes and can be skipped by experienced users.

---

### Training Mode

For organizations onboarding multiple users, AMANI offers a training mode:

- **Sandbox environment** — Practice migrations on sample data with no production risk
- **Guided scenarios** — Pre-built exercises covering common workflows
- **Progress tracking** — Administrators can see who has completed training
- **Certification** — Optional assessment confirming readiness for production access

Training mode helps organizations scale their migration capability without requiring extensive classroom training.

→ [Stakeholder Communication](../../../product/platform/amani/stakeholder-communication.md)
→ [How AMANI Works](how-it-works.md)
→ [AMANI Overview](../../../product/platform/amani/README.md)
