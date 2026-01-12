# Verdict Round 4: The Definitive Implementation Path

**Date**: 2025-01-12  
**Status**: Final architecture decision and implementation specification  
**Purpose**: Reconcile Rounds 1-3 into a buildable system

---

## Executive Summary

**Final Feasibility: 72%**

After four rounds of analysis, the viable path is clear:

> **Build a constrained kernel with guarded workflows, plugin extensibility, and AI as an observing advisor—not an author.**

This document provides the definitive architecture, the exact boundaries of the canonical model, and a week-by-week implementation plan ready for Claude Code.

---

## The Journey So Far

| Round | Focus | Key Insight | Feasibility |
|-------|-------|-------------|-------------|
| 1 | Original vision | DSL compiler is 12-24 months; generic + simple is unprecedented | 60% |
| 2 | Pragmatic pivot | AI should observe and suggest, not author; skip event sourcing | 70% (template-first) |
| 3 | Generic-first validation | Constrained kernel + escape hatches is the only viable generic path | 55-70% |
| **4** | **Reconciliation** | **Merge the best of each: kernel + extensibility + advisory AI** | **72%** |

---

## The Reconciled Architecture

### Core Principles (Non-Negotiable)

1. **Determinism**: Runtime behavior is 100% predictable from published configuration
2. **Small Kernel**: The canonical model expresses only what it can express safely
3. **Escape Hatches**: Everything else goes through plugins, functions, webhooks
4. **Governance**: All changes flow through draft → validate → simulate → publish
5. **AI as Advisor**: AI observes, suggests, explains—never auto-applies

### What the Kernel Covers

```
┌─────────────────────────────────────────────────────────────────┐
│                     THE KERNEL (Canonical Model)                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐ │
│  │  Entity Schema  │  │    Workflow     │  │   Guard Rules   │ │
│  │                 │  │                 │  │                 │ │
│  │ • Field types   │  │ • States (FSM)  │  │ • Conditions    │ │
│  │ • Validation    │  │ • Transitions   │  │ • Simple ops    │ │
│  │ • Relations     │  │ • Permissions   │  │ • Built-in acts │ │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘ │
│                                                                 │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐ │
│  │   UI Schema     │  │  Audit + RBAC   │  │   Versioning    │ │
│  │                 │  │                 │  │                 │ │
│  │ • Form layout   │  │ • Action log    │  │ • Draft/Publish │ │
│  │ • List config   │  │ • Role perms    │  │ • Rollback      │ │
│  │ • Widget hints  │  │ • RLS policies  │  │ • History       │ │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘ │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### What the Kernel Does NOT Cover (Escape Hatches)

```
┌─────────────────────────────────────────────────────────────────┐
│                    ESCAPE HATCHES (Extensions)                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐ │
│  │ Plugin Actions  │  │   Functions     │  │    Webhooks     │ │
│  │                 │  │                 │  │                 │ │
│  │ • Custom logic  │  │ • Computed vals │  │ • External APIs │ │
│  │ • Integrations  │  │ • Complex valid │  │ • Notifications │ │
│  │ • Side effects  │  │ • Derivations   │  │ • Sync triggers │ │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘ │
│                                                                 │
│  Called BY the kernel, but logic lives OUTSIDE the kernel      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### The AI Layer (Observer + Advisor)

```
┌─────────────────────────────────────────────────────────────────┐
│                    AI LAYER (Advisory Only)                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  OBSERVES                          SUGGESTS                     │
│  ─────────                         ────────                     │
│  • All entity operations           • New guard rules            │
│  • Workflow transitions            • Workflow optimizations     │
│  • User behavior patterns          • Schema improvements        │
│  • Anomalies vs baselines          • Missing validations        │
│                                                                 │
│  NEVER                                                          │
│  ─────                                                          │
│  • Auto-applies changes                                         │
│  • Executes at runtime                                          │
│  • Modifies published config                                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## The Canonical Model Specification

### Entity Schema

```typescript
interface EntitySchema {
  id: string;
  name: string;                    // "stocktake", "order", "contact"
  displayName: string;             // "Stock Take", "Sales Order"
  
  fields: FieldDefinition[];
  
  // Relations to other entities
  relations?: RelationDefinition[];
  
  // Computed fields (via functions, not expressions)
  computed?: ComputedFieldDefinition[];
  
  // Indexes for query performance
  indexes?: IndexDefinition[];
}

