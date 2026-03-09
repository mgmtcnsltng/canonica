# TODO

## Sequence (start here)

1) Define the generic demo app scope (request → approval → fulfillment).  
   Why: anchors the runtime engine and UX in a real flow.
2) Build the constrained runtime engine (entity schema, schema-driven UI, guarded workflows, audit/versioning, RBAC).  
   Why: proves deterministic runtime and generic behavior without a DSL.
3) Implement Change Pack v2.0 flow (intent.md + change.md + preview/diff/simulate/publish).  
   Why: human-first change process that AI can draft and humans can trust.
4) Build the non-technical builder UX (visual schema + workflow + guard editors).  
   Why: adoption fails if users fall back to JSON editing.
5) Add Plan vs Accept AI modes with explicit risk gating.  
   Why: enables speed without sacrificing safety.
6) Polish safety UX (preview/simulate/diff/rollback) and operator AI.  
   Why: safety and trust are the core differentiators.

## UI Component Guidelines

- Use shadcn/ui components; map field types to appropriate inputs.
- Examples: enum → `Select`, long text → `Textarea`, boolean → `Checkbox`,
  date → `DatePicker`/`Input[type=date]`, money → `Input` with currency formatting.

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
- Don't move business logic into Redis or caches.  
  Why: source-of-truth must stay deterministic and auditable.
- Don't over-encode change packs with machine-only structure.  
  Why: the pack should stay human-first and agent-friendly.
