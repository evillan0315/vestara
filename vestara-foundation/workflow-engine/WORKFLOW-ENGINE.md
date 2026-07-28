---
id: "FND-009"
title: "Workflow Engine — Automation Architecture"
owner: "@ai-engineer"
status: "draft"
blueprint-ref: "04-platform/automation/"
foundation-version: "1.0.0"
---

# Workflow Engine
## Trigger → Condition → Plan → Execute → Validate — The Automation Backbone

> **The Workflow Engine powers all automation in Vestara. Workflows are user-defined or AI-generated sequences of steps that respond to triggers, execute through agents and tools, and produce validated outcomes.**

---

## Workflow Model

```typescript
interface Workflow {
  id: string;                    // UUID v7
  name: string;
  description: string;
  version: string;
  
  trigger: WorkflowTrigger;
  steps: WorkflowStep[];
  
  config: WorkflowConfig;
  status: 'enabled' | 'disabled' | 'running' | 'completed' | 'failed';
  
  created_at: string;
  updated_at: string;
}

interface WorkflowTrigger {
  type: 'event' | 'schedule' | 'manual' | 'condition';
  
  // Event trigger: fires on specific event
  eventPattern?: string;         // 'project:created', 'memory:*'
  
  // Schedule trigger: fires on cron schedule
  schedule?: string;             // '0 9 * * 1' (Mondays 9AM)
  
  // Condition trigger: fires when condition is met
  condition?: {
    type: 'memory' | 'knowledge' | 'project' | 'system';
    query: string;               // Evaluation query
    frequency: number;           // Check interval in minutes
  };
}

interface WorkflowConfig {
  maxDuration: number;           // Max execution time in ms
  maxRetries: number;            // Retry on step failure
  concurrency: number;           // Max parallel steps
  notifications: boolean;        // Notify on completion/failure
  timeout: number;               // Per-step timeout in ms
}
```

---

## Workflow Steps

```typescript
interface WorkflowStep {
  id: string;
  name: string;
  type: StepType;
  
  // Condition
  condition?: string;            // Expression to evaluate before executing
  
  // Input/output
  input: Record<string, unknown>;
  outputMapping?: Record<string, string>;  // Map output to workflow variables
  
  // Error handling
  onError: 'fail' | 'skip' | 'retry' | 'notify';
  maxRetries?: number;
  
  // Next steps
  next?: string[];               // Next step IDs (for branching)
  onSuccess?: string;
  onFailure?: string;
}

type StepType =
  | 'agent.execute'             // Execute an AI agent
  | 'tool.invoke'               // Invoke a specific tool
  | 'api.call'                  // Call an external API
  | 'condition.check'           // Evaluate a condition
  | 'transform.data'            // Transform data between steps
  | 'notification.send'        // Send a notification
  | 'workflow.subflow'          // Execute another workflow
  | 'human.approval'           // Wait for human approval
  | 'delay'                     // Wait for a duration
  | 'loop'                      // Iterate over items
  | 'parallel'                  // Execute steps in parallel
  | 'event.emit';               // Emit an event
```

---

## Workflow Lifecycle

```
Defined → Enabled → Triggered → Running → Completed
                                      ↓
                                    Failed
                                      ↓
                                    Paused (manual resume)
```

### State Machine

| State | Description | Transitions |
|-------|-------------|-------------|
| `defined` | Workflow created but not active | → enabled |
| `enabled` | Active, listening for triggers | → disabled, triggered |
| `triggered` | Trigger condition met, execution starting | → running |
| `running` | Steps executing | → completed, failed, paused |
| `paused` | Waiting for human approval | → running |
| `completed` | All steps executed successfully | — |
| `failed` | Unrecoverable error | → enabled (retry) |
| `disabled` | Inactive, not listening | → enabled |

---

## Workflow Execution Engine

```typescript
interface WorkflowEngine {
  /** Define a new workflow */
  define(workflow: WorkflowDefinition): Promise<Workflow>;
  
  /** Enable a workflow (start listening for triggers) */
  enable(workflowId: string): Promise<void>;
  
  /** Disable a workflow */
  disable(workflowId: string): Promise<void>;
  
  /** Execute a workflow manually */
  execute(workflowId: string, context?: Record<string, unknown>): Promise<ExecutionResult>;
  
  /** Get execution status */
  getExecutionStatus(executionId: string): Promise<ExecutionStatus>;
  
  /** Cancel a running execution */
  cancel(executionId: string): Promise<void>;
  
  /** List workflow executions */
  listExecutions(workflowId: string): Promise<ExecutionSummary[]>;
}

interface ExecutionResult {
  executionId: string;
  workflowId: string;
  status: 'running' | 'completed' | 'failed' | 'cancelled';
  steps: StepResult[];
  startedAt: string;
  completedAt?: string;
  duration: number;
  error?: ExecutionError;
}

interface StepResult {
  stepId: string;
  status: 'pending' | 'running' | 'completed' | 'failed' | 'skipped';
  input: Record<string, unknown>;
  output?: Record<string, unknown>;
  startedAt?: string;
  completedAt?: string;
  duration?: number;
  error?: string;
}
```

---

## Trigger Registry

```yaml
triggers:
  - id: "event"
    name: "Event Trigger"
    description: "Fires when a platform event matches a pattern"
    parameters: { eventPattern: string }
  
  - id: "schedule"
    name: "Schedule Trigger"
    description: "Fires on a cron schedule"
    parameters: { cron: string; timezone?: string }
  
  - id: "memory"
    name: "Memory Condition Trigger"
    description: "Fires when a memory condition is met"
    parameters: { query: string; frequency: number }
  
  - id: "knowledge"
    name: "Knowledge Condition Trigger"
    description: "Fires when a knowledge condition is met"
    parameters: { query: string; frequency: number }
  
  - id: "manual"
    name: "Manual Trigger"
    description: "Fired manually by the user"
    parameters: {}
```

---

## Example Workflow

```yaml
name: "Daily Project Summary"
description: "Every morning, summarize yesterday's project activity"
trigger:
  type: "schedule"
  schedule: "0 8 * * 1-5"    # Weekdays 8AM
steps:
  - id: "collect-activity"
    name: "Collect Yesterday's Activity"
    type: "api.call"
    input:
      endpoint: "GET /api/v1/projects/:id/activity"
      params:
        after: "{{ yesterday }}"
    outputMapping:
      activities: "$.data"
  
  - id: "generate-summary"
    name: "Generate Summary"
    type: "agent.execute"
    input:
      agent: "conversation"
      task: "Summarize these project activities: {{ activities }}"
  
  - id: "send-notification"
    name: "Send Summary"
    type: "notification.send"
    input:
      title: "Daily Project Summary"
      body: "{{ generate-summary.output }}"
      priority: "normal"
```

---

*The Workflow Engine transforms Vestara from a reactive tool into a proactive platform. Workflows automate the routine so users can focus on the exceptional.*
