# Verdict Round 6: Change Packs Refined + Kernel Integration

**Date**: 2025-01-12  
**Status**: Refinement of change-pack spec with kernel integration  
**Purpose**: Address Round 5 gaps and provide implementation-ready spec

---

## Executive Summary

**Updated Feasibility: 76%**

Round 5's change-pack concept is sound, but the implementation details need refinement:
- JSON Patch is too brittle for AI generation
- 5 files per change is too much ceremony for simple edits
- Missing integration points with kernel validation and simulation

This round addresses those gaps with a **tiered pack system**, **declarative changes** (not patches), and **explicit kernel integration**.

---

## Issues with Round 5

| Issue | Impact | Resolution in Round 6 |
|-------|--------|----------------------|
| JSON Patch is path-brittle | Merge conflicts, silent failures | Replace with declarative change operations |
| 5 files per pack is ceremony-heavy | Discourages simple changes | Tiered packs: micro, standard, complex |
| No kernel integration specified | Gap between pack and runtime | Explicit ingestion → validation → simulation → publish pipeline |
| AI struggles with precise patches | Hallucinated paths, invalid patches | AI generates intent + constraints, kernel generates patch |
| No schema validation in flow | Invalid changes slip through | Kernel validates against canonical model before simulation |

---

## Refined Change Pack Spec

### Tiered Pack System

Not every change needs 5 files. Three tiers:

```
┌─────────────────────────────────────────────────────────────────┐
│                        PACK TIERS                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  MICRO PACK (single file)                                       │
│  ─────────────────────────                                      │
│  • Simple field/rule changes                                    │
│  • Single target config                                         │
│  • Auto-generated tests                                         │
│  • Format: YAML with embedded intent                            │
│                                                                 │
│  STANDARD PACK (2-3 files)                                      │
│  ───────────────────────────                                    │
│  • Workflow changes, multi-field updates                        │
│  • intent.md + changes.yaml + tests.yaml                        │
│  • Diagram auto-generated from changes                          │
│                                                                 │
│  COMPLEX PACK (full structure)                                  │
│  ─────────────────────────────                                  │
│  • Cross-entity changes, new entity types                       │
│  • Full 5-file structure                                        │
│  • Manual diagram review required                               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Micro Pack Format

For simple changes (80% of cases):

```yaml
# change-2025-01-12-001.yaml
specVersion: "1.0"
id: change-2025-01-12-001
tier: micro

meta:
  title: Make phone number required
  author: ai
  status: draft
  createdAt: "2025-01-12T10:00:00Z"

intent:
  userStory: >
    As a sales rep, I want phone number to be required on contacts
    so I always have a way to reach them.
  acceptance:
    - Phone field shows required indicator
    - Form submission fails without phone
    - Existing contacts without phone are not affected

target:
  configType: entity_schema
  configId: contact

# Declarative change - NOT a JSON patch
change:
  type: update_field
  field: phone
  set:
    required: true
    
# Tests auto-generated from change type + acceptance criteria
# Can be overridden if needed
tests:
  auto: true
```

### Standard Pack Format

For workflow and multi-field changes:

**intent.md:**
```markdown
---
specVersion: "1.0"
id: change-2025-01-12-002
tier: standard
title: High-value order approval workflow
author: ai
status: draft
createdAt: "2025-01-12T10:00:00Z"
---

## User Story

As a finance controller, I want orders over $50,000 to require CFO approval
before processing, so we maintain proper controls on large expenditures.

## Acceptance Criteria

1. Orders ≤ $50,000 follow existing workflow (no change)
2. Orders > $50,000 go to "pending_cfo" state after submission
3. Only users with CFO role can approve from "pending_cfo"
4. Approved orders continue to normal "processing" state
5. Rejection returns order to "draft" with reason required

## What This Does NOT Change

- No new fields on orders
- No changes to orders ≤ $50,000
- No changes to other entity types

## Targets

- workflow:order
```

**changes.yaml:**
```yaml
specVersion: "1.0"
packId: change-2025-01-12-002

