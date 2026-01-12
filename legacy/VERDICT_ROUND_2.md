# AI-Native Business App Platform: Verdict Round 2

## Complete Architecture Reassessment & Implementation Plan

**Date**: 2025-01-12  
**Status**: Final Technical Specification  
**Purpose**: Implementation-ready plan for Claude Code

---

## Part 1: Analysis Summary (From Original Documents)

### What We Analyzed

Three documents were reviewed:
1. **HIGH_LEVEL_PLAN.md** - Original vision for AI-native generic app platform
2. **FEASIBILITY_VERDICTS.md** - Component-by-component risk assessment
3. **DSL_COMPILER_ANALYSIS.md** - Deep dive on SKILL.md → JSON compilation

### Original Vision

> "Build a generic platform where non-technical users build apps with AI"

Core concepts:
- Canonical Model as single source of truth
- SKILL.md (Markdown) → JSON compilation
- Workflow/Rules separation
- AI as primary author of business logic
- React Flow for visual workflow editing
- LangChain agents for operations and building

### Key Findings from Feasibility Analysis

| Component | Feasibility | Risk | Timeline |
|-----------|-------------|------|----------|
| Canonical Model | 60% | HIGH | 6-12 months |
| DSL Compiler | 40% | VERY HIGH | 12-24 months |
| Workflow/Rules Split | 50% | MEDIUM | N/A |
| React Flow Sync | 70% | LOW | 2-3 months |
| AI Safety Workflow | 50% | HIGH | 3-4 months |
| LangChain Agents | 40% | VERY HIGH | 6-12 months |
| Schema-Driven UI | 60% | MEDIUM | 4-6 months |
| Generic Platform | 30% | VERY HIGH | Ongoing |

**Overall Feasibility: 60%**

### DSL Compiler Deep Dive

The SKILL.md → JSON compiler is essentially building a domain-specific language. Required components:

```
SKILL.md → Parser → AST → Semantic Analysis → Optimizer → Code Generator → Canonical JSON
```

**Effort breakdown:**

| Component | Time | Risk |
|-----------|------|------|
| Language Spec | 2 months | High |
| Parser | 3 months | Medium |
| Type Checker | 3 months | High |
| Code Generator | 2 months | Low |
| Optimizer | 2 months | Medium |
| Error Handling | 1 month | Low |
| Testing | 2 months | Medium |
| Tooling | 2 months | Low |
| **Total** | **17 months** | **High** |

**Critical issues:**
- AI-generated SKILL.md will have hallucinations, inconsistencies
- Round-trip (SKILL.md → JSON → SKILL.md) is 10x harder than one-way
- Language versioning and schema evolution are ongoing burdens
- Requires compiler engineering expertise (rare, expensive)

### Fundamental Tension Identified

The plan tries to be both:
1. **Generic** (pluggable, extensible, configurable)
2. **Simple for non-technical users**

This is historically unprecedented. Every existing tool picks one:
- Salesforce: Generic, requires certification
- Airtable: Simple, not truly generic
- Excel: Ultimate generic tool, requires expertise

---

## Part 2: Strategic Pivot

### The Problem with "AI Builds Apps"

The original USP was: *"AI builds your business apps"*

This has three issues:

1. **Everyone is trying this** - Not differentiated
2. **LLM agents aren't reliable enough** - 30-50% error rates in production
3. **Building a DSL compiler is a multi-year project** - Blocks all other progress

### The New USP

> **"AI that learns your business and gets smarter over time"**

Key shift: AI is an **observer and advisor**, not the builder.

| Old Approach | New Approach |
|--------------|--------------|
| AI authors business logic | AI suggests improvements |
| Humans approve AI work | AI learns from human work |
| Complex DSL compiler | Simple JSON + activity analysis |
| Generic platform first | Concrete apps first |
| 2-3 year timeline | 5-6 month timeline |

### Why This Is Better

1. **Unique**: Not "AI builds apps" (everyone) but "AI learns your patterns" (nobody)
2. **Defensible**: Value compounds as AI learns your specific business
3. **Achievable**: No DSL compiler, no complex architecture
4. **Measurable**: "AI prevented X errors, suggested Y improvements"

### AI Value That Users Will Pay For

| AI Capability | User Value | Difficulty | Differentiation |
|---------------|------------|------------|-----------------|
| NL data entry | High | Low | Low (table stakes) |
| Smart search | Medium | Low | Low |
| Report generation | Medium | Medium | Low |
| **Schema inference** | High | Medium | High |
| **Rule inference from patterns** | Very High | High | **Very High** |
| **Anomaly detection + explanation** | Very High | Medium | **High** |
| **Workflow optimization** | High | High | **Very High** |

The bottom rows are where differentiation lives. Focus there.

---

## Part 3: Revised Architecture

### Core Principle

```
Simple CRUD + Activity Logging → AI Analysis → Suggestions → Human Approval → Rules/Workflow Updates
```

Not event sourcing. Not DSL compilation. Just:
1. Log what users do
2. AI finds patterns
3. Humans approve suggestions
4. System gets smarter

### Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                         Frontend                                │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────┐ │
│  │   Next.js   │  │   Schema    │  │      AI Copilot         │ │
│  │   App       │  │   Forms     │  │   (suggestions UI)      │ │
│  └─────────────┘  └─────────────┘  └─────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                          API Layer                              │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │                    Next.js API Routes                       ││
│  │     Commands (writes) │ Queries (reads) │ AI (analysis)     ││
│  └─────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────┘
                              │
         ┌────────────────────┼────────────────────┐
         ▼                    ▼                    ▼
┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
│   PostgreSQL    │  │    Temporal     │  │   Claude API    │
│   (data + logs) │  │   (workflows)   │  │   (analysis)    │
└─────────────────┘  └─────────────────┘  └─────────────────┘
```

### What We're NOT Building

| Component | Original Plan | New Plan |
|-----------|---------------|----------|
| DSL Compiler | SKILL.md → JSON | ❌ Skip entirely |
| Event Sourcing | Full event store + projections | ❌ Simple activity log |
| React Flow | Visual workflow editor | ❌ Phase 2 (if needed) |
| LangChain | Agents for ops and building | ❌ Direct Claude API |
| ClickHouse | Analytics database | ❌ Postgres only |
| Custom Workflow Engine | Build from scratch | ❌ Use Temporal |

### What We ARE Building

| Component | Description | Priority |
|-----------|-------------|----------|
| CRUD API | Simple entity management | P0 |
| Activity Log | Append-only log of all actions | P0 |
| Temporal Integration | Workflow execution | P0 |
| JSON Rules Engine | Simple condition evaluator | P0 |
| Schema-Driven Forms | Metadata-driven UI | P0 |
| AI Pattern Detection | Analyze logs, find patterns | P0 |
| AI Suggestions Queue | Propose rules/changes | P0 |
| Anomaly Detection | Flag unusual activity | P0 |

---

## Part 4: Database Schema

### Core Entities (Simple CRUD)

```sql
-- Tenants (multi-tenancy)
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
    role            VARCHAR(50) DEFAULT 'user',
    created_at      TIMESTAMPTZ DEFAULT NOW(),
    UNIQUE(tenant_id, email)
);

-- Entity Schemas (defines what entities exist)
CREATE TABLE entity_schemas (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id       UUID REFERENCES tenants(id) NOT NULL,
    entity_type     VARCHAR(100) NOT NULL,
    data_schema     JSONB NOT NULL,  -- JSON Schema
    ui_schema       JSONB NOT NULL,  -- UI hints
    version         INTEGER DEFAULT 1,
    created_at      TIMESTAMPTZ DEFAULT NOW(),
    updated_at      TIMESTAMPTZ DEFAULT NOW(),
    UNIQUE(tenant_id, entity_type)
);

-- Generic Entity Storage
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
CREATE INDEX idx_entities_state ON entities(workflow_state);
CREATE INDEX idx_entities_data ON entities USING GIN(data);

-- Workflow Definitions
CREATE TABLE workflow_definitions (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id       UUID REFERENCES tenants(id) NOT NULL,
    entity_type     VARCHAR(100) NOT NULL,
    name            VARCHAR(200) NOT NULL,
    definition      JSONB NOT NULL,  -- States, transitions
    version         INTEGER DEFAULT 1,
    status          VARCHAR(20) DEFAULT 'active',
    created_at      TIMESTAMPTZ DEFAULT NOW(),
    UNIQUE(tenant_id, entity_type, name)
);

