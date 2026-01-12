# Requirements Spec v1.0 (DEPRECATED)

This spec is deprecated. Use `INTENT_SPEC.md`, `CHANGE_SPEC.md`, and
`CHANGE_PACK_SPEC.md` for the current authoring flow.

This spec defines the requirements layer for LLM-assisted change planning.
It provides an incremental disclosure path from vision to stories, then to
change packs and implementation.

---

## Goals

- Human-readable requirements with strict, machine-validated front matter
- Embedded Gherkin acceptance criteria (LLM-friendly, testable)
- Traceability from epics and stories to change packs
- Minimal ceremony, consistent format across teams

## Non-goals

- Replacing change packs or canonical config
- Embedding executable logic in requirements

---

## Layout

```
requirements/
  epic-2025-01-12-001/
    epic.md
    stories/
      story-001.md
      story-002.md
```

---

## epic.md (Required)

`epic.md` MUST include YAML front matter. Keep epics intentionally brief.
Details belong in stories.

Required fields:
- `specVersion` (string, "1.0")
- `id` (string)
- `title` (string)
- `status` ("draft" | "approved" | "in_progress" | "done" | "archived")

Optional fields:
- `owner`, `createdAt`, `summary`, `outcomes`, `constraints`, `nonGoals`, `risks`

Schema: `legacy/schemas/requirements.epic-frontmatter.schema.json`

---

## story.md (Required)

`story.md` MUST include YAML front matter with required fields and a
Gherkin block embedded in the body.

Required front matter fields:
- `specVersion` (string, "1.0")
- `id` (string)
- `epicId` (string)
- `title` (string)
- `status` ("draft" | "approved" | "in_progress" | "done" | "archived")
- `createdAt` (ISO 8601 string)
- `owner` (string)
- `userStory` (string, mandatory)

Optional front matter fields:
- `constraints`, `nonGoals`, `assumptions`, `dependencies`, `risks`,
  `successMetric`, `labels`, `entities`, `openQuestions`

Schema: `legacy/schemas/requirements.story-frontmatter.schema.json`

### Required Gherkin Block

Each story MUST include a `gherkin` code block. To avoid duplicating the
`userStory`, keep the narrative out of Gherkin (the front matter is the
source of truth for the user story). The Gherkin block should focus on
scenarios only.

```gherkin
Feature: Require phone on contacts
  Scenario: Missing phone blocks submission
    Given a new contact without a phone number
    When I submit the form
    Then the submission is rejected
```

Use the Gherkin block as the acceptance criteria source of truth.

---

## Mapping to Change Packs

- Each story can produce one or more change packs.
- Change packs SHOULD include an origin reference to `epicId` and `storyId`.
- Gherkin scenarios can be compiled into tests in `tests.yaml`.

---

## Example story.md

```markdown
---
specVersion: "1.0"
id: story-001
epicId: epic-2025-01-12-001
title: Require phone on contacts
status: draft
createdAt: "2025-01-12T10:00:00Z"
owner: product
userStory: As a sales rep, I want phone required so I can reach contacts.
constraints:
  - No new fields
nonGoals:
  - No UI redesign
---

```gherkin
Feature: Require phone on contacts
  Scenario: Missing phone blocks submission
    Given a new contact without a phone number
    When I submit the form
    Then the submission is rejected
```
```
