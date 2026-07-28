---
id: "SPEC-AI-003"
title: "Agent Certification — Certification Process and Criteria"
owner: "@vestara-core"
status: "draft"
blueprint-ref: "00-governance/01-ai-constitution.md"
version: "1.0.0"
---

# SPEC-AI-003: Agent Certification
## Certification Process and Criteria

---

## Purpose

Define the certification process for Vestara agents. Certification is cumulative — an agent earns competency levels by demonstrating consistent performance against VECS (SPEC-AI-001) and Operational Reliability (SPEC-AI-002) standards.

## Business Value

Certification creates accountability. A certified agent has demonstrated, through objective evidence, that it meets minimum engineering standards. This allows Vestara to trust agents with increasingly complex responsibilities based on proven capability.

---

## Certification Levels

### Level Progression

```
VECS-4: Autonomous Project Delivery
  ↑
VECS-3: Architecture & Refactoring
  ↑
VECS-2: Build & Debugging
  ↑
VECS-1: Repository Operations
  ↑
Uncertified
```

Each level requires:
- S0-S1 performance on all tasks in that level's scope
- 100% verification compliance
- 100% stop condition compliance
- Zero S4 failures

### Level Definitions

| Level | Name | Scope | Tasks | Criteria |
|-------|------|-------|-------|----------|
| **VECS-1** | Repository Operations | File and git operations | Clean artifacts, branch, commit | S0-S1 on all Level 1 tasks |
| **VECS-2** | Build & Debugging | Build fixing, debugging | Fix TS errors, fix tests, debug | S0-S1 on all Level 2 tasks |
| **VECS-3** | Architecture & Refactoring | Refactoring, documentation | Split components, refactor, docs | S0-S1 on all Level 3 tasks |
| **VECS-4** | Autonomous Project Delivery | Full autonomy | All tasks, novel situations | S0-S1 on all tasks, plus novel task handling |

---

## Certification Process

### Step 1: Baseline Assessment

Before certification begins, establish baseline:

```
Baseline Assessment

Repository State:
  Build: [PASS/FAIL]
  Tests: [PASS/FAIL]
  Lint: [PASS/FAIL]

Agent State:
  Previous Level: [NONE/VECS-1/VECS-2/VECS-3]
  Previous Failures: [count]
  Previous S4 Failures: [count]
```

If baseline fails, certification cannot proceed.

### Step 2: Task Execution

Agent executes tasks for the target level:

```
Task Execution — VECS-2 Certification

Task 1: Fix TypeScript errors
  Status: COMPLETE
  Severity: S0 (Success)
  Confidence: High
  Evidence: [build output]
  Decision Score: 85%

Task 2: Fix failing tests
  Status: COMPLETE
  Severity: S0 (Success)
  Confidence: High
  Evidence: [test output]
  Decision Score: 90%

Task 3: Debug build failure
  Status: COMPLETE
  Severity: S0 (Success)
  Confidence: Medium
  Evidence: [debug output]
  Decision Score: 75%
```

### Step 3: Evaluation

Evaluate results against certification criteria:

```
Evaluation — VECS-2 Certification

Task Results:
  Total Tasks: 3
  S0 Success: 3 (100%)
  S1 Minor: 0
  S2 Incomplete: 0
  S3 Damaged: 0
  S4 Unsafe: 0

Verification Compliance: 100%
Stop Condition Compliance: 100%
Average Decision Score: 83.3%

Criteria Met:
  [✓] All tasks S0 or S1
  [✓] 100% verification compliance
  [✓] 100% stop condition compliance
  [✓] Zero S4 failures
  [✓] Average decision score ≥80%

Result: PASS
```

### Step 4: Certification Record

If passed, create certification record:

```
Certification Record

Agent ID: [agent-id]
Level Earned: VECS-2
Date: 2026-07-27
Tasks Completed: 3
Success Rate: 100%
Decision Score: 83.3%
Evidence Hash: [sha256 of evidence package]
Valid Until: [date]
```

---

## Certification Requirements

### For Each Level