# Declarative operations, not JSON patches
operations:
  
  # Add new state
  - type: add_state
    target: workflow:order
    state:
      name: pending_cfo
      type: intermediate
      displayName: Pending CFO Approval
      color: orange
      timeout:
        duration: P3D
        action:
          type: builtin
          ref: notify
          params:
            role: cfo
            message: "Order awaiting your approval for 3 days"

  # Modify existing transition
  - type: update_transition
    target: workflow:order
    transition:
      from: draft
      to: submitted
      event: submit
    addGuards:
      - type: field
        field: computed.totalValue
        operator: lte
        value: 50000

  # Add new transition for high-value
  - type: add_transition
    target: workflow:order
    transition:
      from: draft
      to: pending_cfo
      event: submit
      guards:
        - type: field
          field: computed.totalValue
          operator: gt
          value: 50000
      actions:
        - type: builtin
          ref: notify
          params:
            role: cfo
            message: "High-value order requires your approval"

  # Add CFO approval transition
  - type: add_transition
    target: workflow:order
    transition:
      from: pending_cfo
      to: processing
      event: approve
      allowedRoles: [cfo]
      
  # Add CFO rejection transition  
  - type: add_transition
    target: workflow:order
    transition:
      from: pending_cfo
      to: draft
      event: reject
      allowedRoles: [cfo]
      actions:
        - type: builtin
          ref: requireField
          params:
            field: rejectionReason
            message: "Please provide a reason for rejection"
```

**tests.yaml:**
```yaml
specVersion: "1.0"
packId: change-2025-01-12-002

cases:
  - name: "Low-value order follows normal flow"
    setup:
      entity: 
        type: order
        data: { totalValue: 30000 }
        workflowState: draft
    action:
      type: transition
      event: submit
    expect:
      success: true
      newState: submitted

  - name: "High-value order routes to CFO"
    setup:
      entity:
        type: order
        data: { totalValue: 75000 }
        workflowState: draft
    action:
      type: transition
      event: submit
    expect:
      success: true
      newState: pending_cfo

  - name: "Non-CFO cannot approve high-value"
    setup:
      entity:
        type: order
        data: { totalValue: 75000 }
        workflowState: pending_cfo
      actor:
        roles: [manager]
    action:
      type: transition
      event: approve
    expect:
      success: false
      errorsContain: ["not authorized"]

  - name: "CFO can approve high-value"
    setup:
      entity:
        type: order
        data: { totalValue: 75000 }
        workflowState: pending_cfo
      actor:
        roles: [cfo]
    action:
      type: transition
      event: approve
    expect:
      success: true
      newState: processing

  - name: "CFO rejection requires reason"
    setup:
      entity:
        type: order
        data: { totalValue: 75000 }
        workflowState: pending_cfo
      actor:
        roles: [cfo]
    action:
      type: transition
      event: reject
      payload: {}
    expect:
      success: false
      errorsContain: ["rejection reason"]
```

---

## Declarative Change Operations

Instead of JSON Patch, we use **declarative operations** that the kernel understands:

### Entity Schema Operations

```typescript
type EntitySchemaOperation =
  | { type: 'add_field'; target: string; field: FieldDefinition }
  | { type: 'update_field'; target: string; field: string; set: Partial<FieldDefinition> }
  | { type: 'remove_field'; target: string; field: string }
  | { type: 'add_relation'; target: string; relation: RelationDefinition }
  | { type: 'add_index'; target: string; index: IndexDefinition }
  | { type: 'add_computed'; target: string; computed: ComputedFieldDefinition };
```

### Workflow Operations

```typescript
type WorkflowOperation =
  | { type: 'add_state'; target: string; state: WorkflowState }
  | { type: 'update_state'; target: string; state: string; set: Partial<WorkflowState> }
  | { type: 'remove_state'; target: string; state: string }
  | { type: 'add_transition'; target: string; transition: WorkflowTransition }
  | { type: 'update_transition'; target: string; transition: TransitionIdentifier; 
      addGuards?: GuardCondition[]; removeGuards?: string[]; 
      addActions?: ActionRef[]; removeActions?: string[] }
  | { type: 'remove_transition'; target: string; transition: TransitionIdentifier };