interface FieldDefinition {
  name: string;
  type: 'string' | 'number' | 'boolean' | 'date' | 'datetime' | 'enum' | 'json';
  
  // Display
  label?: string;
  description?: string;
  
  // Constraints (simple, no expressions)
  required?: boolean;
  unique?: boolean;
  defaultValue?: unknown;
  
  // Type-specific
  enumValues?: string[];           // for type: 'enum'
  min?: number;                    // for type: 'number'
  max?: number;                    // for type: 'number'
  minLength?: number;              // for type: 'string'
  maxLength?: number;              // for type: 'string'
  pattern?: string;                // regex for type: 'string'
  format?: 'email' | 'url' | 'phone' | 'currency' | 'percentage';
}

interface RelationDefinition {
  name: string;
  type: 'belongsTo' | 'hasMany' | 'hasOne';
  target: string;                  // target entity name
  foreignKey: string;
  cascadeDelete?: boolean;
}

interface ComputedFieldDefinition {
  name: string;
  type: FieldDefinition['type'];
  // Computed via registered function, NOT inline expression
  functionRef: string;             // "calculateTotalValue", "getVariancePercent"
}
```

### Workflow Definition (Guarded FSM)

```typescript
interface WorkflowDefinition {
  id: string;
  entityType: string;
  name: string;
  
  initialState: string;
  states: WorkflowState[];
  transitions: WorkflowTransition[];
}

interface WorkflowState {
  name: string;
  type: 'initial' | 'intermediate' | 'terminal';
  
  // UI hints
  displayName?: string;
  color?: string;
  
  // Timeout (optional)
  timeout?: {
    duration: string;              // ISO 8601 duration: "P3D", "PT4H"
    action: ActionRef;
  };
  
  // Entry/exit hooks (via plugin actions)
  onEnter?: ActionRef[];
  onExit?: ActionRef[];
}

interface WorkflowTransition {
  from: string;                    // state name or '*' for any
  to: string;
  
  // The event that triggers this transition
  event: string;                   // "submit", "approve", "cancel"
  
  // Who can trigger (role-based)
  allowedRoles?: string[];
  
  // Guards: conditions that must pass (simple operators only)
  guards?: GuardCondition[];
  
  // Actions to execute on transition
  actions?: ActionRef[];
}

// Guards use simple operators, NOT expressions
interface GuardCondition {
  type: 'field' | 'role' | 'function';
  
  // For type: 'field'
  field?: string;
  operator?: 'eq' | 'ne' | 'gt' | 'lt' | 'gte' | 'lte' | 'in' | 'notIn' | 'exists' | 'notExists';
  value?: unknown;
  
  // For type: 'role'
  role?: string;
  
  // For type: 'function' (escape hatch for complex logic)
  functionRef?: string;
}

// Actions reference registered handlers
interface ActionRef {
  type: 'builtin' | 'plugin' | 'webhook';
  ref: string;                     // "notify", "sendEmail", "myPlugin.doThing"
  params?: Record<string, unknown>;
}
```

### Built-in Actions (Kernel)

```typescript
// These are the ONLY actions built into the kernel
const BUILTIN_ACTIONS = {
  // Notifications
  'notify': { params: ['role', 'message'] },
  'notifyUser': { params: ['userId', 'message'] },
  
  // Field operations
  'setField': { params: ['field', 'value'] },
  'requireField': { params: ['field', 'message'] },
  'clearField': { params: ['field'] },
  
  // Workflow
  'scheduleTimeout': { params: ['duration', 'action'] },
  'cancelTimeout': { params: ['timeoutId'] },
  
  // Validation
  'blockWithError': { params: ['message'] },
  'warnWithMessage': { params: ['message'] },
  
  // Audit
  'logAuditEvent': { params: ['eventType', 'details'] },
};

// Everything else must be a plugin action or webhook
```

### UI Schema

```typescript
interface UISchema {
  entityType: string;
  
  // Form configuration
  form: {
    sections: FormSection[];
  };
  
  // List/table configuration
  list: {
    columns: ListColumn[];
    defaultSort?: { field: string; direction: 'asc' | 'desc' };
    filters?: FilterDefinition[];
    actions?: ListAction[];
  };
  
  // Detail view configuration
  detail: {
    layout: 'single' | 'tabbed' | 'sidebar';
    sections: DetailSection[];
  };
}

interface FormSection {
  id: string;
  title?: string;
  columns: 1 | 2 | 3;
  fields: FormField[];
  condition?: GuardCondition;      // Show/hide section
}

