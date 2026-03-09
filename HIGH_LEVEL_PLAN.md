# AI‑Native Generic App Platform

This document summarises the **requirements, architecture, tech stack, and delivery plan** for a **generic, AI‑driven platform** that allows non‑technical users to build and operate ERP/CRM/internal apps, while remaining deterministic, safe, and developer‑extensible.

---

## 1. Vision

Build a **generic platform**, not a vertical-specific app.

- Reference demo flow (request → approval → fulfillment), ERP, CRM are **templates** built on top
- Business logic is **AI‑authorable** but **deterministic at runtime**
- Humans and AI collaborate safely
- Developers can extend the platform via open primitives

Core philosophy:

> **AI edits intent. Runtime executes truth.**

---

## 2. Core Requirements

### Functional

- Non‑technical users can:
  - enter data with AI assistance
  - operate daily workflows via UI or chat
  - generate reports using natural language

- Business admins can:
  - modify rules, workflows, and UI via AI + visual editors
  - preview, simulate, diff, and approve changes

- Developers can:
  - extend logic via registries and plugins
  - add widgets, actions, connectors

### Non‑functional

- Deterministic execution
- Strong audit trail
- Versioning and rollback
- Role‑based access and RLS
- Generic (no hard‑coded domain logic)
- Open‑source friendly

---

## 3. Canonical Model (Key Concept)

### Definition

The **Canonical Model** is the **single, authoritative, executable representation** of system behaviour.

Properties:
- deterministic
- schema‑validated
- versioned
- runtime‑safe

Everything (UI edits, AI prompts, diagrams, Markdown) **compiles into canonical models before execution**.

### Runtime Engine Scope

- Executes canonical config deterministically; no embedded business logic.
- Exposes Command/Query APIs; AI uses them but cannot bypass validation.
- Core config types (v1): entity schema, workflow, rules/guards, RBAC/policies,
  UI schema, integrations/notifications.
- Workflow is a simple deterministic graph/FSM; no LangGraph runtime.

---

## 4. Workflow vs Rules (Clear Separation)

### Workflow

Answers:
> *Where am I, and where can I go next?*

- Finite state machine
- Long‑lived state
- Controls lifecycle and transitions

Example:
```
draft → counting → submitted → approved
```

### Rules

Answers:
> *Is this valid, and what should happen?*

- Stateless
- Evaluated on events
- Controls validation, thresholds, derived fields

### Why separate?

| Workflow | Rules |
|--------|------|
| Lifecycle | Validation & logic |
| Stateful | Stateless |
| Navigation | Conditions |
| UI actions | Field behaviour |

Mixing them leads to unsafe AI edits and unmaintainable logic.

---

## 5. Authoring vs Execution

### Authoring Layer (Human + AI)

- **Change Pack: intent.md + change.md**
- Human‑readable
- AI‑editable
- Diff‑friendly
- Explains intent

### Execution Layer (Runtime)

- **Strict JSON canonical models**
- Schema‑validated
- Deterministic
- Executed by rules/workflow engines

Mental model:

```
Intent + Change  →  Canonical Config  →  Runtime Engine
 (source)           (bytecode)
```

---

## 6. Workflow UI

- **React Flow** used as a **visual editor only**
- Canonical workflow JSON is source of truth
- React Flow graph is derived from canonical model
- Graph edits update canonical model

React Flow is:
- ✅ great for UX
- ❌ not runtime
- ❌ not storage

---

## 7. UI Architecture

- Schema‑driven UI
- Defined by metadata, not code

### Components
- Data schema (fields, types, validation)
- UI schema (layout, widgets, actions)

Next.js acts as a **renderer**, not a logic holder.

---

## 8. AI Agents (Generic)

### Operator Agent (LangChain React Agent)

- Used by end‑users
- Data entry, updates, reporting
- Uses generic tools:
  - entity.create / update
  - entity.transition
  - metrics.query
- Always via the Command API with role-based permissions

### Builder Agent (LangChain DeepAgent)

- Used by admins
- Modifies platform behaviour
- Can propose:
  - schema changes
  - rules
  - workflows
  - UI definitions
- Generates PRs / drafts only
- Cannot auto‑deploy

Same tools. Different permissions.

---

## 9. Safety & Governance

### Change lifecycle

```
Draft → Validate → Simulate → Diff → Preview → Publish
```

- Runtime only reads **published** versions
- Full audit log
- Rollback supported
- AI has two modes:
  - **Plan**: draft + review required
  - **Accept**: auto‑apply allowed for low‑risk changes (risk‑gated)

---

## 10. Data Entry & Reporting

### Data Entry

- Chat‑to‑form
- Bulk natural language entry
- Photo/document ingestion
- Always via **Command API**
- Never direct DB writes

### Reporting

- Natural language → approved metrics
- Postgres: operational queries
- ClickHouse: analytics (via PeerDB)

---

## 11. Tech Stack

### Frontend
- Next.js (workspace UI + PWA)
- React Flow (workflow editor)
- Schema‑driven renderer
- Embedded AI copilot

### Backend
- Command API (writes)
- Query / Metrics API (reads)
- Rules engine adapter
- Workflow engine
- Versioning & publishing service

### Data
- PostgreSQL (source of truth + metadata)
- Redis (optional cache, not source of truth)
- ClickHouse (analytics)
- Audit / event log
- No event sourcing in v1

### AI
- LangChain
  - React Agent (Ops)
  - DeepAgent (Builder)

---

## 12. Open‑Source Strategy

### Worth open‑sourcing

Primary opportunity:

> **AI‑native business logic platform**, not ERP itself.

Strong OSS candidates:
- Canonical rules/workflow engine
- Change pack tooling (review, diff, plan generation)
- Schema‑driven UI renderer
- AI‑safe Command API pattern
- Agent tool contracts

Apps (demo flow/CRM/ERP) are templates.

---

## 13. Delivery Plan (Practical)

### Phase 1 – Foundation
- Command API + audit log
- Entity + workflow + rule canonical schemas
- Schema‑driven UI renderer

### Phase 2 – Generic Demo (Request → Approval → Fulfillment)
- Demo entities & workflow
- AI‑assisted data entry
- Ops Agent

### Phase 3 – Builder AI
- Change pack authoring (intent.md + change.md)
- AI propose → preview → PR
- Workflow editor (React Flow)

### Phase 4 – Generalisation
- Template import/export
- Plugin registries
- Open‑source extraction

---

## 14. Core Mental Model (Final)

```
Workflow = Where can this go?
Rules    = Is this allowed and why?
Canonical = The only thing that executes
AI        = Author intent, never truth
```

This separation is what makes the system **generic, AI‑driven, safe, and production‑grade**.

---

## 15. Reference Demo (Request → Approval → Fulfillment)

Use a simple request/approval/fulfillment flow as the generic demo to
pressure-test the runtime engine.

### Entities
- Request
- RequestLine
- Fulfillment

### Workflow
```
draft → requested → approved → fulfilled → closed
```

### Guards and Actions
- Prevent request if required fields are missing
- Require approval above a configurable threshold
- Only approvers can approve
- On fulfill: create a fulfillment record and notify requester

### AI Usage (Build + Use)
- Build: generate schemas, UI, and workflow from natural language
- Use: chat‑to‑form entry, summarize status, and draft fulfillment notes
