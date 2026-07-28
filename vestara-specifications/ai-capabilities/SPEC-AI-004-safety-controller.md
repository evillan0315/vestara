---
id: "SPEC-AI-004"
title: "Safety Controller — Autonomous Agent Safety Architecture"
owner: "@vestara-core"
status: "draft"
blueprint-ref: "00-governance/01-ai-constitution.md"
version: "1.0.0"
---

# SPEC-AI-004: Safety Controller
## Preventing Runaway Repair Loops in Autonomous Agents

---

## Purpose

Define the safety architecture that prevents autonomous agents from making repositories worse while trying to fix them. The Safety Controller is a separate system that decides when an agent may continue, must retry, should rollback, or must escalate to humans.

## Business Value

The difference between a coding assistant and an autonomous engineering system is not code generation quality. It is the ability to recognize when to stop. An agent that cannot stop is an agent that will eventually cause damage. The Safety Controller is what makes autonomous development reliable rather than merely autonomous.

---

## The Core Problem: Runaway Repair Loop

```
Build fails
    ↓
Agent changes code
    ↓
Build fails
    ↓
Agent changes more code
    ↓
Build fails
    ↓
Agent changes even more code
    ↓
Repository becomes worse
```

Most current coding agents have no mechanism to recognize they are making the situation worse. They optimize indefinitely toward "make the build pass." A human engineer does not work this way. After several failed attempts, a senior engineer says:

> "I don't understand the root cause. I'm stopping here."

That is a form of intelligence. The Safety Controller formalizes it.

---

## Architecture

```
            Task
              │
              ▼
    Implementation Agent
              │
              ▼
       Verification
              │
      ┌───────┴────────┐
      │                │
  Success          Failure
      │                │
      ▼                ▼
  Complete      Safety Controller
                       │
            ┌──────────┼──────────┬──────────┐
            │          │          │          │
        Continue    Retry    Rollback   Escalate
```

**Critical Design Decision**: The implementation agent **never** decides to retry. The Safety Controller decides.

---

## Safety Controller Components

### 1. Retry Budget

Every task has a maximum number of retry attempts.

| Task Complexity | Max Retries | Rationale |
|-----------------|-------------|-----------|
| Simple (one file) | 2 | Low complexity, quick iteration |
| Medium (few files) | 3 | Moderate complexity |
| Complex (many files) | 5 | Higher complexity, more variables |
| Critical (production) | 1 | Human should be involved immediately |

```
Attempt 1: Build failed (147 errors)
Attempt 2: Build failed (102 errors)
Attempt 3: Build failed (138 errors)

STOP — Retry budget exhausted.

Repository state restored to checkpoint.
Evidence package attached.
Escalating for human review.
```

**Rules**:
- Retry budget is decremented on each attempt
- Budget cannot be reset or extended
- When budget is exhausted, task terminates immediately
- Termination includes rollback to last checkpoint

### 2. Progress Measurement

The controller asks one question on each attempt: **"Am I getting closer?"**

```
Attempt 1: 147 TypeScript errors
Attempt 2: 102 TypeScript errors

Progress: -45 errors (30.6% improvement)
Decision: CONTINUE
```

versus

```
Attempt 1: 147 TypeScript errors
Attempt 2: 181 TypeScript errors

Progress: +34 errors (23.1% regression)
Decision: STOP — Repository became worse
```

**Progress Metrics**:

| Metric | Calculation | Threshold |
|--------|-------------|-----------|
| Error count delta | errors_after - errors_before | Must be ≤ 0 |
| Test pass delta | tests_after - tests_before | Must be ≥ 0 |
| Lint error delta | lint_after - lint_before | Must be ≤ 0 |
| Composite score | weighted average | Must improve ≥ 5% per attempt |

**Rules**:
- Progress must be non-negative to continue
- If progress is zero for 2 consecutive attempts, stop
- If progress is negative, stop immediately

### 3. Oscillation Detection

Sometimes agents bounce between two solutions:

```
State A → State B → State A → State B → State A
```

This is an infinite loop.