interface TransitionIdentifier {
  from: string;
  to: string;
  event: string;
}
```

### UI Schema Operations

```typescript
type UISchemaOperation =
  | { type: 'add_form_section'; target: string; section: FormSection }
  | { type: 'update_form_field'; target: string; field: string; set: Partial<FormField> }
  | { type: 'add_list_column'; target: string; column: ListColumn }
  | { type: 'reorder_fields'; target: string; order: string[] };
```

### Guard/Rule Operations

```typescript
type GuardOperation =
  | { type: 'add_guard'; target: string; transitionId: TransitionIdentifier; guard: GuardCondition }
  | { type: 'remove_guard'; target: string; transitionId: TransitionIdentifier; guardIndex: number }
  | { type: 'update_guard'; target: string; transitionId: TransitionIdentifier; 
      guardIndex: number; set: Partial<GuardCondition> };
```

### Why Declarative > JSON Patch

| JSON Patch | Declarative Operations |
|------------|----------------------|
| `{"op": "add", "path": "/transitions/0/guards/-", ...}` | `{type: "add_guard", target: "workflow:order", transitionId: {...}, guard: {...}}` |
| Breaks if array order changes | Identifies transition by from/to/event, order-independent |
| AI must know exact path structure | AI specifies what to change, kernel figures out where |
| No semantic validation | Kernel validates operation is legal |
| Merge conflicts on concurrent edits | Operations can be rebased semantically |

---

## Kernel Integration Pipeline

### Pack Ingestion Flow

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   Receive    │────▶│    Parse     │────▶│   Validate   │────▶│   Preview    │
│    Pack      │     │    Pack      │     │   Against    │     │   Changes    │
│              │     │              │     │   Kernel     │     │              │
└──────────────┘     └──────────────┘     └──────────────┘     └──────────────┘
                                                │                      │
                                                ▼                      ▼
                                          Schema OK?            Generate diff
                                          Ops valid?            Generate diagram
                                          Refs exist?           Show before/after
                                                │
                                                ▼
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   Publish    │◀────│   Approve    │◀────│   Simulate   │
│   (Active)   │     │   (Human)    │     │   (Tests)    │
│              │     │              │     │              │
└──────────────┘     └──────────────┘     └──────────────┘
```

### Implementation

```typescript
// packages/change-packs/processor.ts

interface PackProcessResult {
  valid: boolean;
  errors: PackError[];
  warnings: PackWarning[];
  preview?: PackPreview;
}

interface PackPreview {
  configDiffs: ConfigDiff[];      // Before/after for each target
  generatedTests: TestCase[];     // Auto-generated tests
  diagram?: string;               // Mermaid diagram (for workflows)
  impactSummary: ImpactSummary;
}

interface ImpactSummary {
  affectedEntityTypes: string[];
  affectedWorkflows: string[];
  existingDataImpact: 'none' | 'validation_change' | 'migration_required';
  estimatedEntitiesAffected: number;
}

export class PackProcessor {
  constructor(
    private kernel: Kernel,
    private configStore: ConfigStore,
  ) {}

  async process(pack: IntentPack): Promise<PackProcessResult> {
    const errors: PackError[] = [];
    const warnings: PackWarning[] = [];

    // 1. Parse and validate pack structure
    const parseResult = this.parsePack(pack);
    if (!parseResult.valid) {
      return { valid: false, errors: parseResult.errors, warnings: [] };
    }

    // 2. Load current config for all targets
    const currentConfigs = await this.loadTargetConfigs(pack.targets);

    // 3. Validate each operation against kernel constraints
    for (const op of pack.operations) {
      const validation = this.kernel.validateOperation(op, currentConfigs);
      if (!validation.valid) {
        errors.push({
          operation: op,
          message: validation.message,
          code: validation.code,
        });
      }
      if (validation.warnings) {
        warnings.push(...validation.warnings);
      }
    }

    if (errors.length > 0) {
      return { valid: false, errors, warnings };
    }

    // 4. Apply operations to get new config (in memory, not persisted)
    const newConfigs = this.applyOperations(currentConfigs, pack.operations);

    // 5. Generate preview
    const preview = await this.generatePreview(currentConfigs, newConfigs, pack);

    // 6. Run tests (if provided) or generate auto-tests
    const testResults = await this.runTests(pack, newConfigs);
    if (testResults.failures.length > 0) {
      errors.push(...testResults.failures.map(f => ({
        operation: null,
        message: `Test failed: ${f.name} - ${f.reason}`,
        code: 'TEST_FAILURE',
      })));
    }

    return {
      valid: errors.length === 0,
      errors,
      warnings,
      preview,
    };
  }

  async publish(pack: IntentPack, approver: User): Promise<PublishResult> {
    // 1. Re-validate (config may have changed since preview)
    const result = await this.process(pack);
    if (!result.valid) {
      return { success: false, errors: result.errors };
    }

    // 2. Create config versions for each target
    const versions: ConfigVersion[] = [];
    for (const target of pack.targets) {
      const version = await this.configStore.createVersion({
        configType: target.configType,
        configId: target.configId,
        status: 'published',
        configData: result.preview!.configDiffs.find(
          d => d.target === `${target.configType}:${target.configId}`
        )!.after,
        publishedBy: approver.id,
        packId: pack.id,
      });
      versions.push(version);
    }

    // 3. Update active config
    for (const version of versions) {
      await this.configStore.setActive(version);
    }

    // 4. Log pack application
    await this.auditLog.record({
      type: 'pack_published',
      packId: pack.id,
      approver: approver.id,
      versions: versions.map(v => v.id),
    });

    return { success: true, versions };
  }
}
```

