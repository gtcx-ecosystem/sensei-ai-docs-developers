---
description: AMANI support interface internals
---

# How AMANI Works

## Message to Action to Response

This page walks through AMANI's complete communication flow — from receiving a user message to coordinating with agents to delivering a response across any channel.

> For platform overview and business context, see [AMANI Overview](../../../product/platform/amani/README.md).

---

### The Communication Loop

```text
User Message (any channel)
         ↓
┌─────────────────────────────────────┐
│ 1. CHANNEL RECEPTION                 │
│    Parse message from channel format │
│    Extract intent and context        │
└─────────────────────────────────────┘
         ↓
┌─────────────────────────────────────┐
│ 2. LANGUAGE PROCESSING               │
│    Detect language (message-level)   │
│    Translate if needed               │
│    Parse natural language intent     │
└─────────────────────────────────────┘
         ↓
┌─────────────────────────────────────┐
│ 3. CONVERSATION STATE                │
│    Load session context              │
│    Update with new message           │
│    Determine conversation stage      │
└─────────────────────────────────────┘
         ↓
┌─────────────────────────────────────┐
│ 4. AGENT COORDINATION                │
│    Route to appropriate agent(s)     │
│    Execute agent actions             │
│    Collect agent responses           │
└─────────────────────────────────────┘
         ↓
┌─────────────────────────────────────┐
│ 5. RESPONSE GENERATION               │
│    Adapt to stakeholder role         │
│    Adapt to expertise level          │
│    Translate to user's language      │
│    Format for channel constraints    │
└─────────────────────────────────────┘
         ↓
Response delivered (same or different channel)
```

---

### Step 1: Channel Reception

Each channel has different message formats, constraints, and capabilities:

| Channel      | Message Format                  | Constraints        | Capabilities                      |
| ------------ | ------------------------------- | ------------------ | --------------------------------- |
| **Web**      | Rich JSON (text, files, images) | None significant   | Full interactivity, drag-and-drop |
| **WhatsApp** | Text, images, documents         | 4,096 char limit   | Inline buttons, quick replies     |
| **SMS**      | Plain text                      | 160 char segments  | Numeric option selection          |
| **USSD**     | Menu codes                      | 182 char limit     | Hierarchical menus                |
| **Voice**    | Speech audio                    | Linear interaction | Speech-to-text, text-to-speech    |

AMANI normalizes all incoming messages into a common internal format.

---

### Step 2: Language Processing

AMANI performs language detection at the **message level**, not the session level. This is critical for multilingual business environments where users naturally code-switch between languages within a single conversation.

For complex intents, AMANI parses the natural language into structured actions:

- "Pause the migration" → `ACTION: pause_migration`
- "Show me errors from the last hour" → `QUERY: errors, time_filter: last_1h`
- "Why did the customer table fail?" → `QUERY: error_explanation, table: customer`

---

### Step 3: Conversation State

AMANI maintains a unified conversation state across all channels. A user can start a conversation on the web interface, continue via WhatsApp when away from their desk, and receive critical alerts via SMS — all as one continuous conversation.

The state engine tracks:

- **Session context:** Current migration, current phase, pending decisions
- **Conversation history:** Recent messages and responses
- **User profile:** Detected role, expertise level, language preferences
- **Pending actions:** Decisions awaiting user confirmation

---

### Step 4: Agent Coordination

Based on the parsed intent, AMANI routes to the appropriate Sensei agents:

| Intent Category         | Routed To              | Example                                             |
| ----------------------- | ---------------------- | --------------------------------------------------- |
| Status inquiry          | Monitor Agents         | "What's the current progress?"                      |
| Technical question      | Scout Agents           | "Why is the date column failing?"                   |
| Decision confirmation   | Architect Agents       | "Yes, use the safe migration strategy"              |
| Ambiguous mapping       | Resolver Agents        | "Which target column should this map to?"           |
| Error investigation     | Worker/Recovery Agents | "Show me the last 10 errors"                        |
| Validation check        | Validator Agents       | "Has the customer table passed verification?"       |
| Scheduling/coordination | Conductor Agents       | "Run the next migration batch tonight"              |
| Learning insights       | Learning Agents        | "What patterns have we seen in similar migrations?" |

AMANI translates agent responses into human-readable explanations.

---

### Step 5: Response Generation

AMANI adapts responses to three factors:

**Stakeholder role:** Technical operators receive precise descriptions. Business stakeholders receive outcome-focused summaries. Executives receive high-level dashboards.

**Expertise level:** First-time users receive detailed explanations. Expert users receive command-mode responses with minimal explanation.

**Channel constraints:** A web response might include an interactive table. The same information sent via SMS becomes a numbered summary.

→ [Multi-Channel Communication](../../../product/platform/amani/multi-channel.md)
→ [Language Intelligence](../../../product/platform/amani/language-intelligence.md)
→ [Stakeholder Communication](../../../product/platform/amani/stakeholder-communication.md)