-- Rule Definitions
CREATE TABLE rule_definitions (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id       UUID REFERENCES tenants(id) NOT NULL,
    entity_type     VARCHAR(100),
    name            VARCHAR(200) NOT NULL,
    trigger_event   VARCHAR(100) NOT NULL,
    definition      JSONB NOT NULL,  -- Conditions, actions
    priority        INTEGER DEFAULT 0,
    enabled         BOOLEAN DEFAULT true,
    created_at      TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_rules_trigger ON rule_definitions(tenant_id, trigger_event);
```

### Activity Log (For AI Analysis)

```sql
-- Activity log - append only, never delete
CREATE TABLE activity_log (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id       UUID NOT NULL,
    
    -- What happened
    entity_type     VARCHAR(100),
    entity_id       UUID,
    action          VARCHAR(100) NOT NULL,  -- 'created', 'updated', 'submitted', 'approved'
    
    -- Who did it
    actor_id        UUID,
    actor_type      VARCHAR(20) NOT NULL,  -- 'user', 'system', 'ai'
    
    -- State change
    data_before     JSONB,
    data_after      JSONB,
    
    -- Context
    metadata        JSONB DEFAULT '{}',  -- IP, user agent, correlation ID, etc.
    
    created_at      TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_activity_tenant_time ON activity_log(tenant_id, created_at DESC);
CREATE INDEX idx_activity_entity ON activity_log(entity_type, entity_id);
CREATE INDEX idx_activity_action ON activity_log(action);
CREATE INDEX idx_activity_actor ON activity_log(actor_id);

-- Partition by month for performance (optional, for scale)
-- CREATE TABLE activity_log (...) PARTITION BY RANGE (created_at);
```

### AI System Tables

```sql
-- Learned patterns (what AI has discovered)
CREATE TABLE learned_patterns (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id       UUID REFERENCES tenants(id) NOT NULL,
    
    pattern_type    VARCHAR(50) NOT NULL,  -- 'approval_rule', 'anomaly_baseline', 'workflow_timing'
    entity_type     VARCHAR(100),
    
    description     TEXT NOT NULL,
    pattern_data    JSONB NOT NULL,  -- Statistical data, thresholds, etc.
    
    confidence      DECIMAL(3,2) NOT NULL,  -- 0.00 to 1.00
    sample_count    INTEGER NOT NULL,
    
    last_seen       TIMESTAMPTZ DEFAULT NOW(),
    created_at      TIMESTAMPTZ DEFAULT NOW(),
    
    UNIQUE(tenant_id, pattern_type, entity_type, description)
);

CREATE INDEX idx_patterns_tenant ON learned_patterns(tenant_id);
CREATE INDEX idx_patterns_type ON learned_patterns(pattern_type);

-- AI suggestions queue
CREATE TABLE ai_suggestions (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id       UUID REFERENCES tenants(id) NOT NULL,
    
    -- What type of suggestion
    suggestion_type VARCHAR(50) NOT NULL,  -- 'rule', 'workflow', 'alert', 'optimization'
    entity_type     VARCHAR(100),
    
    -- Human-readable
    title           VARCHAR(200) NOT NULL,
    description     TEXT NOT NULL,
    
    -- Supporting evidence
    evidence        JSONB NOT NULL,  -- Examples, data points
    confidence      DECIMAL(3,2) NOT NULL,
    
    -- The actual proposed change
    suggestion      JSONB NOT NULL,  -- Rule definition, workflow change, etc.
    
    -- Status tracking
    status          VARCHAR(20) DEFAULT 'pending',  -- 'pending', 'accepted', 'rejected', 'dismissed'
    reviewed_by     UUID REFERENCES users(id),
    reviewed_at     TIMESTAMPTZ,
    review_notes    TEXT,
    
    created_at      TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_suggestions_tenant_status ON ai_suggestions(tenant_id, status);
CREATE INDEX idx_suggestions_type ON ai_suggestions(suggestion_type);

-- Real-time alerts
CREATE TABLE alerts (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id       UUID REFERENCES tenants(id) NOT NULL,
    
    entity_type     VARCHAR(100),
    entity_id       UUID,
    
    alert_type      VARCHAR(50) NOT NULL,  -- 'anomaly', 'threshold', 'pattern_break'
    severity        VARCHAR(20) NOT NULL,  -- 'low', 'medium', 'high', 'critical'
    
    title           VARCHAR(200) NOT NULL,
    message         TEXT NOT NULL,
    
    context         JSONB,  -- Additional data for investigation
    suggested_actions JSONB,  -- What to do about it
    
    status          VARCHAR(20) DEFAULT 'active',  -- 'active', 'acknowledged', 'resolved', 'dismissed'
    acknowledged_by UUID REFERENCES users(id),
    acknowledged_at TIMESTAMPTZ,
    
    created_at      TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_alerts_tenant_status ON alerts(tenant_id, status);
CREATE INDEX idx_alerts_severity ON alerts(severity);
CREATE INDEX idx_alerts_entity ON alerts(entity_id);

-- AI analysis job tracking
CREATE TABLE ai_analysis_jobs (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id       UUID REFERENCES tenants(id) NOT NULL,
    
    job_type        VARCHAR(50) NOT NULL,  -- 'pattern_detection', 'anomaly_baseline', 'optimization'
    status          VARCHAR(20) DEFAULT 'pending',  -- 'pending', 'running', 'completed', 'failed'
    
    started_at      TIMESTAMPTZ,
    completed_at    TIMESTAMPTZ,
    
    input_params    JSONB,
    result          JSONB,
    error           TEXT,
    
    created_at      TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_jobs_tenant_status ON ai_analysis_jobs(tenant_id, status);
```

---

## Part 5: Core Implementation

### 5.1 Project Structure

```
/
├── apps/
│   └── web/                          # Next.js application
│       ├── app/
│       │   ├── (auth)/               # Auth pages
│       │   │   ├── login/
│       │   │   └── register/
│       │   ├── (workspace)/          # Main app (authenticated)
│       │   │   ├── layout.tsx        # Workspace layout with sidebar
│       │   │   ├── dashboard/        # Dashboard with alerts, suggestions
│       │   │   ├── [entityType]/     # Dynamic entity pages
│       │   │   │   ├── page.tsx      # List view
│       │   │   │   ├── [id]/         # Detail view
│       │   │   │   └── new/          # Create view
│       │   │   ├── suggestions/      # AI suggestions review
│       │   │   ├── rules/            # Rule management
│       │   │   ├── workflows/        # Workflow management
│       │   │   └── settings/         # Tenant settings
│       │   └── api/
│       │       ├── entities/         # Entity CRUD
│       │       ├── workflows/        # Workflow operations
│       │       ├── rules/            # Rule management
│       │       ├── ai/               # AI endpoints
│       │       │   ├── analyze/      # Trigger analysis
│       │       │   ├── suggestions/  # Manage suggestions
│       │       │   └── chat/         # AI assistant
│       │       └── auth/             # Auth endpoints
│       ├── components/
│       │   ├── ui/                   # shadcn components
│       │   ├── forms/                # Schema-driven forms
│       │   ├── entities/             # Entity components
│       │   ├── suggestions/          # AI suggestion UI
│       │   └── alerts/               # Alert components
│       └── lib/
│           ├── db/                   # Database client
│           ├── commands/             # Command handlers
│           ├── rules/                # Rules engine
│           ├── ai/                   # AI integration
│           └── temporal/             # Temporal client
│
├── packages/
│   ├── db/                           # Shared database utilities
│   │   ├── schema.ts                 # Drizzle schema
│   │   ├── migrations/               # SQL migrations
│   │   └── queries/                  # Common queries
│   │
│   ├── rules-engine/                 # JSON rules evaluator
│   │   ├── evaluator.ts
│   │   ├── conditions.ts
│   │   └── actions.ts
│   │
│   └── ai-analyzer/                  # AI analysis logic
│       ├── patterns.ts               # Pattern detection
│       ├── anomalies.ts              # Anomaly detection
│       └── suggestions.ts            # Suggestion generation
│
├── temporal/                         # Temporal worker
│   ├── worker.ts
│   ├── workflows/
│   │   └── entity-lifecycle.ts
│   └── activities/
│       ├── notifications.ts
│       ├── rules.ts
│       └── ai-analysis.ts
│
└── scripts/
    ├── seed.ts                       # Seed data
    └── analyze.ts                    # Manual AI analysis trigger
```

### 5.2 Command Handler Pattern

```typescript
// apps/web/lib/commands/types.ts

export interface Command<T = unknown> {
  type: string;
  payload: T;
  actor: {
    id: string;
    type: 'user' | 'system' | 'ai';
  };
  metadata?: {
    correlationId?: string;
    ipAddress?: string;
    userAgent?: string;
  };
}

export interface CommandResult<T = unknown> {
  success: boolean;
  data?: T;
  errors?: ValidationError[];
}

export interface ValidationError {
  field?: string;
  code: string;
  message: string;
}
```

```typescript
// apps/web/lib/commands/handler.ts

import { db } from '@/lib/db';
import { evaluateRules } from '@/lib/rules';
import { logActivity } from '@/lib/activity';
import { checkForAnomalies } from '@/lib/ai/anomalies';
import { triggerWorkflow } from '@/lib/temporal';

export async function handleCommand<T>(
  command: Command<T>
): Promise<CommandResult> {
  return db.transaction(async (tx) => {
    // 1. Validate command payload
    const validation = await validateCommand(command);
    if (!validation.valid) {
      return { success: false, errors: validation.errors };
    }

    // 2. Load current entity state (if updating)
    const currentState = await loadCurrentState(command, tx);

    // 3. Evaluate rules (validation rules, not workflow)
    const ruleResult = await evaluateRules({
      event: command.type,
      entity: currentState,
      payload: command.payload,
      tx,
    });

    if (ruleResult.blocked) {
      return { success: false, errors: ruleResult.errors };
    }

    // 4. Apply command to get new state
    const newState = applyCommand(currentState, command);

    // 5. Save entity with optimistic locking
    const saved = await saveEntity(newState, currentState?.version, tx);

    // 6. Log activity (for AI analysis)
    await logActivity({
      tenantId: command.actor.tenantId,
      entityType: getEntityType(command),
      entityId: saved.id,
      action: command.type,
      actorId: command.actor.id,
      actorType: command.actor.type,
      dataBefore: currentState?.data,
      dataAfter: saved.data,
      metadata: command.metadata,
    }, tx);

    // 7. Execute rule actions (side effects)
    for (const action of ruleResult.actions) {
      await executeAction(action, saved, tx);
    }

    // 8. Trigger workflow if state change
    if (isWorkflowTransition(command)) {
      await triggerWorkflow(saved);
    }

    // 9. Check for anomalies (async, don't block)
    checkForAnomalies(command, currentState, saved).catch(console.error);

    return { success: true, data: saved };
  });
}
```

### 5.3 Rules Engine (Simple JSON Evaluator)

```typescript
// packages/rules-engine/evaluator.ts

export interface Rule {
  id: string;
  name: string;
  triggerEvent: string;
  conditions: Condition[];
  actions: Action[];
  priority: number;
  enabled: boolean;
}

export interface Condition {
  field: string;
  operator: 'eq' | 'ne' | 'gt' | 'lt' | 'gte' | 'lte' | 'in' | 'contains' | 'exists';
  value: unknown;
  negate?: boolean;
}

export interface Action {
  type: string;
  params: Record<string, unknown>;
}

export interface EvaluationContext {
  event: string;
  entity: Record<string, unknown>;
  payload: Record<string, unknown>;
  actor: { id: string; type: string };
}

export interface EvaluationResult {
  blocked: boolean;
  errors: ValidationError[];
  actions: Action[];
  matchedRules: string[];
}

export class RulesEngine {
  async evaluate(
    context: EvaluationContext,
    rules: Rule[]
  ): Promise<EvaluationResult> {
    const matchingRules = rules
      .filter(r => r.enabled && r.triggerEvent === context.event)
      .sort((a, b) => b.priority - a.priority);

    const result: EvaluationResult = {
      blocked: false,
      errors: [],
      actions: [],
      matchedRules: [],
    };

    for (const rule of matchingRules) {
      const matches = this.evaluateConditions(rule.conditions, context);
      
      if (matches) {
        result.matchedRules.push(rule.id);
        
        for (const action of rule.actions) {
          if (action.type === 'block') {
            result.blocked = true;
            result.errors.push({
              code: 'RULE_BLOCKED',
              message: action.params.message as string || `Blocked by rule: ${rule.name}`,
            });
          } else {
            result.actions.push(action);
          }
        }
      }
    }

    return result;
  }

  private evaluateConditions(
    conditions: Condition[],
    context: EvaluationContext
  ): boolean {
    // All conditions must match (AND logic)
    return conditions.every(condition => {
      const value = this.resolveField(condition.field, context);
      const matches = this.compare(value, condition.operator, condition.value);
      return condition.negate ? !matches : matches;
    });
  }

  private resolveField(field: string, context: EvaluationContext): unknown {
    // Support dot notation: "entity.totalValue", "payload.newStatus"
    const parts = field.split('.');
    let value: unknown = context;
    
    for (const part of parts) {
      if (value === null || value === undefined) return undefined;
      value = (value as Record<string, unknown>)[part];
    }
    
    return value;
  }

  private compare(value: unknown, operator: string, target: unknown): boolean {
    switch (operator) {
      case 'eq':
        return value === target;
      case 'ne':
        return value !== target;
      case 'gt':
        return (value as number) > (target as number);
      case 'lt':
        return (value as number) < (target as number);
      case 'gte':
        return (value as number) >= (target as number);
      case 'lte':
        return (value as number) <= (target as number);
      case 'in':
        return (target as unknown[]).includes(value);
      case 'contains':
        return String(value).includes(String(target));
      case 'exists':
        return value !== null && value !== undefined;
      default:
        return false;
    }
  }
}
```

### 5.4 AI Pattern Detection

```typescript
// packages/ai-analyzer/patterns.ts

import Anthropic from '@anthropic-ai/sdk';
import { db } from '@/lib/db';

const anthropic = new Anthropic();

export interface DetectedPattern {
  type: 'approval_rule' | 'timing_pattern' | 'assignment_rule' | 'threshold';
  description: string;
  confidence: number;
  sampleCount: number;
  evidence: unknown[];
  suggestedRule?: Rule;
}

export async function analyzePatterns(tenantId: string): Promise<DetectedPattern[]> {
  // 1. Load recent activity
  const activity = await db.query(`
    SELECT 
      entity_type,
      action,
      actor_id,
      data_before,
      data_after,
      metadata,
      created_at
    FROM activity_log 
    WHERE tenant_id = $1 
      AND created_at > NOW() - INTERVAL '30 days'
    ORDER BY created_at DESC
    LIMIT 1000
  `, [tenantId]);

  // 2. Load existing rules and patterns
  const existingRules = await db.query(`
    SELECT * FROM rule_definitions WHERE tenant_id = $1
  `, [tenantId]);

  const existingPatterns = await db.query(`
    SELECT * FROM learned_patterns WHERE tenant_id = $1
  `, [tenantId]);

  // 3. Call Claude for analysis
  const response = await anthropic.messages.create({
    model: 'claude-sonnet-4-20250514',
    max_tokens: 4096,
    system: `You are a business process analyst. Analyze activity logs to identify:

1. **Approval patterns**: Who approves what, and under what conditions?
   - Look for: entity.approved actions, who does them, what values trigger them
   
2. **Timing patterns**: How long do things typically take?
   - Look for: time between created → submitted → approved
   
3. **Assignment patterns**: Who handles what?
   - Look for: which users work on which entity types or values
   
4. **Threshold patterns**: What triggers special handling?
   - Look for: consistent cut-offs in numeric fields

For each pattern, provide:
- Type (approval_rule, timing_pattern, assignment_rule, threshold)
- Clear description
- Confidence (0.0 to 1.0)
- Sample count (how many examples support this)
- Evidence (specific examples from the data)
- Suggested rule (if applicable, in JSON format)

Only report patterns with confidence >= 0.7 and sample count >= 5.

Return valid JSON array of patterns.`,
    messages: [{
      role: 'user',
      content: `
## Activity Log (last 30 days)
${JSON.stringify(activity.slice(0, 500), null, 2)}

## Existing Rules
${JSON.stringify(existingRules, null, 2)}

## Already Identified Patterns
${JSON.stringify(existingPatterns, null, 2)}

Identify NEW patterns not already captured in existing rules or patterns.
Return JSON array of DetectedPattern objects.
      `,
    }],
  });

  // 4. Parse and validate response
  const content = response.content[0];
  if (content.type !== 'text') {
    return [];
  }

  try {
    const patterns = JSON.parse(content.text) as DetectedPattern[];
    return patterns.filter(p => p.confidence >= 0.7 && p.sampleCount >= 5);
  } catch {
    console.error('Failed to parse AI response');
    return [];
  }
}

export async function createSuggestionsFromPatterns(
  tenantId: string,
  patterns: DetectedPattern[]
): Promise<void> {
  for (const pattern of patterns) {
    // Check if similar suggestion already exists
    const existing = await db.query(`
      SELECT id FROM ai_suggestions 
      WHERE tenant_id = $1 
        AND suggestion_type = $2 
        AND description = $3
        AND status = 'pending'
    `, [tenantId, pattern.type, pattern.description]);

    if (existing.length > 0) continue;

    // Create suggestion
    await db.query(`
      INSERT INTO ai_suggestions (
        tenant_id, suggestion_type, title, description, 
        evidence, confidence, suggestion
      ) VALUES ($1, $2, $3, $4, $5, $6, $7)
    `, [
      tenantId,
      pattern.type === 'approval_rule' || pattern.type === 'threshold' ? 'rule' : 'optimization',
      generateTitle(pattern),
      pattern.description,
      JSON.stringify(pattern.evidence),
      pattern.confidence,
      JSON.stringify(pattern.suggestedRule || {}),
    ]);
  }
}

function generateTitle(pattern: DetectedPattern): string {
  switch (pattern.type) {
    case 'approval_rule':
      return 'Suggested Approval Rule';
    case 'timing_pattern':
      return 'Process Timing Insight';
    case 'assignment_rule':
      return 'Work Assignment Pattern';
    case 'threshold':
      return 'Threshold Rule Detected';
    default:
      return 'Process Pattern Detected';
  }
}
```

### 5.5 Anomaly Detection

```typescript
// packages/ai-analyzer/anomalies.ts

import Anthropic from '@anthropic-ai/sdk';
import { db } from '@/lib/db';

const anthropic = new Anthropic();

export interface AnomalyResult {
  isAnomaly: boolean;
  score: number;  // 0.0 to 1.0
  explanation: string;
  suggestedActions: string[];
  relatedAlerts?: string[];
}

export async function checkForAnomalies(
  command: Command,
  before: Entity | null,
  after: Entity
): Promise<AnomalyResult | null> {
  const tenantId = command.actor.tenantId;
  const entityType = after.entityType;

  // 1. Load baseline for this entity type
  const baseline = await db.query(`
    SELECT * FROM learned_patterns 
    WHERE tenant_id = $1 
      AND pattern_type = 'anomaly_baseline' 
      AND entity_type = $2
  `, [tenantId, entityType]);

  if (!baseline.length) {
    // No baseline yet, can't detect anomalies
    return null;
  }

  // 2. Quick statistical check
  const stats = baseline[0].pattern_data as BaselineStats;
  const anomalyScore = calculateAnomalyScore(after.data, stats);

  if (anomalyScore < 0.7) {
    // Not anomalous enough
    return null;
  }

  // 3. Get AI explanation for high-scoring anomalies
  const response = await anthropic.messages.create({
    model: 'claude-sonnet-4-20250514',
    max_tokens: 500,
    messages: [{
      role: 'user',
      content: `
This ${entityType} action appears unusual. Explain why in 2-3 sentences and suggest what to check.

Action: ${command.type}
Before: ${JSON.stringify(before?.data || null)}
After: ${JSON.stringify(after.data)}

Historical baseline:
- Average values: ${JSON.stringify(stats.averages)}
- Typical ranges: ${JSON.stringify(stats.ranges)}
- Anomaly score: ${anomalyScore.toFixed(2)}

Explain why this is unusual and what should be investigated.
      `,
    }],
  });

  const explanation = response.content[0].type === 'text' 
    ? response.content[0].text 
    : 'Anomaly detected';

  // 4. Create alert
  await db.query(`
    INSERT INTO alerts (
      tenant_id, entity_type, entity_id, 
      alert_type, severity, title, message,
      context, suggested_actions
    ) VALUES ($1, $2, $3, $4, $5, $6, $7, $8, $9)
  `, [
    tenantId,
    entityType,
    after.id,
    'anomaly',
    anomalyScore > 0.9 ? 'high' : 'medium',
    `Unusual ${entityType} activity detected`,
    explanation,
    JSON.stringify({ command: command.type, before, after, score: anomalyScore }),
    JSON.stringify(extractSuggestedActions(explanation)),
  ]);

  return {
    isAnomaly: true,
    score: anomalyScore,
    explanation,
    suggestedActions: extractSuggestedActions(explanation),
  };
}

interface BaselineStats {
  averages: Record<string, number>;
  ranges: Record<string, { min: number; max: number }>;
  stdDevs: Record<string, number>;
}

function calculateAnomalyScore(
  data: Record<string, unknown>,
  baseline: BaselineStats
): number {
  const numericFields = Object.keys(baseline.averages);
  if (numericFields.length === 0) return 0;

  let totalDeviation = 0;
  let fieldCount = 0;

  for (const field of numericFields) {
    const value = data[field];
    if (typeof value !== 'number') continue;

    const avg = baseline.averages[field];
    const stdDev = baseline.stdDevs[field] || 1;
    const range = baseline.ranges[field];

    // Z-score
    const zScore = Math.abs((value - avg) / stdDev);
    
    // Out of range penalty
    const outOfRange = value < range.min || value > range.max;
    
    // Normalize to 0-1
    const fieldScore = Math.min(1, zScore / 3) * (outOfRange ? 1.5 : 1);
    
    totalDeviation += fieldScore;
    fieldCount++;
  }

  return Math.min(1, totalDeviation / fieldCount);
}

function extractSuggestedActions(explanation: string): string[] {
  // Simple extraction - could be enhanced with structured output
  const actions: string[] = [];
  
  if (explanation.toLowerCase().includes('review')) {
    actions.push('Review the transaction details');
  }
  if (explanation.toLowerCase().includes('verify')) {
    actions.push('Verify with the submitter');
  }
  if (explanation.toLowerCase().includes('photo') || explanation.toLowerCase().includes('evidence')) {
    actions.push('Request supporting documentation');
  }
  if (explanation.toLowerCase().includes('manager') || explanation.toLowerCase().includes('supervisor')) {
    actions.push('Escalate to manager');
  }
  
  if (actions.length === 0) {
    actions.push('Investigate before proceeding');
  }
  
  return actions;
}

// Build baseline from historical data
export async function buildBaseline(
  tenantId: string,
  entityType: string
): Promise<void> {
  // Load last 90 days of data
  const entities = await db.query(`
    SELECT data FROM entities 
    WHERE tenant_id = $1 
      AND entity_type = $2 
      AND created_at > NOW() - INTERVAL '90 days'
  `, [tenantId, entityType]);

  if (entities.length < 10) {
    // Not enough data for baseline
    return;
  }

  // Calculate statistics
  const numericFields = new Map<string, number[]>();
  
  for (const entity of entities) {
    for (const [key, value] of Object.entries(entity.data)) {
      if (typeof value === 'number') {
        if (!numericFields.has(key)) {
          numericFields.set(key, []);
        }
        numericFields.get(key)!.push(value);
      }
    }
  }

  const stats: BaselineStats = {
    averages: {},
    ranges: {},
    stdDevs: {},
  };

  for (const [field, values] of numericFields) {
    if (values.length < 5) continue;
    
    const avg = values.reduce((a, b) => a + b, 0) / values.length;
    const stdDev = Math.sqrt(
      values.reduce((sum, v) => sum + Math.pow(v - avg, 2), 0) / values.length
    );
    
    stats.averages[field] = avg;
    stats.stdDevs[field] = stdDev;
    stats.ranges[field] = {
      min: Math.min(...values),
      max: Math.max(...values),
    };
  }

  // Save baseline
  await db.query(`
    INSERT INTO learned_patterns (
      tenant_id, pattern_type, entity_type, description, 
      pattern_data, confidence, sample_count
    ) VALUES ($1, 'anomaly_baseline', $2, $3, $4, $5, $6)
    ON CONFLICT (tenant_id, pattern_type, entity_type, description)
    DO UPDATE SET 
      pattern_data = $4, 
      sample_count = $6, 
      last_seen = NOW()
  `, [
    tenantId,
    entityType,
    `Baseline statistics for ${entityType}`,
    JSON.stringify(stats),
    0.95,  // High confidence for statistical baseline
    entities.length,
  ]);
}
```

### 5.6 Schema-Driven Forms

```typescript
// apps/web/components/forms/SchemaForm.tsx

'use client';

import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';
import { useMemo } from 'react';

interface DataSchema {
  type: 'object';
  properties: Record<string, FieldSchema>;
  required?: string[];
}

interface FieldSchema {
  type: 'string' | 'number' | 'boolean' | 'array';
  title?: string;
  description?: string;
  format?: 'date' | 'datetime' | 'email' | 'currency' | 'barcode';
  enum?: string[];
  minimum?: number;
  maximum?: number;
}

interface UISchema {
  order?: string[];
  fields?: Record<string, UIFieldHints>;
}

interface UIFieldHints {
  widget?: 'text' | 'textarea' | 'select' | 'radio' | 'checkbox' | 'date' | 'currency' | 'barcode';
  placeholder?: string;
  hidden?: boolean;
  disabled?: boolean;
  colSpan?: 1 | 2;
}

interface SchemaFormProps {
  dataSchema: DataSchema;
  uiSchema?: UISchema;
  defaultValues?: Record<string, unknown>;
  onSubmit: (data: Record<string, unknown>) => Promise<void>;
  readOnly?: boolean;
}

export function SchemaForm({
  dataSchema,
  uiSchema = {},
  defaultValues = {},
  onSubmit,
  readOnly = false,
}: SchemaFormProps) {
  // Generate Zod schema from JSON Schema
  const zodSchema = useMemo(() => {
    const shape: Record<string, z.ZodTypeAny> = {};
    
    for (const [field, schema] of Object.entries(dataSchema.properties)) {
      let fieldSchema: z.ZodTypeAny;
      
      switch (schema.type) {
        case 'string':
          fieldSchema = z.string();
          if (schema.format === 'email') {
            fieldSchema = z.string().email();
          }
          break;
        case 'number':
          fieldSchema = z.number();
          if (schema.minimum !== undefined) {
            fieldSchema = (fieldSchema as z.ZodNumber).min(schema.minimum);
          }
          if (schema.maximum !== undefined) {
            fieldSchema = (fieldSchema as z.ZodNumber).max(schema.maximum);
          }
          break;
        case 'boolean':
          fieldSchema = z.boolean();
          break;
        default:
          fieldSchema = z.unknown();
      }
      
      if (!dataSchema.required?.includes(field)) {
        fieldSchema = fieldSchema.optional();
      }
      
      shape[field] = fieldSchema;
    }
    
    return z.object(shape);
  }, [dataSchema]);

  const form = useForm({
    resolver: zodResolver(zodSchema),
    defaultValues,
  });

  // Determine field order
  const fieldOrder = uiSchema.order || Object.keys(dataSchema.properties);

  return (
    <form onSubmit={form.handleSubmit(onSubmit)} className="space-y-4">
      <div className="grid grid-cols-2 gap-4">
        {fieldOrder.map(fieldName => {
          const schema = dataSchema.properties[fieldName];
          const uiHints = uiSchema.fields?.[fieldName] || {};
          
          if (uiHints.hidden) return null;
          
          return (
            <div 
              key={fieldName}
              className={uiHints.colSpan === 2 ? 'col-span-2' : 'col-span-1'}
            >
              <FieldRenderer
                name={fieldName}
                schema={schema}
                uiHints={uiHints}
                control={form.control}
                readOnly={readOnly || uiHints.disabled}
              />
            </div>
          );
        })}
      </div>
      
      {!readOnly && (
        <div className="flex justify-end gap-2">
          <Button type="button" variant="outline" onClick={() => form.reset()}>
            Reset
          </Button>
          <Button type="submit" disabled={form.formState.isSubmitting}>
            {form.formState.isSubmitting ? 'Saving...' : 'Save'}
          </Button>
        </div>
      )}
    </form>
  );
}

function FieldRenderer({
  name,
  schema,
  uiHints,
  control,
  readOnly,
}: {
  name: string;
  schema: FieldSchema;
  uiHints: UIFieldHints;
  control: any;
  readOnly: boolean;
}) {
  const widget = uiHints.widget || inferWidget(schema);
  const label = schema.title || formatLabel(name);
  
  return (
    <div className="space-y-1">
      <Label htmlFor={name}>{label}</Label>
      
      <Controller
        name={name}
        control={control}
        render={({ field, fieldState }) => (
          <>
            {widget === 'select' && schema.enum ? (
              <Select
                value={field.value}
                onValueChange={field.onChange}
                disabled={readOnly}
              >
                <SelectTrigger>
                  <SelectValue placeholder={uiHints.placeholder || `Select ${label}`} />
                </SelectTrigger>
                <SelectContent>
                  {schema.enum.map(option => (
                    <SelectItem key={option} value={option}>
                      {option}
                    </SelectItem>
                  ))}
                </SelectContent>
              </Select>
            ) : widget === 'textarea' ? (
              <Textarea
                {...field}
                placeholder={uiHints.placeholder}
                disabled={readOnly}
              />
            ) : widget === 'currency' ? (
              <Input
                {...field}
                type="number"
                step="0.01"
                placeholder={uiHints.placeholder || '0.00'}
                disabled={readOnly}
              />
            ) : widget === 'date' ? (
              <Input
                {...field}
                type="date"
                disabled={readOnly}
              />
            ) : (
              <Input
                {...field}
                type={schema.type === 'number' ? 'number' : 'text'}
                placeholder={uiHints.placeholder}
                disabled={readOnly}
              />
            )}
            
            {fieldState.error && (
              <p className="text-sm text-red-500">{fieldState.error.message}</p>
            )}
          </>
        )}
      />
      
      {schema.description && (
        <p className="text-sm text-gray-500">{schema.description}</p>
      )}
    </div>
  );
}

function inferWidget(schema: FieldSchema): string {
  if (schema.enum) return 'select';
  if (schema.format === 'date' || schema.format === 'datetime') return 'date';
  if (schema.format === 'currency') return 'currency';
  if (schema.type === 'boolean') return 'checkbox';
  return 'text';
}

function formatLabel(name: string): string {
  return name
    .replace(/([A-Z])/g, ' $1')
    .replace(/^./, str => str.toUpperCase())
    .trim();
}
```

---

## Part 6: Stocktake Template

### 6.1 Entity Schema

```typescript
// Stocktake data schema
const stocktakeDataSchema = {
  type: 'object',
  properties: {
    warehouse: { type: 'string', title: 'Warehouse' },
    countDate: { type: 'string', format: 'date', title: 'Count Date' },
    assignee: { type: 'string', title: 'Assigned To' },
    notes: { type: 'string', title: 'Notes' },
  },
  required: ['warehouse', 'countDate'],
};

const stocktakeUISchema = {
  order: ['warehouse', 'countDate', 'assignee', 'notes'],
  fields: {
    warehouse: { widget: 'select' },
    notes: { widget: 'textarea', colSpan: 2 },
  },
};

// Stocktake item schema
const stocktakeItemDataSchema = {
  type: 'object',
  properties: {
    sku: { type: 'string', title: 'SKU' },
    productName: { type: 'string', title: 'Product Name' },
    expectedQty: { type: 'number', title: 'Expected Quantity', minimum: 0 },
    countedQty: { type: 'number', title: 'Counted Quantity', minimum: 0 },
    notes: { type: 'string', title: 'Notes' },
  },
  required: ['sku', 'countedQty'],
};
```

### 6.2 Workflow Definition

```typescript
const stocktakeWorkflow = {
  id: 'stocktake-workflow',
  name: 'Stocktake Lifecycle',
  entityType: 'stocktake',
  initialState: 'draft',
  states: [
    {
      name: 'draft',
      type: 'initial',
      allowedActions: ['edit', 'delete', 'start'],
    },
    {
      name: 'counting',
      type: 'intermediate',
      allowedActions: ['edit', 'addItem', 'submit'],
    },
    {
      name: 'submitted',
      type: 'intermediate',
      allowedActions: ['approve', 'reject'],
      timeout: {
        duration: 'P3D',  // 3 days
        action: { type: 'notify', params: { role: 'manager' } },
      },
    },
    {
      name: 'approved',
      type: 'final',
      allowedActions: ['view'],
    },
    {
      name: 'rejected',
      type: 'intermediate',
      allowedActions: ['edit', 'resubmit'],
    },
  ],
  transitions: [
    { from: 'draft', to: 'counting', event: 'start', roles: ['counter', 'manager'] },
    { from: 'counting', to: 'submitted', event: 'submit', roles: ['counter'] },
    { from: 'submitted', to: 'approved', event: 'approve', roles: ['manager', 'cfo'] },
    { from: 'submitted', to: 'rejected', event: 'reject', roles: ['manager', 'cfo'] },
    { from: 'rejected', to: 'submitted', event: 'resubmit', roles: ['counter'] },
  ],
};
```

### 6.3 Default Rules

```typescript
const stocktakeDefaultRules = [
  {
    id: 'rule-high-value-approval',
    name: 'High Value CFO Approval',
    triggerEvent: 'stocktake.submit',
    conditions: [
      { field: 'entity.data.totalValue', operator: 'gt', value: 50000 },
    ],
    actions: [
      { type: 'requireApproval', params: { role: 'cfo' } },
      { type: 'notify', params: { role: 'cfo', message: 'High value stocktake requires your approval' } },
    ],
    priority: 100,
    enabled: true,
  },
  {
    id: 'rule-variance-flag',
    name: 'High Variance Flag',
    triggerEvent: 'stocktakeItem.update',
    conditions: [
      {
        field: 'computed.variancePercent',
        operator: 'gt',
        value: 0.20,  // 20%
      },
    ],
    actions: [
      { type: 'flag', params: { reason: 'High variance detected' } },
      { type: 'requireField', params: { field: 'notes', message: 'Please explain the variance' } },
    ],
    priority: 50,
    enabled: true,
  },
  {
    id: 'rule-weekend-warning',
    name: 'Weekend Submission Warning',
    triggerEvent: 'stocktake.submit',
    conditions: [
      { field: 'computed.isWeekend', operator: 'eq', value: true },
    ],
    actions: [
      { type: 'warn', params: { message: 'Submitting on weekend - approval may be delayed' } },
    ],
    priority: 10,
    enabled: true,
  },
];
```

---

## Part 7: Delivery Plan

### Phase 1: Foundation (Weeks 1-4)

**Goal**: Basic CRUD with activity logging and simple rules.

| Week | Tasks |
|------|-------|
| 1 | Project setup, database schema, auth |
| 2 | Entity CRUD API, activity logging |
| 3 | Rules engine, command handler pattern |
| 4 | Schema-driven forms, stocktake template |

**Deliverables**:
- Working stocktake CRUD
- Activity logged for all actions
- Basic rules evaluated on submit
- Form rendered from schema

### Phase 2: Workflows (Weeks 5-6)

**Goal**: Temporal integration for workflow execution.

| Week | Tasks |
|------|-------|
| 5 | Temporal setup, workflow definition format |
| 6 | Stocktake workflow, transitions, timeouts |

**Deliverables**:
- Workflow state management
- Role-based transition permissions
- Timeout notifications

### Phase 3: AI Foundation (Weeks 7-10)

**Goal**: Pattern detection and anomaly alerts.

| Week | Tasks |
|------|-------|
| 7 | AI analysis job infrastructure |
| 8 | Pattern detection implementation |
| 9 | Anomaly detection, baseline building |
| 10 | Suggestions UI, alert dashboard |

**Deliverables**:
- Nightly pattern analysis
- Real-time anomaly detection
- Admin suggestions queue
- Alert notifications

### Phase 4: AI Copilot (Weeks 11-14)

**Goal**: Natural language assistance for data entry.

| Week | Tasks |
|------|-------|
| 11 | Chat interface, NL data entry |
| 12 | Smart search, query assistance |
| 13 | Report generation from NL |
| 14 | Polish, testing, documentation |

**Deliverables**:
- Chat-based data entry
- Natural language search
- AI-generated reports
- Production-ready v1

---

## Part 8: Tech Stack

| Layer | Technology | Rationale |
|-------|------------|-----------|
| Frontend | Next.js 14+ (App Router) | SSR, API routes, modern React |
| UI | shadcn/ui + Tailwind | Accessible, customizable |
| Forms | React Hook Form + Zod | Type-safe validation |
| State | Zustand | Simple, performant |
| Backend | Next.js API Routes | Integrated, fast |
| Database | PostgreSQL (Neon or Supabase) | Proven, JSONB support |
| Workflow | Temporal | Durable execution |
| AI | Claude API (direct) | Best reasoning |
| Auth | Clerk or Supabase Auth | Fast setup, secure |
| Hosting | Vercel + Fly.io (Temporal) | Edge + containers |

### What We're NOT Using

| Technology | Reason |
|------------|--------|
| LangChain | Adds complexity without value for our use case |
| ClickHouse | Premature optimization |
| React Flow | Phase 2 if users demand visual editing |
| Custom Workflow Engine | Temporal is proven, maintained |
| Event Sourcing | Overkill; activity log is sufficient |

---

## Part 9: Success Metrics

### Phase 1-2 Success (Foundation)
- [ ] 5+ users completing stocktakes daily
- [ ] <500ms response time for all operations
- [ ] 100% of actions logged
- [ ] Zero data loss

### Phase 3 Success (AI Foundation)
- [ ] Pattern detection finds 3+ valid patterns per tenant
- [ ] Anomaly detection has <20% false positive rate
- [ ] 50%+ of suggestions are accepted
- [ ] Alerts prevent at least 1 real issue

### Phase 4 Success (AI Copilot)
- [ ] NL data entry used for 30%+ of entries
- [ ] AI reduces data entry time by 25%+
- [ ] Users rate AI assistance 4+/5
- [ ] Support tickets reduce 20%+

---

## Part 10: Risk Mitigation

| Risk | Mitigation |
|------|------------|
| AI suggestions low quality | Start with high-confidence only (>0.8), iterate |
| Users don't trust AI | Show evidence, make review mandatory |
| Pattern detection too slow | Run nightly, cache results |
| Anomaly false positives | Allow users to tune sensitivity |
| Scope creep to "platform" | Explicit phase gates, ship stocktake first |

---

## Conclusion

This plan prioritizes:

1. **Shipping fast** - 14 weeks to production v1
2. **AI differentiation** - Learn patterns, detect anomalies, suggest improvements
3. **Simplicity** - No DSL compiler, no event sourcing, no custom workflow engine
4. **User value** - AI that gets smarter over time, not AI that builds from scratch

The original vision (generic platform, SKILL.md compiler, AI as author) is technically achievable but would take 2-3 years and carries high risk. This revised plan delivers 80% of the user value in 20% of the time.

**Start here. Add complexity when users demand it.**
