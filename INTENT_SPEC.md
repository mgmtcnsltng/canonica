# Intent Spec v1.0

This spec defines the human/LLM intent layer for a change. It captures
what the user wants, without implementation detail.

---

## Required Sections

- Summary (one sentence)
- User Story (mandatory)
- Acceptance Scenarios (Gherkin text, mandatory)

Gherkin example (plain text, no code fences required):

Feature: <short title>
  Scenario: <scenario name>
    Given ...
    When ...
    Then ...

---

## Recommended Sections

- Constraints (what must not change)
- Out of scope (explicit exclusions)
- Open questions (unknowns to resolve)

---

## Example

# Intent

## Summary
Require approval before fulfillment.

## User Story
As an ops manager, I want approvals before fulfillment so I can prevent mistakes.

## Acceptance Scenarios (Gherkin)
Feature: Approval required before fulfillment
  Scenario: Fulfillment blocked without approval
    Given a request in "requested"
    When I attempt to fulfill it
    Then the action is rejected
