---
title: "Cognitive Architecture v1.0 — The Five Stages of Processing"
volume: "05-ai-core"
book: "Book 3: AI Architecture"
version: "1.0.0"
status: "ratified"
owner: "@ai-engineer"
last-reviewed: "2025-07-23"
tags: ["cognitive", "architecture", "five-stages", "memory", "reasoning"]
---

# Cognitive Architecture v1.0
## The Five Stages of Processing — Perception → Understanding → Memory → Reasoning → Action

> **Every interaction Vestara has passes through five cognitive stages. This architecture transforms Vestara from an execution engine into a cognitive engine — one that perceives, understands, remembers, reasons, and acts in a continuous learning loop.**

---

## The Cognitive Loop

```
               USER
                 │
                 ▼
         Conversation Runtime
                 │
                 ▼
            Perception
          (Stage 1)
                 │
                 ▼
         Understanding
          (Stage 2)
                 │
                 ▼
             Memory
          (Stage 3)
                 │
                 ▼
            Reasoning
          (Stage 4)
                 │
                 ▼
             Action
          (Stage 5)
                 │
                 ▼
         Observation (loop)
```

Every action produces a new observation. Vestara continuously learns.

---

## Stage 1 — Perception

**Purpose**: Normalize all incoming information into a canonical `Observation` model.

Sources:
- Conversation (messages, responses)
- Voice (speech input)
- Filesystem (file changes, git events)
- Workspace (window changes, project switches)
- Tools (execution results)
- Events (system events, external APIs)
- Organization (team activity, shared context)

```typescript
interface Observation {
  id: string;
  source: 'conversation' | 'filesystem' | 'workspace' | 'voice' | 'tool' | 'organization' | 'system';
  timestamp: string;
  type: string;
  payload: unknown;
  confidence: number;
  metadata?: Record<string, unknown>;
}
```

**Output**: Canonical Observation objects.

---

## Stage 2 — Understanding

**Purpose**: Interpret observations to extract structured meaning.

Questions answered:
- Is this important?
- Is it a fact, preference, decision, or event?
- Is it temporary or permanent?
- Does it relate to an existing mission?
- Does it change previous knowledge?
- What type of memory should it become?

```typescript
interface Understanding {
  observationId: string;
  extracted: MemoryExtraction[];
  relationships: string[];     // IDs of related memories
  confidence: number;
}

interface MemoryExtraction {
  type: MemoryType;
  content: string;
  importance: number;           // 0.0 — 1.0
  confidence: number;           // 0.0 — 1.0
  tags: string[];
  suggestedLayer: MemoryLayer;
  relatesTo?: string[];         // Related memory IDs
}
```

**Output**: Structured extractions ready for memory storage.

---

## Stage 3 — Memory

**Purpose**: Persist, organize, and retrieve information across four layers.

| Layer | TTL | Purpose |
|-------|-----|---------|
| Working | Session | Current conversation context |
| Episodic | 30 days | Recent events and interactions |
| Semantic | Infinite | Facts, preferences, decisions |
| Long-Term | Permanent | Consolidated knowledge |

```typescript
interface Memory {
  id: string;
  type: MemoryType;
  layer: MemoryLayer;
  content: string;
  summary?: string;
  importance: number;
  confidence: number;
  tags: string[];
  source: string;
  relationships: string[];
  embedding?: number[];
  missionId?: string;
  createdAt: string;
  lastAccessed: string;
  expiresAt?: string;
}
```

**Pipeline**: Store → Score → Link → Consolidate → Archive/Prune

---

## Stage 4 — Reasoning

**Purpose**: Use memories to answer questions, make decisions, and form plans.

```
Question → Retrieve Memories → Compare → Evaluate → Infer → Plan → Respond
```

Reasoning engines (future):
- Planning Engine — Goal decomposition and scheduling
- Mission Engine — Long-running business outcomes
- Workflow Engine — Automated multi-step processes
- Decision Engine — Trade-off analysis and recommendations

---

## Stage 5 — Action

**Purpose**: Execute tools and observe results.

```
Need Identified → Permission Check → Tool Selection → Execution → Observation → Memory
```

Every action returns an Observation. The loop continues.

---

## The Cognitive Engine

```
cognitive/
├── observation/     Stage 1: Normalize inputs
├── understanding/   Stage 2: Extract meaning
├── memory/          Stage 3: Store and retrieve
├── reasoning/       Stage 4: Use memories (future)
└── learning/        Stage 5: Close the loop
```

---

## Milestone 3.7 Deliverables

| Deliverable | Description |
|-------------|-------------|
| Observation Engine | Normalizes all incoming information into canonical Observations |
| Understanding Engine | Extracts structured facts, preferences, decisions from Observations |
| Context Builder v2 | Incorporates conversation history + memories + knowledge + mission state |
| Memory Lifecycle | Scores importance, stores in correct layer, links related memories |
| Background Consolidation | Merges duplicates, summarizes, expires, refreshes |

---

**The Cognitive Architecture transforms Vestara from an execution engine into a cognitive engine. Every interaction makes Vestara incrementally more knowledgeable. Storage saves everything. Memory remembers what matters.**
