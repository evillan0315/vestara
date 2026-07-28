---
title: "AI Core Architecture — Volume Overview"
volume: "05-ai-core"
book: "Book 3: AI Architecture"
version: "1.0.0"
status: "approved"
owner: "@ai-engineer"
last-reviewed: "2025-07-23"
next-review: "2026-01-23"
tags: ["ai", "core", "architecture", "overview"]
---

# Volume 05: AI Core Architecture
## The Heart of Vestara — Every AI Subsystem Specified

> **Mission**: Build an AI-native platform where conversation, memory, reasoning, planning, knowledge, agents, and providers work together as a unified intelligence layer — provider-agnostic, privacy-first, and locally capable.

---

## 📋 Volume Contents

```
05-ai-core/
│
├── README.md                              ← This file
├── AI_OVERVIEW.md                         ← High-level AI architecture
├── AI_CONSTITUTION.md                     ← AI behavior governance
├── AI_PIPELINE.md                         ← End-to-end AI processing pipeline
├── AI_DECISION_MODEL.md                   ← How AI decisions are made
│
├── conversation/                          ← Multi-turn conversation engine
├── reasoning/                             ← Chain-of-thought, structured reasoning
├── planning/                              ← Task decomposition & planning
├── memory/                                ← Memory layers & consolidation
├── knowledge/                             ← Knowledge base & RAG
├── filesystem/                            ← AI filesystem operations
├── providers/                             ← Provider management & routing
├── models/                                ← Model registry & capability map
├── prompts/                               ← Prompt management & versioning
├── context/                               ← Context window management
├── orchestration/                         ← Multi-agent orchestration
├── agents/                                ← Agent runtime & lifecycle
├── evaluation/                            ← AI evaluation & benchmarking
├── safety/                                ← AI safety & guardrails
├── voice/                                 ← Speech-to-text, text-to-speech
├── vision/                                ← Image analysis & generation
├── automation/                            ← Automated AI workflows
└── learning/                              ← Continuous learning & adaptation
```

---

## 🧠 AI Architecture Principles

| Principle | Description |
|-----------|-------------|
| **Provider-Agnostic** | Abstract interface over all AI providers (OpenCode, Ollama, OpenAI, Anthropic, Google) |
| **Local-First** | Core AI capabilities work completely offline via local models |
| **Memory-Centric** | Persistent memory across sessions is the foundation of intelligence |
| **Agent-Native** | Agents are first-class citizens with lifecycle, tools, and memory |
| **Composable** | AI subsystems combine like LEGO blocks |
| **Evaluated** | Every capability has benchmarks, regression tests, and safety checks |
| **Transparent** | Users see which model, cost, latency for every AI interaction |

---

## 🎯 Subsystems

| Subsystem | Purpose | Gen 1 Scope |
|-----------|---------|-------------|
| **Conversation** | Multi-turn chat with context, memory, tool use | ✅ Full |
| **Memory** | Short-term, long-term, episodic memory with consolidation | ✅ Full |
| **Knowledge** | RAG pipeline with FTS + vector search | ✅ Core |
| **Providers** | Router with OpenCode default, Ollama local, external fallback | ✅ Full |
| **Agents** | Runtime, tools, execution, persistence | ✅ Core |
| **Prompts** | Templates, versioning, optimization | 📋 MVP |
| **Evaluation** | Benchmarks, regression, safety checks | 📋 Planned |
| **Voice** | STT/TTS via Web Speech API + Whisper | 🔄 Planned |
| **Planning** | Task decomposition, multi-step execution | 🔮 Gen 2 |
| **Reasoning** | Chain-of-thought, structured reasoning | 🔮 Gen 2 |

---

## 🔗 Cross-References

| Volume | Relationship |
|--------|--------------|
| `04-platform` | Platform consumes AI capabilities |
| `06-workspace` | Workspace UI presents AI interactions |
| `08-cloud` | Cloud extends AI with distributed inference |
| `12-data` | Memory and knowledge storage |
| `21-research` | AI research feeds into core improvements |

---

**END OF AI CORE VOLUME OVERVIEW**

*The AI core is the brain of Vestara. All intelligence flows through these subsystems.*
