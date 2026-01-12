# Verdict Round 8: Change Packs + Requirements Layer

**Date**: 2025-01-12  
**Status**: Consolidation of rounds 7+ discussion  
**Purpose**: Capture naming change, requirements layer, and safe AI posture

---

## Executive Summary

**Updated Feasibility: 77%**

Round 8 keeps the constrained kernel and advisory AI, but tightens the
authoring stack:
- "Intent packs" are now **Change Packs**
- A **requirements layer** (epic/story) sits above change packs
- Gherkin is embedded in story.md (scenario-only, no user story duplication)
- Epic format is simplified to reduce overhead

---

## Key Decisions Since Round 7

### 1) Terminology
- "Intent pack" renamed to **Change Pack** everywhere
- Spec file renamed to `CHANGE_PACK_SPEC.md`
- Schemas renamed to `schemas/change-pack.*`

### 2) Requirements Layer Added
- New `REQUIREMENTS_SPEC.md` with `epic.md` + `story.md`
- Stories include mandatory `userStory` and embedded Gherkin
- Gherkin is scenario-only to avoid duplicating the user story
- Change packs link back via `origin.epicId` and `origin.storyId`

### 3) Epic Simplification
- Epic front matter reduced to the minimum set of required fields
- All detail pushed down into stories

---

## Current Stack (Incremental Disclosure)

1) Vision and principles (why and constraints)
2) Epics (what outcomes)
3) Stories (user story + Gherkin scenarios)
4) Change Packs (diff/tests/diagram)
5) Canonical config (runtime truth)

This preserves determinism while keeping authoring AI-friendly.

---

## USP (Refined)

**AI-safe change management for business logic.**
AI proposes and explains; humans approve; runtime stays deterministic,
auditable, and reversible.

---

## LLM Role (Safe Pattern)

- LLM generates stories, constraints, and draft change packs
- LLM never applies runtime changes directly
- Kernel validates, simulates, and publishes

---

## Updated Risks

### Reduced
- Ambiguity in change pack naming and spec ownership
- Duplication between user story and Gherkin narratives
- Epic overhead for small teams

### Remaining
- Authoring UX still the adoption bottleneck
- Extension determinism must be enforced
- Schema versioning discipline is critical

---

## References (New or Updated)

- `CHANGE_PACK_SPEC.md`
- `REQUIREMENTS_SPEC.md`
- `schemas/change-pack.*`
- `schemas/requirements.*`

