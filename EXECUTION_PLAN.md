# Execution Plan (Draft)

This plan follows the sequence in `TODO.md` and focuses on the generic
request → approval → fulfillment demo.

---

## Phase 0: Setup (Week 0)

- Decide repo structure for app + packages.
- Optional: add `docker-compose.yml` for Postgres.
- Define initial shadcn/ui components to install (form, select, table, dialog).

Deliverable: runnable dev environment and UI component baseline.

---

## Phase 1: Demo Scope + Runtime Engine Core (Weeks 1-2)

### Demo Scope
- Entities: Request, RequestLine, Fulfillment.
- Workflow: draft → requested → approved → fulfilled → closed.
- Guards: approval required, role-based transitions.

### Runtime Engine Core
- Entity schema storage + validation.
- Workflow engine with guarded transitions.
- Command API boundary (writes) + Query API (reads).
- Audit log and versioning.
- RBAC enforcement.

Deliverable: deterministic runtime engine with a working demo workflow.

---

## Phase 2: Change Pack Flow (Weeks 3-4)

- Implement change pack ingestion (`intent.md` + `change.md`).
- Preview/diff output for change packs.
- Simulation runner (basic).
- Publish pipeline with rollback.

Deliverable: safe, reviewable change pipeline.

---

## Phase 3: Builder UX (Weeks 5-6)

- Visual schema editor (shadcn/ui inputs per field type).
- Workflow editor (state list + transitions + guards).
- Guard builder UI with validation hints.

Deliverable: non-technical builder can modify the demo without JSON.

---

## Phase 4: AI Modes + Operator Assist (Weeks 7-8)

- Plan vs Accept mode toggle with risk gating.
- AI drafts intent/change packs from user input.
- Operator AI: chat-to-form entry and status summaries via Command API.

Deliverable: AI-assisted build and use loop for the demo.

---

## UI Component Mapping (Initial)

- string → `Input`
- text/notes → `Textarea`
- enum → `Select`
- boolean → `Checkbox`
- date → `Input[type=date]` or `DatePicker`
- money/number → `Input` with formatting

---

## Success Criteria

- Non-technical user can create a request and move it through approval and fulfillment.
- Change packs show diff/simulate before publish.
- Accept mode only applies low-risk changes.