interface FormField {
  field: string;                   // References EntitySchema field
  widget?: 'text' | 'textarea' | 'number' | 'select' | 'radio' | 'checkbox' | 
           'date' | 'datetime' | 'currency' | 'file' | 'richtext' | 'relation';
  colSpan?: 1 | 2 | 3;
  placeholder?: string;
  helpText?: string;
  readOnly?: boolean | GuardCondition;
  hidden?: boolean | GuardCondition;
}

interface ListColumn {
  field: string;
  label?: string;
  width?: number | 'auto';
  sortable?: boolean;
  format?: 'default' | 'currency' | 'date' | 'datetime' | 'badge';
}
```

---

## The Extension System

### Plugin Actions

```typescript
// Plugin registration (server-side)
interface PluginDefinition {
  id: string;
  name: string;
  version: string;
  
  actions: PluginAction[];
}

interface PluginAction {
  name: string;                    // "sendSlackMessage"
  description: string;
  
  // Input schema (validated before execution)
  inputSchema: {
    type: 'object';
    properties: Record<string, unknown>;
    required?: string[];
  };
  
  // The handler (sandboxed execution)
  handler: (input: unknown, context: ExecutionContext) => Promise<ActionResult>;
}

interface ExecutionContext {
  entity: Record<string, unknown>;
  actor: { id: string; roles: string[] };
  tenant: { id: string; settings: Record<string, unknown> };
  
  // Limited capabilities
  capabilities: {
    readEntity: (type: string, id: string) => Promise<unknown>;
    queryEntities: (type: string, filter: unknown) => Promise<unknown[]>;
    // NO write capabilities - plugins can only read and return results
    // Writes happen through the normal command flow
  };
}

interface ActionResult {
  success: boolean;
  data?: unknown;
  error?: string;
  // Plugins can SUGGEST field updates, but don't apply them directly
  suggestedUpdates?: Record<string, unknown>;
}
```

### Computed Functions

```typescript
// Function registration for computed fields
interface ComputedFunction {
  name: string;                    // "calculateTotalValue"
  description: string;
  
  // Which entity types this function applies to
  entityTypes: string[];
  
  // Dependencies (for cache invalidation)
  dependsOn: string[];             // ["items.quantity", "items.unitPrice"]
  
  // The computation (pure function, no side effects)
  compute: (entity: Record<string, unknown>, context: ReadOnlyContext) => unknown;
}

// Example
const calculateTotalValue: ComputedFunction = {
  name: 'calculateTotalValue',
  description: 'Sum of all item values',
  entityTypes: ['stocktake'],
  dependsOn: ['items'],
  compute: (entity, ctx) => {
    const items = entity.items as Array<{ quantity: number; unitPrice: number }>;
    return items.reduce((sum, item) => sum + item.quantity * item.unitPrice, 0);
  },
};
```

### Webhooks

```typescript
interface WebhookDefinition {
  id: string;
  name: string;
  
  // When to fire
  triggers: WebhookTrigger[];
  
  // Where to send
  url: string;
  method: 'POST' | 'PUT';
  headers?: Record<string, string>;
  
  // Retry policy
  retry: {
    maxAttempts: number;
    backoffMs: number;
  };
  
  // Optional response handling
  onSuccess?: ActionRef;
  onFailure?: ActionRef;
}

interface WebhookTrigger {
  event: string;                   // "entity.created", "workflow.transitioned"
  entityType?: string;
  conditions?: GuardCondition[];
}
```

---

## Governance System

### Change Lifecycle

```
┌─────────┐    ┌──────────┐    ┌──────────┐    ┌─────────┐    ┌───────────┐
│  Draft  │───▶│ Validate │───▶│ Simulate │───▶│ Preview │───▶│  Publish  │
└─────────┘    └──────────┘    └──────────┘    └─────────┘    └───────────┘
     │              │               │               │               │
     │              ▼               ▼               ▼               │
     │         Syntax OK?      Test cases      UI preview          │
     │         Schema OK?      pass?           looks right?        │
     │                                                             │
     └─────────────────────────────────────────────────────────────┘
                            (can revert at any stage)