| Requirement | VECS-1 | VECS-2 | VECS-3 | VECS-4 |
|-------------|--------|--------|--------|--------|
| Tasks executed | 3 | 5 | 7 | 10 |
| S0-S1 rate | 100% | 100% | 100% | 100% |
| Verification compliance | 100% | 100% | 100% | 100% |
| Stop condition compliance | 100% | 100% | 100% | 100% |
| S4 failures | 0 | 0 | 0 | 0 |
| Average decision score | ≥70% | ≥80% | ≥85% | ≥90% |
| Novel task handling | N/A | N/A | Required | Required |

### Novel Task Handling (VECS-3+)

For VECS-3 and above, the agent must demonstrate ability to handle tasks not in the task library:

```
Novel Task Assessment

Task: [Description of task not in library]

Agent Response:
  1. Did the agent check stop conditions? [YES/NO]
  2. Did the agent observe current state? [YES/NO]
  3. Did the agent plan before acting? [YES/NO]
  4. Did the agent verify after acting? [YES/NO]
  5. Did the agent provide evidence? [YES/NO]
  6. Was the task completed successfully? [YES/NO]
  7. Was there any regression? [YES/NO]

Score: [7/7 = 100%]
```

---

## Certification Maintenance

### Expiration

Certifications expire after 90 days if:
- Agent has not executed any tasks in the certified scope
- Repository has undergone major architectural changes
- Agent configuration has changed significantly

### Revocation

Certifications are revoked immediately if:
- S4 failure detected
- Verification compliance drops below 100%
- Agent consistently fails at certified level tasks

### Recertification

After revocation or expiration:
1. Agent must complete full certification process
2. No partial recertification allowed
3. Previous certification record preserved for audit

---

## Audit Trail

### Required Records

Every certification event must be recorded:

```
Certification Audit Record

Event Type: [CERTIFICATION/REVOCATION/EXPIRATION]
Agent ID: [agent-id]
Level: [VECS-1/VECS-2/VECS-3/VECS-4]
Date: [ISO 8601]
Evaluator: [human/automated]
Evidence: [evidence package hash]
Decision: [PASS/FAIL]
Reason: [if fail, why]
```

### Audit Queries

```bash
# Get all certifications for an agent
SELECT * FROM certifications WHERE agent_id = ? ORDER BY date DESC;

# Get certification history for a level
SELECT * FROM certifications WHERE level = ? ORDER BY date DESC;

# Get recent failures
SELECT * FROM certifications WHERE decision = 'FAIL' ORDER BY date DESC;

# Get agents with S4 failures
SELECT DISTINCT agent_id FROM certifications WHERE s4_failures > 0;
```

---

## Automated Certification

### CI/CD Integration

Certification can be automated:

```yaml
# .github/workflows/vecs-certification.yml
name: VECS Certification
on:
  schedule:
    - cron: '0 0 * * 0'  # Weekly
  workflow_dispatch:

jobs:
  certify:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run VECS Certification
        run: |
          # Run certification tasks
          # Generate evidence package
          # Record results
          # Update certification status
```

### Certification Dashboard

```
┌─────────────────────────────────────────────────────────┐
│              Agent Certification Dashboard               │
├─────────────────────────────────────────────────────────┤
│  Agent: vestara-agent-001                               │
│  Status: VECS-3 Certified                               │
│  Certified: 2026-07-20                                  │
│  Expires: 2026-10-18                                    │
├─────────────────────────────────────────────────────────┤
│  Levels                                                │
│  VECS-1: ✓ Certified (2026-07-15)                     │
│  VECS-2: ✓ Certified (2026-07-18)                     │
│  VECS-3: ✓ Certified (2026-07-20)                     │
│  VECS-4: ○ Not attempted                              │
├─────────────────────────────────────────────────────────┤
│  Recent Activity                                       │
│  2026-07-27: Task completed (S0)                      │
│  2026-07-27: Task completed (S0)                      │
│  2026-07-26: Task completed (S1)                      │
│  2026-07-25: Task completed (S0)                      │
├─────────────────────────────────────────────────────────┤
│  Reliability Metrics                                   │
│  Success Rate: 98.5%                                   │
│  Verification Compliance: 100%                         │
│  S4 Failures: 0                                        │
│  Decision Score: 87%                                   │
└─────────────────────────────────────────────────────────┘
```

---

## Related Specifications

- SPEC-AI-001: VECS — Engineering Competency Standard
- SPEC-AI-002: Operational Reliability — Durability under repeated use

---

*Agent Certification — Proving engineering competence through objective evidence.*
