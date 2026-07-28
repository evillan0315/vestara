---
id: "FND-010"
title: "Mission Engine — Long-Living Business Outcomes Across Sessions"
owner: "@ai-engineer"
status: "draft"
blueprint-ref: "05-ai-core/planning/"
foundation-version: "1.0.0"
---

# Mission Engine
## Long-Lived Business Outcomes That Survive Sessions and Coordinate Multiple Agents

> **Instead of asking "Do this task," users create a Mission — a long-lived objective that persists across sessions, coordinates multiple agents, tracks progress, and adapts to changing conditions. The Mission Engine is what transforms Vestara from a task executor into a strategic partner.**

---

## Why Missions?

Most AI tools operate at the **task level**: one request, one response. Missions operate at the **outcome level**: a business goal that may take hours, days, or weeks to achieve — involving multiple agents, tools, and human decisions along the way.

| Dimension | Task | Mission |
|-----------|------|---------|
| Duration | Minutes | Hours to weeks |
| Scope | Single action | Business outcome |
| Agents | One | Multiple, coordinated |
| Persistence | Session-bound | Cross-session |
| Planning | Implicit | Explicit, dynamic |
| Adaptability | None | Re-plans on failure |
| Human involvement | Per-action | Milestone reviews |

---

## Mission Model

```typescript
interface Mission {
  id: string;                    // UUID v7
  name: string;
  description: string;
  objective: string;             // Primary success criterion
  status: MissionStatus;
  
  // Strategic context
  priority: 'low' | 'normal' | 'high' | 'critical';
  deadline?: string;             // ISO 8601
  constraints: MissionConstraint[];
  
  // Planning
  plan: MissionPlan | null;
  
  // Execution
  agents: AgentAssignment[];
  progress: number;              // 0.0 — 1.0
  artifacts: Artifact[];
  
  // History
  created_at: string;
  started_at?: string;
  completed_at?: string;
  updated_at: string;
  
  // Ownership
  ownerId: string;               // User or Agent ID
  organizationId?: string;
}

type MissionStatus =
  | 'draft'          // Defined but not yet started
  | 'planning'       // Planning Engine decomposing goal
  | 'active'         // Execution in progress
  | 'blocked'        // Waiting for external input
  | 'reviewing'      // Completed phase, awaiting review
  | 'completed'      // Objective achieved
  | 'failed'         // Objective not achievable
  | 'cancelled'      // Manually cancelled
  | 'archived';      // Preserved for learning

interface MissionConstraint {
  type: 'time' | 'resource' | 'quality' | 'security' | 'compliance';
  description: string;
  severity: 'must' | 'should' | 'nice';
}

interface MissionPlan {
  phases: MissionPhase[];
  dependencies: Dependency[];
  risks: Risk[];
  timeline: Timeline;
  created_at: string;
  updated_at: string;
}

interface MissionPhase {
  id: string;
  name: string;
  description: string;
  order: number;
  status: 'pending' | 'active' | 'completed' | 'failed' | 'skipped';
  tasks: MissionTask[];
  dependencies: string[];           // Phase IDs this depends on
  estimatedHours: number;
  progress: number;                 // 0.0 — 1.0
  started_at?: string;
  completed_at?: string;
}

interface MissionTask {
  id: string;
  title: string;
  description: string;
  assignedAgentId?: string;
  toolIds?: string[];
  estimatedHours: number;
  status: 'pending' | 'active' | 'completed' | 'failed';
  result?: string;
  artifactIds?: string[];
}

interface Risk {
  description: string;
  probability: 'low' | 'medium' | 'high';
  impact: 'low' | 'medium' | 'high' | 'critical';
  mitigation: string;
  contingency: string;
}

interface AgentAssignment {
  agentId: string;
  role: 'lead' | 'contributor' | 'reviewer';
  capabilities: string[];
  phaseIds: string[];
}
```

---

## Mission Lifecycle

```
Draft → Planning → Active → Reviewing → Completed
                  ↓            ↓
              Replanning    Blocked → Active
                  ↓            ↓
              Active        Cancelled
                            Failed
                            Archived
```

### State Details

