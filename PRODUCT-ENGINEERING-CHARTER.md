# Product Engineering Charter
## Architecture Discovery Complete — Product Era Begins

> **Effective immediately, Vestara transitions from Architecture-Driven Development to Product-Driven Engineering. The frozen architecture is the platform. The question is no longer "How should Vestara be built?" but "What capability does Vestara gain this sprint?"**

---

## The Transition

| Phase | Focus | Status |
|-------|-------|--------|
| Architecture Discovery | "What is Vestara?" | ✅ Complete |
| Product Engineering | "What capability does Vestara gain?" | 🔄 Active |

## Success Definition

Every sprint answers four questions:

| Question | Measurement |
|----------|-------------|
| **What user capability did Vestara gain?** | Demo-able feature |
| **Which existing contracts did it implement? | Traceability in commit |
| **What measurable proof demonstrates it works? | Test + metric |
| **Did the Golden Path remain intact? | `vestara doctor` passes |

## First Capability: Repository Intelligence

**Demonstration:**
```
$ vestara open ~/projects/unknown-repo

Analyzing repository...
✓ Framework detected
✓ Architecture identified
✓ Entry points mapped
✓ Dependency graph built
✓ Coding conventions learned
✓ Documentation indexed

Repository Summary
────────────────────────────────
Type: React + Vite + TypeScript
Architecture: Feature-first modular application
Key modules: Authentication, Dashboard, Billing, AI Services
Potential risks: 2 circular dependencies, 4 outdated packages

Recommended reading order:
  1. README.md
  2. ARCHITECTURE.md
  3. src/app
  4. src/features
```

## Four Brains Collaboration

```
Knowledge Brain  →  Builds repository model (structure, dependencies, conventions)
Executive Brain  →  Decides what to analyze (critical paths, risks, patterns)
Memory Brain     →  Remembers previous discoveries (cross-session context)
Conversation     →  Presents findings naturally (tiered, actionable)
```

---

*Architecture is the platform. Product capabilities are the destination.*
