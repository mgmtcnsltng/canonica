# Verdict Round 7: Stable Intent-Pack Spec v1.1

**Date**: 2025-01-12  
**Status**: Consolidated architecture + spec alignment  
**Purpose**: Resolve Round 6 tradeoffs and lock the ecosystem-facing format

---

## Executive Summary

**Updated Feasibility: 77%**

Round 6 improved AI robustness, but introduced two strategic risks:
1) an expanding operation DSL, and 2) fragmented pack tiers.
Round 7 resolves both by adopting a **single, stable pack format** and
**fragment changes** (schema-aware merges) as the recommended mode.

**Verdict**: Proceed with the constrained kernel + advisory AI, using
change-pack v1.1 as the external integration contract.

---

## What Changed Since Round 6

- **No tiers**: one pack format for the ecosystem, reducing tooling complexity
- **Fragment changes**: order-independent merges replace op DSL as the default
- **Auto artifacts**: tests/diagrams can be generated to reduce ceremony
- **Base version safety**: `baseConfigVersion` supports stale-pack detection

---

## Why This Is Better

### Counter-argument to Round 6
- Declarative operations become a new DSL with version churn
- Tiered packs fragment external tooling and increase AI misclassification
- Two-step translation (intent -> ops -> config) reduces review clarity

### Round 7 Resolution
- **Fragment changes** are expressive without becoming a DSL
- **Single pack format** simplifies validation and ecosystem adoption
- **Direct config diff** is clearer for human review and approvals

---

## Round 7 Canonical Flow

```
Change Pack (v1.1)
   └─ manifest + intent + fragment changes + tests/diagram
         └─ validate against kernel + schemas
               └─ generate preview (diff + diagram + tests)
                     └─ simulate + approve
                           └─ publish (deterministic runtime)
```

AI edits **intent and constraints**, not runtime config. The kernel remains
the only authority that validates and publishes changes.

---

## Updated Risk Assessment

### Reduced Risks
- **AI brittleness**: fragment merges tolerate structure drift
- **Ecosystem fragmentation**: single pack format
- **Review friction**: auto-generated tests/diagrams

### Remaining Risks
- **Authoring UX**: still the dominant adoption risk
- **Extension determinism**: plugin/function behavior must be explicit
- **Schema drift**: requires strict versioning and compatibility policy

---

## Updated Feasibility Verdict

**77%** if:
- The kernel spec remains small and deterministic
- Fragment changes are validated strictly
- Builder UX reaches non-technical usability

**<60%** if:
- The fragment merge language expands into a complex DSL
- Extensions are allowed to mutate state without governance
- Pack versioning is not enforced

---

## Implementation Notes (Additions)

- Pack validator must understand fragment changes and `baseConfigVersion`
- Diff generator should compare `before/after` configs for review
- Diagram generator should run when workflow targets are present
- Tests can be auto-generated when `tests.yaml` is `auto: true`

---

## Final Verdict

Round 7 locks a **stable, AI-friendly, ecosystem-ready** pack format without
creating a new DSL or fragmenting tooling. This preserves determinism while
keeping authoring practical.

**Proceed with:**
- Constrained kernel
- Intent-pack v1.1 (single format, fragment changes)
- Advisory AI only