| State | Who Drives | Duration | Exit Criteria |
|-------|------------|----------|---------------|
| `draft` | User | Indefinite | User triggers start |
| `planning` | Planning Agent | Minutes | Plan approved |
| `active` | Assigned agents | Hours-weeks | Phase completed |
| `blocked` | External | Variable | Human unblock |
| `reviewing` | Human + QA | Hours | Approval or revision |
| `replanning` | Planning Agent | Minutes | New plan approved |
| `completed` | — | — | Mission archived |
| `failed` | — | — | Mission archived |
| `cancelled` | — | — | Mission archived |
| `archived` | — | — | Learning extracted |

---

## Mission Engine Interface

```typescript
interface MissionEngine {
  /** Create a new mission */
  create(input: CreateMissionInput): Promise<Mission>;
  
  /** Start planning a mission */
  startPlanning(missionId: string): Promise<Mission>;
  
  /** Approve a plan and start execution */
  approvePlan(missionId: string): Promise<Mission>;
  
  /** Get mission status and progress */
  getStatus(missionId: string): Promise<MissionStatus>;
  
  /** Get detailed mission progress */
  getProgress(missionId: string): Promise<MissionProgress>;
  
  /** Update mission priority or constraints */
  updateMission(missionId: string, updates: Partial<Mission>): Promise<Mission>;
  
  /** Pause/unblock mission */
  setBlocked(missionId: string, reason: string): Promise<Mission>;
  unblock(missionId: string): Promise<Mission>;
  
  /** Cancel mission */
  cancel(missionId: string, reason: string): Promise<Mission>;
  
  /** Archive completed/failed mission */
  archive(missionId: string): Promise<void>;
  
  /** List missions with filters */
  listMissions(filter?: MissionFilter): Promise<MissionSummary[]>;
  
  /** Add artifact to mission */
  addArtifact(missionId: string, artifact: Artifact): Promise<void>;
  
  /** Replan a failed or blocked mission */
  replan(missionId: string): Promise<Mission>;
  
  /** Extract learning from completed mission */
  extractLearnings(missionId: string): Promise<LearningReport>;
}
```

---

## Mission Planning Algorithm

```
1. Receive: Mission objective + constraints
2. Analyze: Understand scope, identify key domains
3. Decompose: Break into phases (what order?)
4. Dependencies: Identify phase ordering constraints
5. Agents: Determine required agent types
6. Risks: Identify risks at each phase
7. Estimate: Time/resources per phase
8. Review: Present plan for human approval
9. Execute (when approved): Assign agents, begin
10. Monitor: Track progress, detect deviations
11. Adapt: Replan when conditions change
12. Complete: Validate outcomes, extract learnings
```

---

## Mission Example

```yaml
mission:
  name: "Build SaaS MVP"
  objective: "Launch a functional MVP of a project management SaaS tool"
  priority: "high"
  deadline: "2025-12-31"
  
  plan:
    phases:
      - name: "Research & Requirements"
        order: 1
        estimatedHours: 8
        tasks:
          - "Analyze competitor PM tools"
          - "Define MVP feature set"
          - "Create user stories"
      
      - name: "Architecture & Design"
        order: 2
        estimatedHours: 16
        tasks:
          - "Design database schema"
          - "Design API contracts"
          - "Design UI wireframes"
      
      - name: "Core Implementation"
        order: 3
        estimatedHours: 80
        tasks:
          - "Implement authentication"
          - "Implement project CRUD"
          - "Implement task management"
          - "Implement team features"
      
      - name: "Testing & Launch"
        order: 4
        dependencies: [3]
        estimatedHours: 16
        tasks:
          - "Integration testing"
          - "Performance optimization"
          - "Deployment setup"
          - "Launch"
  
  agents:
    - agentId: "planning-agent"
      role: "lead"
      phaseIds: ["1", "2"]
    - agentId: "coding-agent"
      role: "contributor"
      phaseIds: ["3"]
    - agentId: "qa-agent"
      role: "reviewer"
      phaseIds: ["4"]
```

---

## Mission ↔ Other Objects

```
User creates Mission
    ↓
MissionEngine creates Mission (draft)
    ↓
Planning Agent creates MissionPlan
    ↓
Plan approved → Mission becomes Active
    ↓
For each phase:
    ├─ Agent Runtime executes tasks
    ├─ Task creates Artifacts
    └─ Human reviews phase output
    ↓
All phases complete → Mission Completed
    ↓
Learnings extracted → Mission Archived
```

---

*The Mission Engine is what makes Vestara a strategic partner, not just a tool. Missions turn vague goals into structured outcomes, coordinate multiple agents toward a common objective, and learn from every completed mission to improve future ones.*