### Kernel Validation

```typescript
// packages/kernel/validator.ts

export class KernelValidator {
  validateOperation(
    op: Operation,
    currentConfigs: Map<string, Config>
  ): ValidationResult {
    
    switch (op.type) {
      case 'add_field':
        return this.validateAddField(op, currentConfigs);
      case 'add_state':
        return this.validateAddState(op, currentConfigs);
      case 'add_transition':
        return this.validateAddTransition(op, currentConfigs);
      case 'add_guard':
        return this.validateAddGuard(op, currentConfigs);
      // ... etc
    }
  }

  private validateAddField(
    op: AddFieldOperation,
    configs: Map<string, Config>
  ): ValidationResult {
    const schema = configs.get(op.target);
    if (!schema) {
      return { valid: false, message: `Target schema not found: ${op.target}`, code: 'TARGET_NOT_FOUND' };
    }

    // Check field doesn't already exist
    if (schema.fields.some(f => f.name === op.field.name)) {
      return { valid: false, message: `Field already exists: ${op.field.name}`, code: 'FIELD_EXISTS' };
    }

    // Validate field type is allowed
    if (!ALLOWED_FIELD_TYPES.includes(op.field.type)) {
      return { valid: false, message: `Invalid field type: ${op.field.type}`, code: 'INVALID_TYPE' };
    }

    // Validate constraints are within kernel limits
    if (op.field.type === 'string' && op.field.maxLength && op.field.maxLength > 10000) {
      return { 
        valid: true, 
        warnings: [{ message: 'Large maxLength may impact performance', code: 'PERF_WARNING' }] 
      };
    }

    return { valid: true };
  }

  private validateAddGuard(
    op: AddGuardOperation,
    configs: Map<string, Config>
  ): ValidationResult {
    const workflow = configs.get(op.target);
    if (!workflow) {
      return { valid: false, message: `Target workflow not found: ${op.target}`, code: 'TARGET_NOT_FOUND' };
    }

    // Find the transition
    const transition = workflow.transitions.find(
      t => t.from === op.transitionId.from && 
           t.to === op.transitionId.to && 
           t.event === op.transitionId.event
    );
    if (!transition) {
      return { valid: false, message: `Transition not found: ${JSON.stringify(op.transitionId)}`, code: 'TRANSITION_NOT_FOUND' };
    }

    // Validate guard uses allowed operators
    if (!ALLOWED_GUARD_OPERATORS.includes(op.guard.operator)) {
      return { valid: false, message: `Invalid guard operator: ${op.guard.operator}`, code: 'INVALID_OPERATOR' };
    }

    // Validate field reference exists (for type: 'field')
    if (op.guard.type === 'field') {
      const fieldExists = this.resolveFieldRef(op.guard.field, configs);
      if (!fieldExists) {
        return { valid: false, message: `Guard references unknown field: ${op.guard.field}`, code: 'UNKNOWN_FIELD' };
      }
    }

    // Validate function reference exists (for type: 'function')
    if (op.guard.type === 'function') {
      const funcExists = this.functionRegistry.has(op.guard.functionRef);
      if (!funcExists) {
        return { valid: false, message: `Guard references unknown function: ${op.guard.functionRef}`, code: 'UNKNOWN_FUNCTION' };
      }
    }

    return { valid: true };
  }
}
```

