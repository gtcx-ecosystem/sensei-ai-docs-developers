---
description: Low-bandwidth and offline environments
---

# Frontier Markets

## Serving Where Others Can't — Or Won't

AMANI is designed for deployments where existing enterprise tools fail: frontier markets and developing economies where reliable internet connectivity, smartphone penetration, and technical literacy cannot be assumed.

---

### The Opportunity

Legacy system modernization is most urgently needed in developing economies:

- **Government digitization:** Paper-based systems being migrated to digital databases
- **Financial inclusion:** Informal record-keeping moving to formal systems
- **Infrastructure modernization:** Post-colonial legacy systems finally being replaced
- **Regional integration:** ECOWAS, EAC, ASEAN driving cross-border system harmonization

These migrations are underserved because existing tools assume:

- Always-on broadband connectivity
- Technical staff fluent in English
- Modern smartphones or laptops for all users
- Silicon Valley infrastructure standards

AMANI assumes none of these things.

---

### Infrastructure Reality

| Assumption      | Silicon Valley              | Frontier Reality                |
| --------------- | --------------------------- | ------------------------------- |
| Connectivity    | 1 Gbps fiber, 99.99% uptime | 2G/3G mobile, intermittent      |
| Devices         | Latest iPhone/MacBook       | Feature phones, shared devices  |
| Power           | Uninterrupted               | Load shedding, generator backup |
| Technical staff | Abundant, specialized       | Scarce, generalist              |
| Language        | English                     | 50+ local languages             |
| Support hours   | 24/7 remote                 | Business hours, local timezone  |

AMANI is built for the right column.

---

### Addressing Each Challenge

#### Intermittent Connectivity

- [Offline-first architecture](../../../product/platform/amani/offline-first.md) enables 30+ days without server contact
- [CRDT synchronization](crdt-sync.md) merges offline work automatically
- Progressive sync prioritizes critical data
- 2G-compatible data payloads (compressed, incremental)

#### Device Diversity

- [Five-channel architecture](../../../product/platform/amani/multi-channel.md) reaches any device
- USSD works on any GSM phone (no smartphone required)
- SMS templates for basic interaction
- Web PWA works on low-end Android browsers
- Feature phone support for 40%+ of users in many markets

#### Power Reliability

- Aggressive checkpointing (every 10K records default, configurable lower)
- Resume-from-checkpoint for interrupted migrations
- Battery-aware mobile clients (reduce sync frequency on low battery)
- Graceful degradation when power is lost mid-operation

#### Technical Capacity

- [Progressive disclosure](change-management.md) adapts to user expertise
- Step-by-step guided workflows for non-technical users
- Local language support for [200+ languages](../../../product/platform/amani/language-intelligence.md)
- Training mode with sandbox environment

#### Language

- [Message-level language detection](../../../product/platform/amani/language-intelligence.md)
- Particular investment in underserved African and Asian languages
- Domain-specific glossaries with culturally appropriate explanations
- Code-switching support for multilingual environments

---

### Regional Focus Areas

#### West Africa (ECOWAS)

**Languages:** Hausa, Yoruba, Igbo, Akan, Wolof, French, English, Portuguese
**Use cases:** Government record digitization, financial system modernization, cross-border trade systems
**Infrastructure:** GSM coverage good, data coverage limited outside cities, frequent power outages

#### East Africa (EAC)

**Languages:** Swahili, Amharic, Oromo, Somali, English
**Use cases:** M-Pesa ecosystem migrations, government services, agricultural systems
**Infrastructure:** Mobile-first region, M-Pesa integration expectations, solar power increasingly common

#### South Asia

**Languages:** Hindi, Bengali, Tamil, Telugu, Urdu, Marathi, Gujarati, plus 20+ others
**Use cases:** Government digital transformation (India Stack), banking modernization, state-level system integration
**Infrastructure:** Variable by region, strong mobile penetration, improving 4G coverage

#### Southeast Asia (ASEAN)

**Languages:** Thai, Vietnamese, Bahasa Indonesia/Malay, Tagalog, Khmer, Burmese
**Use cases:** Regional economic integration, government modernization, financial inclusion
**Infrastructure:** Rapid improvement, but rural areas still underserved

---

### Pricing Considerations

Frontier market customers often have:

- Lower absolute budgets
- Higher relative need (modernization is critical, not optional)
- Different payment patterns (government procurement cycles, grant funding)
- Currency volatility concerns

Sensei offers:

- **Regional pricing tiers** reflecting purchasing power parity
- **Government/NGO discounts** for public sector modernization
- **Multi-year agreements** with fixed pricing for budget predictability
- **Local currency billing** where feasible
- **Success-based pricing options** for risk-sharing

---

### Local Partnerships

Sensei works with local partners for:

- **Implementation support:** Local consulting partners who understand regional context
- **Language localization:** Native speakers for glossary development and testing
- **Infrastructure:** Regional cloud presence (AWS Africa, Google Cloud Africa)
- **Payment processing:** Local payment methods and currencies
- **Support coverage:** Local timezone support teams

---

### Why This Matters Strategically

Frontier markets aren't a charity case — they're a strategic opportunity:

1. **First-mover advantage:** No serious competition in these markets
2. **Compound intelligence:** Every migration adds to the pattern library
3. **Government relationships:** Early presence creates long-term partnerships
4. **Market growth:** Fastest-growing economies for enterprise software
5. **Talent pipeline:** Local technical capacity building creates future customers and partners

Serving frontier markets isn't just accessible with AMANI's architecture — it's a core competitive advantage.

→ [Offline-First Architecture](../../../product/platform/amani/offline-first.md)
→ [Language Intelligence](../../../product/platform/amani/language-intelligence.md)
→ [Multi-Channel Communication](../../../product/platform/amani/multi-channel.md)
