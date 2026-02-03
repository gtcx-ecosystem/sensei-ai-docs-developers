# Web Interface

## The Full Experience

The web interface is AMANI's richest channel — a modern, responsive application providing complete access to Sensei platform capabilities. While [other channels](../../../product/platform/amani/multi-channel.md) provide accessibility across devices and connectivity scenarios, the web interface is where power users do their primary work.

---

### Key Features

#### Interactive Schema Explorer

Visual representation of source and target schemas with:

- Clickable entity-relationship diagrams
- Drill-down from table to column to sample data
- Visual mapping lines showing source-to-target relationships
- Color-coded confidence indicators for AI-suggested mappings
- Hover details showing statistics, inferred types, and transformation logic

#### Real-Time Progress Dashboard

Live migration monitoring with:

- Overall progress percentage and estimated time remaining
- Per-table progress bars with throughput metrics
- Error count and trend visualization
- Resource utilization graphs (CPU, memory, I/O, network)
- Phase timeline showing completed, in-progress, and pending stages

#### Drag-and-Drop Mapping Refinement

Interactive mapping editor allowing:

- Drag source columns to target columns to create/modify mappings
- Visual transformation builder for type conversions
- Side-by-side source/target data preview
- Batch operations for similar mappings
- Undo/redo for all mapping changes

#### Conversational Interface

Chat-style interaction panel with:

- Natural language queries about migration status
- AI explanations of errors and recommendations
- Voice input support for hands-free operation
- Context-aware suggestions based on current view
- Conversation history with search

#### Validation Reports

Comprehensive verification displays with:

- [Multi-source validation](../kora/multi-source-validation.md) results visualization
- Drill-down from summary to specific validation failures
- [Time-Travel Testing](../../../product/platform/kora/time-travel-testing.md) replay results
- Exportable reports for compliance documentation
- Certificate status and download

---

### Technical Implementation

| Component          | Technology           | Purpose                   |
| ------------------ | -------------------- | ------------------------- |
| Frontend framework | React 18             | Component-based UI        |
| State management   | Redux Toolkit        | Application state         |
| Real-time updates  | WebSocket            | Live progress, metrics    |
| Visualization      | D3.js + React Flow   | Schemas, graphs, diagrams |
| Styling            | Tailwind CSS         | Responsive design         |
| API communication  | GraphQL + REST       | Data fetching             |
| Offline support    | Service Worker (PWA) | 7-day offline capability  |

---

### Progressive Web App (PWA)

The web interface is installable as a PWA, providing:

- **Offline capability:** 7 days of cached operation
- **Native-like experience:** Installable on desktop and mobile
- **Push notifications:** Migration alerts even when browser is closed
- **Background sync:** Queued actions execute when connectivity returns

PWA installation is optional — the interface works fully in any modern browser.

---

### Responsive Design

The interface adapts to screen size:

| Viewport            | Layout                                             | Primary Use Case |
| ------------------- | -------------------------------------------------- | ---------------- |
| Desktop (>1200px)   | Three-panel: navigation, main content, detail/chat | Primary work     |
| Tablet (768-1200px) | Two-panel: collapsible navigation, main content    | Mobile work      |
| Mobile (<768px)     | Single-panel: bottom navigation, stacked views     | Quick checks     |

All features are accessible at all screen sizes, though some complex visualizations recommend desktop for best experience.

---

### Accessibility

The web interface follows WCAG 2.1 AA guidelines:

- **Keyboard navigation:** All functions accessible without mouse
- **Screen reader support:** ARIA labels and roles throughout
- **Color contrast:** Minimum 4.5:1 ratio for text
- **Focus indicators:** Clear visual focus states
- **Motion preferences:** Respects reduced-motion settings
- **Text scaling:** Functional up to 200% zoom

---

### Performance Targets

| Metric                             | Target     | Current |
| ---------------------------------- | ---------- | ------- |
| Initial load (cold)                | <3s        | 2.4s    |
| Initial load (cached)              | <1s        | 0.8s    |
| Time to interactive                | <2s        | 1.6s    |
| Dashboard update latency           | <500ms     | 320ms   |
| Schema visualization (1000 tables) | <2s render | 1.4s    |
| Search response                    | <200ms     | 150ms   |

Performance budgets are enforced in CI/CD — builds that exceed targets are flagged for review.

---

### Browser Support

| Browser       | Minimum Version | Notes                     |
| ------------- | --------------- | ------------------------- |
| Chrome        | 90+             | Full support, recommended |
| Firefox       | 90+             | Full support              |
| Safari        | 14+             | Full support              |
| Edge          | 90+             | Full support              |
| Mobile Chrome | 90+             | Full support              |
| Mobile Safari | 14+             | Full support              |

Internet Explorer is not supported. Users on older browsers see a message recommending upgrade.

→ [Multi-Channel Communication](../../../product/platform/amani/multi-channel.md)
→ [How AMANI Works](how-it-works.md)
→ [AMANI Overview](../../../product/platform/amani/README.md)