---

## AI Intent Generation

The key insight: **AI generates intent + constraints, kernel generates operations.**

### AI's Responsibility

```typescript
// What AI produces from natural language
interface AIGeneratedIntent {
  // Mandatory framing
  userStory: string;
  acceptanceCriteria: string[];
  whatThisDoesNot: string[];

  // Constraints AI extracted from conversation
  constraints: IntentConstraint[];
  
  // AI's suggested tier
  suggestedTier: 'micro' | 'standard' | 'complex';
}

interface IntentConstraint {
  type: 'require_field' | 'add_validation' | 'add_workflow_state' | 
        'add_guard' | 'add_transition' | 'modify_transition';
  target: string;
  description: string;
  parameters: Record<string, unknown>;
}
```

### Kernel's Responsibility

```typescript
// Kernel converts constraints to operations
class IntentCompiler {
  compile(intent: AIGeneratedIntent): IntentPack {
    const operations: Operation[] = [];

    for (const constraint of intent.constraints) {
      const ops = this.constraintToOperations(constraint);
      operations.push(...ops);
    }

    return {
      id: generatePackId(),
      tier: intent.suggestedTier,
      meta: {
        title: this.generateTitle(intent),
        userStory: intent.userStory,
        acceptance: intent.acceptanceCriteria,
      },
      operations,
      tests: this.generateTests(intent, operations),
    };
  }

  private constraintToOperations(constraint: IntentConstraint): Operation[] {
    switch (constraint.type) {
      case 'require_field':
        return [{
          type: 'update_field',
          target: constraint.target,
          field: constraint.parameters.field as string,
          set: { required: true },
        }];

      case 'add_guard':
        return [{
          type: 'add_guard',
          target: constraint.target,
          transitionId: constraint.parameters.transition as TransitionIdentifier,
          guard: constraint.parameters.guard as GuardCondition,
        }];

      case 'add_workflow_state':
        // May produce multiple operations (state + transitions)
        const state = constraint.parameters.state as WorkflowState;
        const transitions = constraint.parameters.transitions as WorkflowTransition[];
        return [
          { type: 'add_state', target: constraint.target, state },
          ...transitions.map(t => ({ type: 'add_transition', target: constraint.target, transition: t })),
        ];

      // ... etc
    }
  }
}
```

### Why This Split?

| AI | Kernel |
|----|--------|
| Understands natural language | Understands schema structure |
| Can hallucinate paths | Knows exact config layout |
| Good at "what user wants" | Good at "how to achieve it" |
| Fuzzy constraints | Precise operations |

By splitting responsibilities:
- AI doesn't need to know JSON structure
- Kernel ensures operations are valid
- AI errors are caught before they become patches
- Same constraint can map to different operations as kernel evolves

---

## Auto-Generated Tests

For micro packs and simple operations, tests can be auto-generated:

