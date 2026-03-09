# Review Context

This file captures recent decisions and constraints so a new reviewer can
quickly align on scope and tradeoffs.

## Goals and Audience

- Build a generic, AI-assisted app platform for non-technical users.
- Demo flow is request -> approval -> fulfillment (not stock/CRM-specific).
- Two modes: Plan (draft only) and Accept (risk-gated auto-apply).

## Determinism and AI Role

- Runtime stays deterministic; AI drafts changes and assists users.
- AI never executes changes directly; humans review and publish.
- End users can use AI for daily work via a safe Command API.

## Runtime Engine (formerly "kernel")

- The runtime engine executes canonical config and enforces rules.
- Business logic lives in config (schemas, workflows, guards, policies),
  not in the runtime engine code.
- The runtime engine exposes deterministic APIs for create/update/transition.

## Config Types (v1)

- Entity schema (fields, types, validation)
- Workflow (state graph, transitions, guards)
- Rules/guards (validation and computed logic)
- RBAC/policies (permissions and access)
- UI schema (layout, widgets, actions)
- Integrations (connectors, notifications)

## Change Packs

- Authoring artifacts only: `intent.md` + `change.md`.
- Human-first, no DSL, no patch language in v1.
- Agents convert change packs into TODOs/implementation plans.

## Data and Performance

- Postgres is the source of truth.
- No event sourcing in v1.
- Redis is optional for caching, not for business logic.

## Workflow Model

- Simple deterministic graph/FSM defined in config.
- Graph UI is for editing only; it is not the runtime.
- No LangGraph runtime dependency.

## Review Focus

- Is the runtime engine scope small and deterministic enough?
- Are the change pack specs clear enough for AI + human authoring?
- Does the demo flow prove generic viability without domain lock-in?
- Do Plan vs Accept gates provide safe speed for non-technical users?

## Open Questions

- Runtime implementation language (Rust vs TypeScript) is undecided.
- Integration catalog breadth and permission model detail are TBD.
