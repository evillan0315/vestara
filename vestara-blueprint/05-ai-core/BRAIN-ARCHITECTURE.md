---
title: "Vestara Brain Architecture — The Four Brains"
volume: "05-ai-core"
book: "Book 3: AI Architecture"
version: "1.0.0"
status: "ratified"
owner: "@chief-architect"
last-reviewed: "2025-07-23"
tags: ["architecture", "brain", "four-brains", "cognitive", "long-term-vision"]
---

# Vestara Brain Architecture — The Four Brains

> **Most AI platforms have one brain — the language model. Vestara has four. Each brain handles a distinct cognitive function. Together they form a complete AI operating platform that learns, remembers, plans, and acts autonomously.**

---

## The Four Brains

```
                    VESTARA BRAIN
══════════════════════════════════════════════════════════════

                Conscious Layer
        Conversation • Voice • Avatar • UI

────────────────────────────────────────────────────────────

              Cognitive Layer
    Planning • Reasoning • Missions • Memory

────────────────────────────────────────────────────────────

             Knowledge Layer
   Files • Projects • Git • Documentation
   APIs • Vector Search • Internet

────────────────────────────────────────────────────────────

             Execution Layer
 Tools • Shell • IDE • Automation • Workflows

────────────────────────────────────────────────────────────

             Runtime Layer
 Kernel • Events • State • Metrics • Logging

────────────────────────────────────────────────────────────

           Hardware / Operating System
     Linux • Raspberry Pi • Portable SSD
```

---

## Brain 1 — Conversation (The Interface)

**What users see.** The multi-turn dialogue, voice, avatar, and UI that form the human-AI interface.

| Component | Status | Description |
|-----------|--------|-------------|
| Conversation Runtime | ✅ v0.1 | Multi-turn chat, context assembly, streaming |
| Voice (STT/TTS) | 📋 v0.4 | Speech-to-text and text-to-speech |
| Avatar | 🔮 Future | Visual AI presence |
| Workspace UI | 📋 Phase 4 | Desktop integration |

**Architecture**: Conversation Runtime → Context Builder → Provider Runtime

---

## Brain 2 — Memory (The Persistent Self)

**What users don't see.** Four-layer memory that persists across sessions, learns from every interaction, and grows more useful over time.

| Layer | TTL | Function | Status |
|-------|-----|----------|--------|
| Working (L1) | Session | Current conversation context | ✅ v0.1 |
| Episodic (L2) | 30 days | Recent events and interactions | ✅ v0.1 |
| Semantic (L3) | Infinite | Facts, preferences, decisions | ✅ v0.1 |
| Long-Term (L4) | Permanent | Consolidated knowledge | ✅ v0.1 |

**Pipeline**:
```
Conversation → Memory Extraction → Importance Scoring → 
Layer Assignment → Persistence → Background Consolidation
```

**Key Principle**: Every response ends with `MemoryExtractor` running in the background. Not everything is stored — the system decides what deserves remembering.

---

## Brain 3 — Knowledge (The Expert)

**Vestara's understanding of the world.** Documents, repositories, code, specifications, and external APIs indexed for retrieval-augmented generation.

| Component | Status | Description |
|-----------|--------|-------------|
| Knowledge Runtime | 📋 v0.2 | Document ingestion, indexing, search |
| RAG Pipeline | 📋 v0.2 | Query → Retrieve → Rerank → Generate |
| Vector Search | 📋 v0.2 | Embedding-based similarity search |
| Filesystem Watcher | 📋 v0.2 | Auto-index project files |
| Web Import | 🔮 v0.4 | Import documentation, articles |

**Pipeline**:
```
Document → Chunk → Embed → Index → Search → Retrieve → Context
```

---

## Brain 4 — Planning (The Executive)

**Vestara's ability to think ahead.** Missions, goals, task decomposition, scheduling, and autonomous execution.

| Component | Status | Description |
|-----------|--------|-------------|
| Mission Engine | 📋 v0.3 | Long-lived business outcomes |
| Planning Agent | 📋 v0.3 | Goal decomposition |
| Task Scheduler | 📋 v0.3 | Background execution |
| Workflow Engine | 📋 v0.3 | Automated multi-step processes |

**Pipeline**:
```
Goal → Mission → Tasks → Dependencies → Execution → Verification
```

---

## Platform Evolution

```
Phase 3.1-3.6   ✅  AI Core v0.1 — Runtime, Provider, Conversation,
                    Streaming, Action, Persistence, Memory
                            ↓
Phase 3.7       🔄  Cognitive Memory Integration — Wire memory into
                    every conversation (extract, score, retrieve)
                            ↓
Phase 3.8       📋  Knowledge Runtime — RAG, document indexing,
                    vector search, filesystem watcher
                            ↓
Phase 3.9       📋  Mission & Planning — Long-running goals,
                    autonomous execution, scheduling
                            ↓
Phase 4.0       📋  Workspace Runtime — Desktop environment around
                    the AI Core
                            ↓
Phase 5.0       📋  Vestara AI OS — Bootable LXQt distribution
                    with deep runtime integration
                            ↓
Phase 6.0       🔮  Cloud Runtime — Sync, remote agents, collaboration
                            ↓
Phase 7.0       🔮  Developer Platform — Plugin ecosystem, SDK,
                    marketplace, replacing external tools
```

---

## Key Architectural Principles

| Principle | Description |
|-----------|-------------|
| **Provider is one step** | The AI provider is not the center — it's one component in a pipeline |
| **Memory is automatic** | Every interaction feeds memory; no explicit save needed |
| **Missions are first-class** | Conversations, tasks, files, and memories belong to missions |
| **Background autonomy** | The runtime continues working when the user is away |
| **Organizational memory** | Companies have their own persistent intelligence |
| **Boot is an experience** | The startup sequence reinforces platform identity |

---

## The Autonomous Runtime

Even when the user is away, Vestara continues:

```
Indexing → Planning → Monitoring → Learning → Syncing → Reporting
```

When the user returns:

> "Good morning. While you were away I indexed 24 new files, consolidated 37 memories, found two security advisories, and created four suggested tasks."

The user didn't ask. Vestara volunteered useful work.

---

## The Boot Experience

```
Power On → Secure Boot → Vestara Animation → Kernel → Runtime →
Memory Restore → Knowledge Load → Provider Connect → Workspace →
Voice → Mission Engine → Ready
```

Instead of seeing a desktop immediately, the user sees the platform coming to life:

```
Initializing Cognitive Engine...
Loading Long-Term Memory...
Restoring Workspace...
Synchronizing Missions...
Connecting AI Providers...
Checking Background Agents...

Vestara Ready.
```

---

## From AI Application to AI Operating Environment

| Stage | Identity |
|-------|----------|
| Today | "An AI runtime" |
| Next | "An AI workspace" |
| Later | "An AI operating environment" |
| Eventually | "A personal and organizational cognitive platform" |

---

*This architecture defines the long-term evolution of Vestara. Each brain is independently buildable, testable, and improvable. The Four Brains working together create something fundamentally different from any single-model AI platform.*
