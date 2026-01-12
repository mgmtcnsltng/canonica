# Change Pack Spec v2.0

This spec defines a human-first change pack format. Change packs are
authoring artifacts only. Runtime execution uses the compiled canonical
config, not the change pack itself.

---

## Goals

- Minimal ceremony for non-technical builders
- Clear intent and clear change description
- Safe review and reversible publishing
- AI-friendly input that can be converted into TODO/plan

## Non-goals

- Machine-executable change files
- DSL or patch languages in the pack
- Auto-applying runtime changes without review

---

## Pack Layout

Each change uses a separate pack directory:

```
change-packs/
  change-2025-01-12-001/
    intent.md          # required, follows INTENT_SPEC.md
    change.md          # required, follows CHANGE_SPEC.md
    tests.md           # optional, human notes
    diagram.mmd        # optional, required before publish if workflows change
    logs.md            # optional, logs/examples
```

---

## intent.md (Required)

Intent captures what the user wants and acceptance scenarios.
See `INTENT_SPEC.md`.

---

## change.md (Required)

Change describes current vs expected state per tech stack and includes
targets with `targetType` and `targetName`.
See `CHANGE_SPEC.md`.

---

## tests.md (Optional)

Human-readable test notes or manual validation steps.

---

## diagram.mmd (Optional)

Mermaid diagram for workflow changes. Required before publish if any
workflow is modified.

---

## Legacy Schemas (Deprecated)

Machine-readable schemas in `schemas/change-pack.*` are legacy and optional.
Use v2.0 with `INTENT_SPEC.md` + `CHANGE_SPEC.md` as the source of truth.
