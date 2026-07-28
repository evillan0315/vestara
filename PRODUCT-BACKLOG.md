# Product Backlog
## Vestara Product Era — Capability-Driven Development

> **Every sprint delivers one capability that can be demonstrated live in under five minutes. The architecture backlog is closed. The product backlog is open.**

---

## Epic 001 — Repository Comprehension

**User Outcome:** A developer can point Vestara at any repository and understand it in minutes instead of hours.

**Primary Command:** `vestara open .`

**Pipeline:**

```
vestara open .
    │
Repository Discovery
    │
Project Detection
    │
Repository Intelligence
    │
Knowledge Index
    │
Memory Initialization
    │
Executive Reasoning
    │
Project Summary
    │
Interactive Workspace
```

**User Experience:**
```
$ vestara open .

Opening repository...

✓ Detected React 19 + Vite workspace
✓ Package manager: pnpm
✓ TypeScript project
✓ 14 applications
✓ 37 packages
✓ Indexed 18,432 files
✓ Built knowledge index
✓ Identified architecture patterns
✓ Loaded project context

Repository Summary
────────────────────────────────────
Project:  Vestara AI Core
Purpose:  AI-native engineering platform
Architecture: Monorepo, React, Fastify, TypeScript, SQLite
Entry Points: apps/cli, packages/kernel, packages/conversation
Detected Risks: 2 circular dependencies, 4 TODO hotspots
Suggested: explain, plan, build, find technical debt
```

**Brains Involved:** Knowledge, Memory, Executive, Conversation

**Platform Contracts:** knowledge.*, memory.*, reasoning.*, conversation.*

**Definition of Done:**
- [ ] Repository type and framework detected
- [ ] Architecture summarized
- [ ] Entry points located
- [ ] Dependency graph created
- [ ] Risks identified
- [ ] Recommendations generated
- [ ] Follow-up questions answered
- [ ] Demo: `vestara open .` on an unfamiliar repo, under 5 minutes

---

## Capability Ladder

| Sprint | Command | User Capability |
|--------|---------|-----------------|
| Epic 001 | `vestara open` | Understand any codebase in minutes |
| Epic 002 | `vestara explain` | Reason about any design decision |
| Epic 003 | `vestara plan` | Decompose goals into executable plans |
| Epic 004 | `vestara implement` | Execute approved changes safely |
| Epic 005 | `vestara verify` | Validate correctness automatically |
| Epic 006 | `vestara remember` | Persist knowledge across sessions |
| Epic 007 | `vestara collaborate` | Coordinate team workflows |

---

## Sprint Success

Every sprint produces one capability that:
- Is demonstrable live in under five minutes
- Implements existing contracts (no new architecture)
- Leaves the Golden Path intact
- Answers: "What could a user do today they couldn't yesterday?"

---

*The architecture backlog is closed. The product backlog is open. Deliver one meaningful capability at a time without violating the architecture.*
