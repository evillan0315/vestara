---
title: "Memory Architecture — Layered Memory System"
volume: "05-ai-core"
book: "Book 3: AI Architecture"
version: "1.0.0"
status: "approved"
owner: "@ai-engineer"
last-reviewed: "2025-07-23"
next-review: "2026-01-23"
tags: ["memory", "consolidation", "layers", "scoring"]
---

# Memory Architecture
## The Layered Memory System That Makes Vestara Persistent

> **Memory is the foundation of Vestara's intelligence. Unlike stateless AI chatbots, Vestara remembers everything across sessions through a layered memory architecture with automatic consolidation and importance scoring.**

---

## 📚 MEMORY LAYERS

```
Layer 0: Constitutional Memory (Permanent)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
• Mission, vision, architecture, standards
• Never consolidated or evicted
• Loaded at system initialization
• Stored in: Blueprint documents (VESTARA_CONSTITUTION.md, AI_CONSTITUTION.md)

Layer 1: Platform Memory (Long-term, months-years)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
• User preferences, learned patterns
• Cross-project context (Gen 3+)
• Important decisions and rationale
• Stored in: SQLite (memories table) + .vestara/memory/

Layer 2: Project Memory (Medium-term, weeks-months)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
• Project-specific context and decisions
• Code patterns and conventions
• Team collaboration history
• Stored in: .vestara/memory/ (per project)

Layer 3: Session Memory (Short-term, session duration)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
• Current conversation context
• Temporary working state
• Recent tool outputs
• Stored in: In-memory (volatile), persisted to summaries

Layer 4: Execution Memory (Ephemeral, per execution)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
• Current agent execution context
• Files, code, logs, intermediate states
• Evicted on completion
• Stored in: In-memory (volatile)
```

---

## 🔄 CONSOLIDATION STRATEGY

```typescript
interface ConsolidationConfig {
  schedule: {
    interval: number;        // Every N interactions (default: 50)
    idleThreshold: number;   // Minutes of inactivity (default: 5)
  };
  importance: {
    highThreshold: number;    // Score ≥8: Full resolution, permanent
    mediumThreshold: number;  // Score 4-7: Summary with key details
    lowThreshold: number;     // Score 1-3: Keyword-only reference
  };
  archiving: {
    sessionAgeDays: number;   // Sessions older than this are archived (default: 30)
    summaryFormat: 'monthly' | 'quarterly'; // Archive granularity
  };
}
```

### Consolidation Process
1. **Trigger**: Every N interactions or idle threshold
2. **Score**: Each memory gets importance score (0-10)
3. **Summarize**: Low-importance memories coarsened; high-importance preserved
4. **Archive**: Sessions >30 days rolled into summaries
5. **Prune**: Remove truly ephemeral data (score <1, age >90 days)

---

## 📊 IMPORTANCE SCORING

| Factor | Weight | Description |
|--------|--------|-------------|
| **Recency** | 0.3 | More recent = higher score |
| **Frequency** | 0.2 | Repeated patterns = higher score |
| **User feedback** | 0.2 | Explicit save, star, or deletion |
| **Semantic novelty** | 0.15 | New information = higher score |
| **Action impact** | 0.15 | Created file, set preference = higher score |

---

## 🔗 CROSS-REFERENCES
- `05-ai-core/memory/` — Full memory subsystem specification
- `12-data/memory-storage.md` — Memory storage architecture
- `services/memory/src/` — Memory service implementation

---

**END OF MEMORY ARCHITECTURE**

*Memory makes Vestara persistent. Without it, every session starts blank.*