**Detection Method**:
```typescript
interface StateRecord {
  hash: string;        // SHA-256 of repository state
  timestamp: string;
  attempt: number;
}

class OscillationDetector {
  private history: StateRecord[] = [];

  record(state: string): void {
    this.history.push({
      hash: sha256(state),
      timestamp: new Date().toISOString(),
      attempt: this.history.length + 1,
    });
  }

  detectOscillation(): boolean {
    const hashes = this.history.map(r => r.hash);
    const lastFour = hashes.slice(-4);

    // Check for A-B-A-B pattern
    if (lastFour.length >= 4) {
      if (lastFour[0] === lastFour[2] && lastFour[1] === lastFour[3]) {
        return true;
      }
    }

    // Check for any repeated state
    const uniqueHashes = new Set(hashes);
    if (uniqueHashes.size < hashes.length * 0.5) {
      return true; // More than 50% repeated states
    }

    return false;
  }
}
```

**Rules**:
- Record state hash after each attempt
- If oscillation detected, stop immediately
- Report the oscillation pattern in evidence package

### 4. Confidence Threshold

The agent continuously estimates its confidence that the next attempt will succeed.

```
Attempt 1: Confidence 0.91
Attempt 2: Confidence 0.82
Attempt 3: Confidence 0.74
Attempt 4: Confidence 0.51
Attempt 5: Confidence 0.29

STOP — Confidence below threshold (0.50)
```

**Confidence Calculation**:

| Factor | Weight | Impact |
|--------|--------|--------|
| Progress trend | 30% | Improving = high, regressing = low |
| Error categorization | 25% | Known errors = high, unknown = low |
| Strategy novelty | 25% | New approach = high, repeated = low |
| Historical success | 20% | Similar tasks succeeded = high |

**Rules**:
- Confidence must be ≥ 0.50 to continue
- If confidence drops below 0.50, stop
- If confidence drops by > 20% in one attempt, stop

### 5. Failed Strategy Memory

Agent memory must store failures, not just successes.

```typescript
interface FailedStrategy {
  task: string;
  strategy: string;
  attempt: number;
  result: 'failed';
  reason: string;
  errorPattern: string;
  timestamp: string;
}

// Memory store
const failedStrategies: FailedStrategy[] = [
  {
    task: "Fix Dashboard build error",
    strategy: "Added missing import",
    attempt: 1,
    result: 'failed',
    reason: "Import was not root cause",
    errorPattern: "TS2305: Module has no exported member",
    timestamp: "2026-07-27T10:30:00Z",
  }
];
```

**Rules**:
- Before attempting a strategy, check failed strategies for same task/pattern
- If identical strategy already failed, skip it
- If similar pattern failed, reduce confidence

### 6. Repository Checkpoints

Before every major change, create a checkpoint.

```
Checkpoint Flow:

1. git stash push -m "checkpoint-[task-id]-[attempt]"
   ↓
2. Apply change
   ↓
3. Verify
   ↓
   ├── Success → Continue
   └── Failure → git stash pop
               → Return to checkpoint state
```

**Checkpoint Rules**:
- Create checkpoint before each attempt
- Checkpoint includes all uncommitted changes
- On failure, restore to last checkpoint
- Never lose more than one attempt's worth of work
- Checkpoints are cleaned up on success

```typescript
interface Checkpoint {
  id: string;
  taskId: string;
  attempt: number;
  stashRef: string;
  timestamp: string;
  repositoryState: {
    branch: string;
    commitHash: string;
    dirty: boolean;
  };
}

class CheckpointManager {
  private checkpoints: Checkpoint[] = [];

  async create(taskId: string, attempt: number): Promise<Checkpoint> {
    const stashRef = await this.gitStash(`checkpoint-${taskId}-${attempt}`);
    const checkpoint: Checkpoint = {
      id: `cp_${Date.now()}`,
      taskId,
      attempt,
      stashRef,
      timestamp: new Date().toISOString(),
      repositoryState: await this.getRepositoryState(),
    };
    this.checkpoints.push(checkpoint);
    return checkpoint;
  }

  async restore(checkpointId: string): Promise<void> {
    const checkpoint = this.checkpoints.find(c => c.id === checkpointId);
    if (!checkpoint) throw new Error('Checkpoint not found');
    await this.gitStashPop(checkpoint.stashRef);
  }
}
```

### 7. Investigation vs Repair Separation

Most agents combine observation, analysis, and repair into one loop. This is dangerous. Separate them completely.

