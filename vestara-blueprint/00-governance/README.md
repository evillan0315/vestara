---
title: "Governance — Volume Overview"
volume: "00-governance"
book: "Book 1: Vision & Business"
version: "1.0.0"
status: "approved"
owner: "@chief-architect"
last-reviewed: "2025-07-23"
next-review: "2026-01-23"
tags: ["governance", "constitution", "rules", "lifecycle"]
---

# Volume 00: Governance
## The Constitution, Rules, and Lifecycle That Govern All of Vestara

> **Mission**: Define the supreme authority documents that govern every decision, architecture, and implementation in the Vestara ecosystem.

---

## 📋 Volume Contents

```
00-governance/
│
├── README.md                              ← This file
├── 01-ai-constitution.md                  ← Master prompt for AI agents
├── 02-engineering-rules.md                ← Non-negotiable engineering standards
├── 03-ai-development-lifecycle.md         ← AIDL — AI Development Lifecycle
├── 04-decision-log.md                     ← Architectural Decision Records
├── 05-compatibility.md                    ← AI agent compatibility (Claude, OpenCode, etc.)
├── 06-ai-development-framework.md         ← VADF — Vestara AI Development Framework
```

---

## 📜 Governance Hierarchy

```
VESTARA_CONSTITUTION.md  ← Supreme authority (root level)
        │
        ▼
01-ai-constitution.md   ← Master prompt for all AI agents
        │
        ▼
02-engineering-rules.md ← Non-negotiable rules enforced by CI
        │
        ▼
03-ai-development-lifecycle.md ← AIDL workflow
        │
        ▼
04-decision-log.md      ← Immutable record of decisions
        │
        ▼
05-compatibility.md     ← AI agent integration (Claude, OpenCode, etc.)
        │
        ▼
06-ai-development-framework.md ← VADF methodology
```

---

## 🔗 Cross-References

| Document | Supersedes/Maps To |
|----------|-------------------|
| `VESTARA_CONSTITUTION.md` | Root-level constitution |
| `AI_INSTRUCTION.md` | Maps to 01-ai-constitution.md |
| `AI_RULES.md` | Maps to 02-engineering-rules.md |
| `AI_AGENTS.md` | Maps to 06-ai-development-framework.md |
| `AI_CONTEXT.md` | Current project state |

---

**END OF GOVERNANCE VOLUME OVERVIEW**

*Governance makes Vestara predictable, consistent, and self-sustaining.*