```

### Version Storage

```sql
-- Configuration versions
CREATE TABLE config_versions (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id       UUID NOT NULL,
    
    -- What changed
    config_type     VARCHAR(50) NOT NULL,  -- 'entity_schema', 'workflow', 'ui_schema', 'rule'
    config_id       VARCHAR(100) NOT NULL,
    
    -- Version info
    version         INTEGER NOT NULL,
    status          VARCHAR(20) NOT NULL,  -- 'draft', 'validated', 'published', 'archived'
    
    -- The actual configuration
    config_data     JSONB NOT NULL,
    
    -- Diff from previous
    diff_from       UUID REFERENCES config_versions(id),
    diff_summary    JSONB,
    
    -- Audit
    created_by      UUID NOT NULL,
    created_at      TIMESTAMPTZ DEFAULT NOW(),
    published_by    UUID,
    published_at    TIMESTAMPTZ,
    
    -- Simulation results (if run)
    simulation_result JSONB,
    
    UNIQUE(tenant_id, config_type, config_id, version)
);

-- Currently active configuration (denormalized for fast reads)
CREATE TABLE active_config (
    tenant_id       UUID NOT NULL,
    config_type     VARCHAR(50) NOT NULL,
    config_id       VARCHAR(100) NOT NULL,
    version_id      UUID REFERENCES config_versions(id),
    config_data     JSONB NOT NULL,
    
    PRIMARY KEY (tenant_id, config_type, config_id)
);
```

### Simulation Engine

```typescript
interface SimulationRequest {
  configVersionId: string;
  testCases: TestCase[];
}

interface TestCase {
  name: string;
  // Starting state
  entity: Record<string, unknown>;
  workflowState?: string;
  // Action to simulate
  action: {
    type: 'command' | 'transition';
    payload: unknown;
  };
  // Expected outcome
  expected: {
    success?: boolean;
    newState?: string;
    fieldsSet?: Record<string, unknown>;
    errorsContain?: string[];
  };
}

interface SimulationResult {
  passed: number;
  failed: number;
  results: TestCaseResult[];
}

async function simulateConfig(request: SimulationRequest): Promise<SimulationResult> {
  const config = await loadConfigVersion(request.configVersionId);
  const results: TestCaseResult[] = [];
  
  for (const testCase of request.testCases) {
    // Create isolated execution context
    const ctx = createSimulationContext(config, testCase.entity);
    
    // Execute action against the NEW config (not published)
    const outcome = await executeInSimulation(ctx, testCase.action);
    
    // Compare to expected
    const passed = compareOutcome(outcome, testCase.expected);
    
    results.push({
      testCase: testCase.name,
      passed,
      actual: outcome,
      expected: testCase.expected,
      diff: passed ? null : computeDiff(outcome, testCase.expected),
    });
  }
  
  return {
    passed: results.filter(r => r.passed).length,
    failed: results.filter(r => !r.passed).length,
    results,
  };
}
```

---

## AI Advisory System

### What AI Observes

```typescript
// Activity types the AI analyzes
type ObservableActivity =
  | { type: 'entity_created'; entityType: string; data: unknown }
  | { type: 'entity_updated'; entityType: string; before: unknown; after: unknown }
  | { type: 'workflow_transitioned'; entityType: string; from: string; to: string; actor: string }
  | { type: 'guard_failed'; transition: string; reason: string }
  | { type: 'action_executed'; action: string; result: 'success' | 'failure' }
  | { type: 'user_behavior'; userId: string; action: string; duration: number };

