# Vestara Product Principles

> **The Architecture Era created confidence for engineers.**
> **The Product Era creates confidence for users.**

---

## Product Principle #1 — Understand Before Acting

```
Open → Understand → Explain → Plan → Ask → Execute → Verify → Remember
```

Every capability inherits this principle. The first responsibility is comprehension, not generation.

---

## Product Principle #2 — The CLI Is the Reference

Every flagship capability is demonstrable from the command line before it appears in a Workspace or IDE. The CLI is the reference implementation. The Workspace is the visual experience. The IDE is the developer experience. All three exercise the same runtime.

```
vestara open      → understand any codebase
vestara explain   → reason about any decision
vestara plan      → decompose any goal
vestara implement → execute any approved change
vestara verify    → validate any result
vestara commit    → persist any outcome
```

---

## Product Principle #3 — Four Brains, One Experience

Users never need to know the names. They experience the result.

| Brain | What It Does | User Sees |
|-------|-------------|-----------|
| Knowledge | Discovers facts, frameworks, dependencies, architecture | Repository summary |
| Memory | Remembers previous analyses, decisions, preferences | Historical context |
| Executive | Prioritizes, analyzes, plans, recommends | Actionable insights |
| Conversation | Explains, teaches, answers, guides | Natural dialogue |

---

## First Flagship Capability: Repository Comprehension

**Success**: After opening a repository, Vestara answers questions an experienced teammate could answer after hours of study.

**Success Criteria**:
- [ ] Repository type and framework detected
- [ ] Architecture summarized
- [ ] Entry points located
- [ ] Dependency graph created
- [ ] Risks identified
- [ ] Recommendations generated
- [ ] Follow-up questions answered

**CLI Demo**:
```bash
$ vestara open ~/projects/unknown-repo
# → Framework detected, architecture identified, risks surfaced
```

---

## Sprint Success Definition

Every sprint answers four questions:

| Question | Measurement |
|----------|-------------|
| What user capability did Vestara gain? | Demo-able feature |
| Which existing contracts did it implement? | Traceability in commit |
| What measurable proof demonstrates it works? | Test + metric |
| Did the Golden Path remain intact? | `vestara doctor` passes |

---

*Architecture is the platform. Product capabilities are the destination. Understand before acting.*
