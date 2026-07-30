---
title: "Conversation — Volume Overview"
volume: "14-conversation"
book: "Book 3: AI Architecture"
version: "1.0.0"
status: "draft"
owner: "@ai-engineer"
last-reviewed: "2025-07-30"
next-review: "2026-01-30"
tags: ["conversation", "runtime", "agents", "collaboration"]
---

# Volume 14: Conversation
## The Core Interaction Paradigm of Vestara

> **Mission**: Define the architecture, protocols, and behaviors governing every conversation in the Vestara ecosystem — human-to-AI, AI-to-AI, and group collaboration.

Conversation is not a feature of AI Core. It is the primary interface through which humans and AI agents collaborate. Every interaction — chat, voice, agent handoff, group discussion — is a variation of the conversation model defined here.

---

## Scope

| Interaction Type | Covered By |
|-----------------|------------|
| Human ↔ AI chat | Conversation Runtime |
| AI ↔ AI coordination | Agent Collaboration |
| Multi-participant (human + AI) | Group Chat |
| Voice / speech | Voice layer |
| Streaming responses | Streaming protocol |
| Shared context / memory | Memory-in-Conversation |

---

## Volume Contents

```
14-conversation/
│
├── README.md                         ← This file
├── 01-conversation-runtime.md        ─ Message lifecycle, session model, state
├── 02-message-bus.md                 ─ Event-driven message routing, topics
├── 03-agent-collaboration.md         ─ AI-to-AI delegation, handoff, escalation
├── 04-group-chat.md                  ─ Multi-agent collaboration vision (flagship)
├── 05-voice-conversations.md         ─ Speech in/out, VAD, turn-taking
├── 06-presence.md                    ─ Online/away/typing indicators, status
├── 07-mentions.md                    ─ @mentions, slash commands, intent routing
├── 08-threads.md                     ─ Threaded replies, branching, summarization
├── 09-conversation-memory.md         ─ What conversation remembers and surfaces
├── 10-conversation-artifacts.md      ─ Shared documents, code, drawings in chat
└── 11-multi-agent-delegation.md      ─ Agent-to-agent subtask delegation protocol
```

---

## Key Concepts

| Concept | Definition |
|---------|------------|
| **Session** | A single conversation span with a defined scope and participants |
| **Message** | An atomic unit of communication with type, author, and payload |
| **Thread** | A branching sub-conversation within a session |
| **Artifact** | A persistent shared object created or modified during conversation |
| **Agent Handoff** | Delegation of a subtask from one AI agent to another |

---

## Cross-References

| Volume | Relationship |
|--------|-------------|
| `05-ai-core` | Conversation runs on AI Core capabilities (planning, reasoning, memory) |
| `06-workspace` | Conversation surfaces in workspace UI |
| `13-design-system` | Conversation UI inherits VDS chat tokens and patterns |
| `11-security` | Message encryption, audit trails, access control |

---

**Conversation is the interface. Everything else is infrastructure.**