// Stored for analysis
CREATE TABLE activity_observations (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id       UUID NOT NULL,
    activity_type   VARCHAR(50) NOT NULL,
    entity_type     VARCHAR(100),
    activity_data   JSONB NOT NULL,
    created_at      TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_observations_tenant_time ON activity_observations(tenant_id, created_at DESC);
CREATE INDEX idx_observations_type ON activity_observations(activity_type);
```

### What AI Suggests

```typescript
type AISuggestion =
  | {
      type: 'new_guard';
      description: string;
      evidence: PatternEvidence;
      suggestedConfig: GuardCondition;
    }
  | {
      type: 'workflow_optimization';
      description: string;
      evidence: TimingEvidence;
      suggestedChange: Partial<WorkflowDefinition>;
    }
  | {
      type: 'missing_validation';
      description: string;
      evidence: ErrorEvidence;
      suggestedField: Partial<FieldDefinition>;
    }
  | {
      type: 'anomaly_alert';
      description: string;
      evidence: AnomalyEvidence;
      suggestedActions: string[];
    };

interface PatternEvidence {
  patternDescription: string;
  occurrences: number;
  confidence: number;
  examples: Array<{
    timestamp: Date;
    entityId: string;
    details: unknown;
  }>;
}
```

### Suggestion Review UI

```typescript
// Suggestions are presented with full evidence
interface SuggestionPresentation {
  id: string;
  type: AISuggestion['type'];
  
  // Human-readable
  title: string;
  description: string;
  
  // Why AI suggests this
  reasoning: string;
  evidence: {
    summary: string;
    dataPoints: number;
    examples: unknown[];
    confidence: number;
  };
  
  // What would change
  preview: {
    before: unknown;
    after: unknown;
    diff: Diff[];
  };
  
  // Impact assessment
  impact: {
    affectedEntities: number;
    riskLevel: 'low' | 'medium' | 'high';
    reversible: boolean;
  };
  
  // Actions
  actions: {
    accept: () => Promise<void>;   // Creates draft config version
    reject: (reason: string) => Promise<void>;
    defer: () => Promise<void>;
    customize: () => void;          // Opens editor with suggested config
  };
}
```

---

## Database Schema (Complete)

```sql
-- ============================================
-- CORE TABLES
-- ============================================

-- Tenants
CREATE TABLE tenants (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name            VARCHAR(200) NOT NULL,
    settings        JSONB DEFAULT '{}',
    created_at      TIMESTAMPTZ DEFAULT NOW()
);

-- Users
CREATE TABLE users (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id       UUID REFERENCES tenants(id) NOT NULL,
    email           VARCHAR(200) NOT NULL,
    name            VARCHAR(200),
    roles           TEXT[] DEFAULT '{}',
    settings        JSONB DEFAULT '{}',
    created_at      TIMESTAMPTZ DEFAULT NOW(),
    UNIQUE(tenant_id, email)
);

-- ============================================
-- CONFIGURATION (Versioned)
-- ============================================

-- All configuration versions
CREATE TABLE config_versions (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id       UUID REFERENCES tenants(id) NOT NULL,
    config_type     VARCHAR(50) NOT NULL,
    config_id       VARCHAR(100) NOT NULL,
    version         INTEGER NOT NULL,
    status          VARCHAR(20) NOT NULL DEFAULT 'draft',
    config_data     JSONB NOT NULL,
    diff_summary    JSONB,
    simulation_result JSONB,
    created_by      UUID REFERENCES users(id),
    created_at      TIMESTAMPTZ DEFAULT NOW(),
    published_by    UUID REFERENCES users(id),
    published_at    TIMESTAMPTZ,
    UNIQUE(tenant_id, config_type, config_id, version)
);

-- Active configuration (fast reads)
CREATE TABLE active_config (
    tenant_id       UUID NOT NULL,
    config_type     VARCHAR(50) NOT NULL,
    config_id       VARCHAR(100) NOT NULL,
    version_id      UUID REFERENCES config_versions(id),
    config_data     JSONB NOT NULL,
    updated_at      TIMESTAMPTZ DEFAULT NOW(),
    PRIMARY KEY (tenant_id, config_type, config_id)
);

-- ============================================
-- ENTITY DATA
-- ============================================

-- Generic entity storage
CREATE TABLE entities (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id       UUID REFERENCES tenants(id) NOT NULL,
    entity_type     VARCHAR(100) NOT NULL,
    data            JSONB NOT NULL,
    workflow_state  VARCHAR(100),
    version         INTEGER DEFAULT 1,
    created_by      UUID REFERENCES users(id),
    created_at      TIMESTAMPTZ DEFAULT NOW(),
    updated_at      TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_entities_tenant_type ON entities(tenant_id, entity_type);
CREATE INDEX idx_entities_workflow ON entities(entity_type, workflow_state);
CREATE INDEX idx_entities_data ON entities USING GIN(data);

-- ============================================
-- AUDIT & ACTIVITY
-- ============================================

-- Command audit log (immutable)
CREATE TABLE audit_log (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id       UUID NOT NULL,
    entity_type     VARCHAR(100),
    entity_id       UUID,
    command_type    VARCHAR(100) NOT NULL,
    actor_id        UUID,
    actor_type      VARCHAR(20) NOT NULL,
    data_before     JSONB,
    data_after      JSONB,
    metadata        JSONB DEFAULT '{}',
    created_at      TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_audit_tenant_time ON audit_log(tenant_id, created_at DESC);
CREATE INDEX idx_audit_entity ON audit_log(entity_id);

-- Activity observations (for AI analysis)
CREATE TABLE activity_observations (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id       UUID NOT NULL,
    activity_type   VARCHAR(50) NOT NULL,
    entity_type     VARCHAR(100),
    entity_id       UUID,
    activity_data   JSONB NOT NULL,
    created_at      TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_observations_tenant_time ON activity_observations(tenant_id, created_at DESC);
CREATE INDEX idx_observations_type ON activity_observations(activity_type);

-- ============================================
-- AI SYSTEM
-- ============================================

-- Learned patterns
CREATE TABLE learned_patterns (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id       UUID REFERENCES tenants(id) NOT NULL,
    pattern_type    VARCHAR(50) NOT NULL,
    entity_type     VARCHAR(100),
    description     TEXT NOT NULL,
    pattern_data    JSONB NOT NULL,
    confidence      DECIMAL(3,2) NOT NULL,
    sample_count    INTEGER NOT NULL,
    last_seen       TIMESTAMPTZ DEFAULT NOW(),
    created_at      TIMESTAMPTZ DEFAULT NOW(),
    UNIQUE(tenant_id, pattern_type, entity_type, description)
);

-- AI suggestions
CREATE TABLE ai_suggestions (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id       UUID REFERENCES tenants(id) NOT NULL,
    suggestion_type VARCHAR(50) NOT NULL,
    entity_type     VARCHAR(100),
    title           VARCHAR(200) NOT NULL,
    description     TEXT NOT NULL,
    reasoning       TEXT,
    evidence        JSONB NOT NULL,
    confidence      DECIMAL(3,2) NOT NULL,
    suggested_config JSONB NOT NULL,
    impact_assessment JSONB,
    status          VARCHAR(20) DEFAULT 'pending',
    reviewed_by     UUID REFERENCES users(id),
    reviewed_at     TIMESTAMPTZ,
    review_notes    TEXT,
    created_at      TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_suggestions_tenant_status ON ai_suggestions(tenant_id, status);

-- Alerts
CREATE TABLE alerts (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id       UUID REFERENCES tenants(id) NOT NULL,
    entity_type     VARCHAR(100),
    entity_id       UUID,
    alert_type      VARCHAR(50) NOT NULL,
    severity        VARCHAR(20) NOT NULL,
    title           VARCHAR(200) NOT NULL,
    message         TEXT NOT NULL,
    context         JSONB,
    status          VARCHAR(20) DEFAULT 'active',
    acknowledged_by UUID REFERENCES users(id),
    acknowledged_at TIMESTAMPTZ,
    created_at      TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_alerts_tenant_status ON alerts(tenant_id, status, severity);

-- ============================================
-- EXTENSIONS
-- ============================================

-- Registered plugins
CREATE TABLE plugins (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id       UUID REFERENCES tenants(id) NOT NULL,
    plugin_id       VARCHAR(100) NOT NULL,
    name            VARCHAR(200) NOT NULL,
    version         VARCHAR(20) NOT NULL,
    definition      JSONB NOT NULL,
    enabled         BOOLEAN DEFAULT true,
    created_at      TIMESTAMPTZ DEFAULT NOW(),
    UNIQUE(tenant_id, plugin_id)
);

-- Webhook configurations
CREATE TABLE webhooks (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id       UUID REFERENCES tenants(id) NOT NULL,
    name            VARCHAR(200) NOT NULL,
    url             TEXT NOT NULL,
    method          VARCHAR(10) DEFAULT 'POST',
    headers         JSONB DEFAULT '{}',
    triggers        JSONB NOT NULL,
    retry_policy    JSONB DEFAULT '{"maxAttempts": 3, "backoffMs": 1000}',
    enabled         BOOLEAN DEFAULT true,
    created_at      TIMESTAMPTZ DEFAULT NOW()
);

-- Webhook delivery log
CREATE TABLE webhook_deliveries (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    webhook_id      UUID REFERENCES webhooks(id) NOT NULL,
    trigger_event   VARCHAR(100) NOT NULL,
    payload         JSONB NOT NULL,
    response_status INTEGER,
    response_body   TEXT,
    attempts        INTEGER DEFAULT 1,
    status          VARCHAR(20) DEFAULT 'pending',
    created_at      TIMESTAMPTZ DEFAULT NOW(),
    completed_at    TIMESTAMPTZ
);
```

---

## Implementation Plan (20 Weeks)

### Phase 1: Kernel Foundation (Weeks 1-6)

**Week 1-2: Project Setup & Data Layer**
```
□ Next.js 14 project with App Router
□ PostgreSQL setup (Neon or Supabase)
□ Drizzle ORM configuration
□ Database migrations (core tables)
□ Authentication (Clerk)
□ Basic tenant/user management
```

**Week 3-4: Entity System**
```
□ Entity schema storage and validation
□ Generic entity CRUD API
□ Audit logging middleware
□ Schema-driven form component
□ Schema-driven list component
□ Entity detail view
```

**Week 5-6: Configuration System**
```
□ Config version storage
□ Draft/publish workflow
□ Active config cache
□ Config diff generation
□ Rollback functionality
□ Admin UI for config management
```

### Phase 2: Workflow & Rules (Weeks 7-12)

**Week 7-8: Workflow Engine**
```
□ Workflow definition storage
□ State machine evaluator
□ Transition validation
□ Temporal integration for timeouts
□ Workflow state UI (badge, timeline)
```

**Week 9-10: Guard System**
```
□ Guard condition evaluator
□ Built-in operators implementation
□ Role-based guard evaluation
□ Function-ref escape hatch
□ Guard failure messaging
```

**Week 11-12: Action System**
```
□ Built-in actions implementation
□ Action execution pipeline
□ Plugin action interface
□ Webhook trigger system
□ Action result handling
```

### Phase 3: Governance & Extensions (Weeks 13-18)

**Week 13-14: Simulation Engine**
```
□ Test case definition format
□ Isolated execution context
□ Simulation runner
□ Result comparison
□ Simulation UI
```

**Week 15-16: Extension System**
```
□ Plugin registration API
□ Plugin sandbox (limited capabilities)
□ Computed function registry
□ Webhook management UI
□ Extension marketplace concept
```

**Week 17-18: Builder UX**
```
□ Visual schema editor
□ Visual workflow editor (simple, not React Flow)
□ Guard builder UI
□ Action configuration UI
□ Live preview
```

### Phase 4: AI Advisory (Weeks 19-24)

**Week 19-20: Observation System**
```
□ Activity observation pipeline
□ Pattern storage
□ Baseline calculation
□ Anomaly scoring
```

**Week 21-22: Analysis Engine**
```
□ Pattern detection (Claude integration)
□ Anomaly detection
□ Suggestion generation
□ Evidence compilation
```

**Week 23-24: Advisory UI**
```
□ Suggestions dashboard
□ Evidence presentation
□ Accept/reject workflow
□ Alert notifications
□ Impact preview
```

---

## Success Criteria

### Phase 1 Gate (Week 6)
- [ ] Users can define entity schemas via UI
- [ ] CRUD operations work with audit logging
- [ ] Config versioning with rollback works
- [ ] No JSON editing required for basic operations

### Phase 2 Gate (Week 12)
- [ ] Workflows with guards function correctly
- [ ] Transitions respect role permissions
- [ ] Timeouts trigger via Temporal
- [ ] 90% of simple business rules expressible

### Phase 3 Gate (Week 18)
- [ ] Simulation catches breaking changes
- [ ] Plugins can extend functionality
- [ ] Builder UX rated 4+/5 by test users
- [ ] Governance prevents bad deploys

### Phase 4 Gate (Week 24)
- [ ] AI detects 3+ valid patterns per tenant
- [ ] Anomaly false positive rate <25%
- [ ] 40%+ suggestion acceptance rate
- [ ] Users report AI is "helpful"

---

## Final Verdict

**Feasibility: 72%**

This architecture is buildable in 24 weeks with a small team. The key decisions that make it viable:

1. **Constrained kernel** - Only simple operators, no expressions
2. **Explicit escape hatches** - Plugins handle complex logic
3. **Governance first** - All changes go through review
4. **AI as advisor** - Suggests, never applies
5. **Guarded workflows** - Rules and workflows unified at transitions

**The single biggest risk** is still authoring UX. If the builder feels like a JSON editor, adoption will fail. Invest heavily in weeks 17-18.

**What we deliberately deferred**:
- DSL compiler (maybe never)
- AI auto-apply (maybe Phase 5 if trust builds)
- React Flow visual editor (maybe Phase 5)
- ClickHouse analytics (maybe Phase 6)
- Full event sourcing (probably never)

**Ship the kernel. Let usage data guide the roadmap.**