```typescript
class TestGenerator {
  generate(operations: Operation[], intent: AIGeneratedIntent): TestCase[] {
    const tests: TestCase[] = [];

    for (const op of operations) {
      tests.push(...this.generateForOperation(op));
    }

    // Add tests from acceptance criteria
    for (const criterion of intent.acceptanceCriteria) {
      const test = this.generateFromCriterion(criterion, operations);
      if (test) tests.push(test);
    }

    return tests;
  }

  private generateForOperation(op: Operation): TestCase[] {
    switch (op.type) {
      case 'update_field':
        if (op.set.required === true) {
          return [
            {
              name: `${op.field} is required - missing`,
              setup: { entity: { type: this.getEntityType(op.target), data: {} } },
              action: { type: 'validate' },
              expect: { success: false, errorsContain: [op.field, 'required'] },
            },
            {
              name: `${op.field} is required - present`,
              setup: { entity: { type: this.getEntityType(op.target), data: { [op.field]: 'test' } } },
              action: { type: 'validate' },
              expect: { success: true },
            },
          ];
        }
        break;

      case 'add_guard':
        return this.generateGuardTests(op);

      case 'add_transition':
        return this.generateTransitionTests(op);
    }

    return [];
  }

  private generateGuardTests(op: AddGuardOperation): TestCase[] {
    const guard = op.guard;
    
    if (guard.type === 'field' && guard.operator === 'gt') {
      const threshold = guard.value as number;
      return [
        {
          name: `Guard blocks when ${guard.field} > ${threshold}`,
          setup: {
            entity: { data: { [guard.field.replace('computed.', '')]: threshold + 1 } },
            workflowState: op.transitionId.from,
          },
          action: { type: 'transition', event: op.transitionId.event },
          expect: { success: false },
        },
        {
          name: `Guard allows when ${guard.field} <= ${threshold}`,
          setup: {
            entity: { data: { [guard.field.replace('computed.', '')]: threshold } },
            workflowState: op.transitionId.from,
          },
          action: { type: 'transition', event: op.transitionId.event },
          expect: { success: true, newState: op.transitionId.to },
        },
      ];
    }

    // ... other guard types
    return [];
  }
}
```

---

## Diagram Generation

Mermaid diagrams are auto-generated from workflow config:

```typescript
class DiagramGenerator {
  generateWorkflowDiagram(
    before: WorkflowDefinition,
    after: WorkflowDefinition,
    highlightChanges: boolean = true
  ): string {
    const lines: string[] = ['stateDiagram-v2'];

    // Find added states and transitions
    const addedStates = new Set(
      after.states
        .filter(s => !before.states.some(bs => bs.name === s.name))
        .map(s => s.name)
    );
    const addedTransitions = after.transitions.filter(
      t => !before.transitions.some(bt => 
        bt.from === t.from && bt.to === t.to && bt.event === t.event
      )
    );

    // Initial state
    lines.push(`  [*] --> ${after.initialState}`);

    // States (highlight new ones)
    for (const state of after.states) {
      if (state.type === 'terminal') {
        lines.push(`  ${state.name} --> [*]`);
      }
      if (highlightChanges && addedStates.has(state.name)) {
        lines.push(`  ${state.name}:::newState`);
      }
    }

    // Transitions
    for (const t of after.transitions) {
      const guardText = t.guards?.length 
        ? ` [${t.guards.map(g => this.guardToText(g)).join(', ')}]`
        : '';
      const roleText = t.allowedRoles?.length 
        ? ` {${t.allowedRoles.join(',')}}`
        : '';
      
      const isNew = addedTransitions.some(
        at => at.from === t.from && at.to === t.to && at.event === t.event
      );
      
      if (highlightChanges && isNew) {
        lines.push(`  ${t.from} --> ${t.to}: ${t.event}${guardText}${roleText}`);
        lines.push(`  linkStyle ${this.getLinkIndex(after, t)} stroke:green,stroke-width:2px`);
      } else {
        lines.push(`  ${t.from} --> ${t.to}: ${t.event}${guardText}${roleText}`);
      }
    }

    // Styles
    if (highlightChanges) {
      lines.push('  classDef newState fill:#90EE90,stroke:#228B22');
    }

    return lines.join('\n');
  }

  private guardToText(guard: GuardCondition): string {
    if (guard.type === 'field') {
      return `${guard.field}${guard.operator}${guard.value}`;
    }
    if (guard.type === 'role') {
      return `role=${guard.role}`;
    }
    if (guard.type === 'function') {
      return `fn:${guard.functionRef}`;
    }
    return '?';
  }
}
```

