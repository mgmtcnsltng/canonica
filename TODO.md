# TODO

## Do (with why)

- Build the constrained kernel (entity schema, schema-driven UI, guarded workflows, audit/versioning, RBAC).  
  Why: proves deterministic runtime and generic behavior without a DSL.
- Implement Plan vs Accept AI modes with explicit risk gating.  
  Why: non-technical builders need AI speed but require safety and reversibility.
- Adopt Change Pack v2.0 (intent.md + change.md + optional tests/diagram/logs).  
  Why: human-first changes that are clear to review and easy for AI to draft.
- Deprecate the requirements layer (epic/story) in favor of intent + change.  
  Why: reduce ceremony for non-technical builders.
- Build the generic demo app (request → approval → fulfillment).  
  Why: validates the kernel and builder UX without vertical lock-in.
- Ship preview/simulate/diff/rollback in the change pipeline.  
  Why: safety and trust are the core differentiators.
- Focus builder UX on non-technical users (visual editors + guard builders).  
  Why: adoption fails if users fall back to JSON editing.

## Don't (with why)

- Don't build a DSL compiler or SKILL.md language in v1.  
  Why: multi-year complexity with low early payoff.
- Don't let AI modify published config directly without review.  
  Why: breaks determinism, trust, and auditability.
- Don't introduce change pack tiers or a DSL-style change language.  
  Why: fragments tooling and recreates DSL maintenance burden.
- Don't make React Flow (or any graph) the source of truth.  
  Why: sync, drift, and conflict complexity outweighs early value.
- Don't add event sourcing, ClickHouse, or complex infra early.  
  Why: premature optimization slows validation of core value.
- Don't over-encode change packs with machine-only structure.  
  Why: the pack should stay human-first and agent-friendly.
