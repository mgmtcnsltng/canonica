# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an **AI-Native Generic App Platform** - a greenfield project in the design/specification phase. The platform enables non-technical users to build and operate ERP/CRM/internal apps through AI assistance while maintaining deterministic, safe runtime execution.

**Core Philosophy**: AI edits intent. Runtime executes truth.

The platform is **generic** - stocktake, ERP, CRM are templates built on top, not the platform itself.

## Architecture Principles

### Canonical Model (The Core Concept)

The **Canonical Model** is the single, authoritative, executable representation of system behavior. It is:
- Deterministic
- Schema-validated
- Versioned
- Runtime-safe

All authoring (UI edits, AI prompts, diagrams, Markdown) **compiles into canonical models before execution**. This is the bytecode that actually runs.

### Workflow vs Rules (Critical Separation)

These two concerns must **never** be mixed:

**Workflow** answers: "Where am I, and where can I go next?"
- Finite state machine
- Long-lived state (e.g., `draft → counting → submitted → approved`)
- Controls lifecycle and transitions
- Stateful

**Rules** answer: "Is this valid, and what should happen?"
- Stateless
- Evaluated on events
- Controls validation, thresholds, derived fields
- Pure logic

Mixing workflow and rules leads to unsafe AI edits and unmaintainable logic.

### Authoring vs Execution

**Authoring Layer** (Human + AI):
- SKILL.md-style Markdown
- Human-readable, AI-editable
- Diff-friendly
- Explains intent

**Execution Layer** (Runtime):
- Strict JSON canonical models
- Schema-validated, deterministic
- Executed by rules/workflow engines

Mental model: `SKILL.md → Canonical JSON → Runtime Engine`

## Tech Stack

### Frontend
- **Next.js** - Workspace UI + PWA
- **React Flow** - Visual workflow editor (NOT runtime, NOT storage)
- Schema-driven UI renderer - Metadata-driven UI generation
- Embedded AI copilot

### Backend
- **Command API** - Writes (create, update, transition)
- **Query/Metrics API** - Reads (Postgres for operational, ClickHouse for analytics via PeerDB)
- **Rules engine adapter** - Executes rules
- **Workflow engine** - Manages state machines
- **Versioning & publishing service** - Draft → Validate → Simulate → Diff → Preview → Publish

### Data
- **PostgreSQL** - Source of truth + metadata storage
- **ClickHouse** - Analytics
- **Audit/Event log** - Comprehensive audit trail

### AI
- **LangChain**
  - React Agent - Operations (end-users: data entry, reporting)
  - DeepAgent - Builders (admins: modify platform behavior)

## Key Patterns

### Command Query Separation
- **Command API**: All writes go through here (never direct DB writes)
- **Query/Metrics API**: Separate read paths

### Schema-Driven UI
- Data schema defines fields, types, validation
- UI schema defines layout, widgets, actions
- Next.js acts as a renderer, not logic holder

### React Flow Usage
- React Flow is a **visual editor only**
- Canonical workflow JSON is source of truth
- Graph is derived from canonical model
- Graph edits update canonical model

### AI Agents

**Operator Agent** (LangChain React Agent):
- Used by end-users
- Generic tools: `entity.create`, `entity.update`, `entity.transition`, `metrics.query`

**Builder Agent** (LangChain DeepAgent):
- Used by admins
- Can propose schema changes, rules, workflows, UI definitions
- Generates PRs/drafts only (never auto-deploys)

Same tools, different permissions.

### Safety & Governance

Change lifecycle: `Draft → Validate → Simulate → Diff → Preview → Publish`

- Runtime only reads **published** versions
- Full audit log required
- Rollback must be supported

## Development Status

This is currently a **design-phase project** - no source code exists yet. The `HIGH_LEVEL_PLAN.md` contains the comprehensive specification.

## Planned Delivery Phases

### Phase 1 - Foundation
- Command API + audit log
- Entity + workflow + rule canonical schemas
- Schema-driven UI renderer

### Phase 2 - Stocktake Template
- Stocktake entities & workflow
- AI-assisted data entry
- Ops Agent

### Phase 3 - Builder AI
- SKILL.md authoring
- DeepAgent propose → preview → PR
- Workflow editor (React Flow)

### Phase 4 - Generalisation
- Template import/export
- Plugin registries
- Open-source extraction

## Open Source Strategy

The **platform** is open-sourced, not the apps. Apps (stocktake/CRM/ERP) are templates.

Strong OSS candidates:
- Canonical rules/workflow engine
- SKILL.md → JSON compiler
- Schema-driven UI renderer
- AI-safe Command API pattern
- Agent tool contracts

## Core Mental Model

```
Workflow = Where can this go?
Rules    = Is this allowed and why?
Canonical = The only thing that executes
AI        = Author intent, never truth
```

This separation is what makes the system **generic, AI-driven, safe, and production-grade**.

## License

Apache License 2.0