---

## Updated Risk Assessment

### Risks Reduced by Round 6

| Risk | Mitigation |
|------|------------|
| JSON Patch brittleness | Replaced with declarative operations |
| Ceremony overhead | Tiered packs (micro for simple changes) |
| AI path hallucination | AI generates constraints, kernel generates operations |
| Missing validation | Kernel validates before simulation |
| Gap to runtime | Explicit ingestion → validation → simulation → publish pipeline |

### Remaining Risks

| Risk | Level | Mitigation Plan |
|------|-------|-----------------|
| Authoring UX | HIGH | Invest heavily in visual editors |
| AI constraint accuracy | MEDIUM | Human reviews all packs before publish |
| Operation coverage | MEDIUM | Start with common ops, add more based on usage |
| Test coverage | LOW | Auto-generation covers 80%, manual for complex |

---

## Implementation Additions (from Round 4)

Add to the 24-week plan:

**Week 5-6 (Config System):**
```
□ Add: Declarative operation types
□ Add: Operation validators
□ Add: Change pack parser
□ Add: Pack processor pipeline
```

**Week 11-12 (Action System):**
```
□ Add: Intent compiler (constraint → operations)
□ Add: Test generator
□ Add: Diagram generator
```

**Week 21-22 (AI Analysis):**
```
□ Add: AI intent generator
□ Add: Constraint extraction from conversation
□ Add: Pack suggestion UI
```

---

## Final Verdict

**Updated Feasibility: 76%** (up from 74%)

The refined change-pack spec addresses Round 5's gaps:

1. **Declarative operations** instead of brittle JSON Patches
2. **Tiered packs** to reduce ceremony for simple changes
3. **Split responsibility**: AI generates constraints, kernel generates operations
4. **Explicit pipeline**: ingestion → validation → simulation → publish
5. **Auto-generated tests and diagrams** reduce manual work

The architecture is now:
- **Coherent**: All pieces connect clearly
- **AI-friendly**: AI works at constraint level, not patch level
- **Reviewable**: Every change has intent, diff, tests, diagram
- **Deterministic**: Kernel validates all operations before runtime

**Proceed with Round 4 architecture + Round 6 change-pack spec.**

---

## Appendix: Complete Pack Examples

### Micro Pack: Make Field Required

```yaml
specVersion: "1.0"
id: change-2025-01-12-micro-001
tier: micro

meta:
  title: Make email required on contacts
  author: user
  status: draft

intent:
  userStory: As a sales rep, I need email on all contacts.
  acceptance:
    - Contact form requires email
    - Existing contacts unaffected

target:
  configType: entity_schema
  configId: contact

change:
  type: update_field
  field: email
  set:
    required: true

tests:
  auto: true
```

### Standard Pack: Add Approval Workflow

See examples in "Standard Pack Format" section above.

### Complex Pack: New Entity Type

```
change-packs/
  change-2025-01-12-complex-001/
    manifest.json        # Pack metadata
    intent.md            # Full user story + acceptance criteria
    entity-schema.yaml   # New entity definition
    workflow.yaml        # New workflow definition
    ui-schema.yaml       # Form/list configuration
    changes.yaml         # Operations to create all configs
    tests.yaml           # Comprehensive test suite
    diagram.mmd          # Workflow diagram
```

**manifest.json:**
```json
{
  "specVersion": "1.0",
  "id": "change-2025-01-12-complex-001",
  "tier": "complex",
  "title": "Add Purchase Request entity",
  "author": "ai",
  "status": "draft",
  "createdAt": "2025-01-12T10:00:00Z",
  "files": [
    "intent.md",
    "entity-schema.yaml",
    "workflow.yaml", 
    "ui-schema.yaml",
    "changes.yaml",
    "tests.yaml",
    "diagram.mmd"
  ],
  "targets": [
    { "configType": "entity_schema", "configId": "purchase_request" },
    { "configType": "workflow", "configId": "purchase_request" },
    { "configType": "ui_schema", "configId": "purchase_request" }
  ]
}
```

This structure supports adding entirely new entity types while maintaining the same review/test/publish flow.