```
Investigation Phase:
  - Read error messages
  - Trace to source
  - Identify root cause
  - Estimate confidence
  - IF confidence < 90% → STOP, do not modify code

Implementation Phase:
  - Apply fix from investigation
  - Verify fix
  - IF verification fails → Safety Controller decides

Verification Phase:
  - Run full test suite
  - Run build
  - Run lint
  - Compare before/after
  - Generate evidence package
```

**Rules**:
- Investigation must complete before implementation begins
- Investigation confidence must be ≥ 90% to proceed
- Implementation must use only the approved fix from investigation
- Verification must be complete before task is marked done
- No code changes during investigation phase

---

## Engineering Safety Contract

Every implementation agent must obey these rules:

```
AI Safety Contract

Rule 1: Never modify code without understanding the failure.
  - Investigation must precede implementation
  - Root cause must be identified
  - Confidence must be ≥ 90%

Rule 2: Maximum three repair attempts.
  - Retry budget enforced by Safety Controller
  - Cannot be reset or extended

Rule 3: Never repeat an identical strategy.
  - Failed strategy memory consulted before each attempt
  - Identical strategies blocked

Rule 4: Always checkpoint before modification.
  - Repository state saved before each attempt
  - Rollback possible at any time

Rule 5: Rollback on regression.
  - If repository state worsens, restore checkpoint
  - Do not attempt to "fix forward"

Rule 6: Verification is mandatory.
  - No task complete without verification
  - Verification evidence required

Rule 7: Escalate rather than guess.
  - When uncertain, ask for human guidance
  - Confidence below threshold triggers escalation

Rule 8: Repository health is more important than task completion.
  - A broken repository helps no one
  - Safe failure is better than unsafe success
```

---

## Safety Controller Interface

```typescript
interface SafetyController {
  // Core decision
  decide(outcome: TaskOutcome): SafetyDecision;

  // State tracking
  recordAttempt(attempt: AttemptRecord): void;
  getState(): SafetyState;

  // Checkpoints
  createCheckpoint(taskId: string): Promise<Checkpoint>;
  restoreCheckpoint(checkpointId: string): Promise<void>;

  // Memory
  recordFailedStrategy(strategy: FailedStrategy): void;
  getFailedStrategies(task: string): FailedStrategy[];
}

interface TaskOutcome {
  success: boolean;
  errorCount: number;
  testPassCount: number;
  lintErrorCount: number;
  confidence: number;
  strategy: string;
  stateHash: string;
}

type SafetyDecision =
  | { type: 'continue' }
  | { type: 'retry'; reason: string }
  | { type: 'rollback'; checkpointId: string; reason: string }
  | { type: 'escalate'; reason: string; evidence: EvidencePackage };

interface SafetyState {
  currentTask: string;
  attemptCount: number;
  retryBudget: number;
  confidence: number;
  progress: number[];
  oscillationDetected: boolean;
  checkpoints: Checkpoint[];
  failedStrategies: FailedStrategy[];
}
```

---

## Evidence Package (Extended)

Every Safety Controller decision must include:

```
Safety Controller Decision — Task [TASK-ID]

Decision: ROLLBACK
Reason: Repository state worsened (147 → 181 TypeScript errors)

Attempt History:
  Attempt 1: 147 errors → 162 errors (+15, regression)
  Attempt 2: 147 errors → 181 errors (+34, regression)

Progress: Negative on both attempts
Oscillation: Not detected
Confidence: 0.34 (below 0.50 threshold)
Retry Budget: 0 remaining

Checkpoint Restored: cp_1722076200000
Repository State: Clean, build passes

Failed Strategy Recorded:
  Task: Fix TypeScript errors
  Strategy: Add missing type annotations
  Reason: Did not address root cause

Recommendation: Investigate root cause before retrying.
Root cause may be in dependency chain, not source code.
```

---

## Integration with VECS

The Safety Controller is not separate from VECS — it is part of it.

| VECS Requirement | Safety Controller Role |
|------------------|----------------------|
| Verification compliance | Enforced by Safety Controller |
| Stop condition compliance | Implemented by Safety Controller |
| S4 failure prevention | Checkpoints and rollback |
| Decision quality | Progress measurement and confidence |
| Evidence packages | Generated by Safety Controller |

---

## Related Specifications

- SPEC-AI-001: VECS — Engineering Competency Standard
- SPEC-AI-002: Operational Reliability — Durability under repeated use
- SPEC-AI-003: Agent Certification — Certification process and criteria

---

*Safety Controller — The difference between autonomous and trustworthy.*
