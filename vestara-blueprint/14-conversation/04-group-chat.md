---
title: "Group Chat"
volume: "14-conversation"
book: "Book 3: AI Architecture"
version: "1.0.0"
status: "draft"
owner: "@chief-architect"
last-reviewed: "2025-07-30"
next-review: "2026-07-30"
tags: ["conversation", "group-chat", "multi-agent", "collaboration", "vision"]
---

# Group Chat

## Vision

Vestara supports collaborative conversations between humans and multiple AI agents.

Unlike traditional AI assistants where one model impersonates many roles, Vestara presents each specialized agent as an independent participant within a shared conversation.

The user collaborates with an AI engineering organization rather than a single assistant.

---

## Goals

- Human ↔ AI collaboration
- AI ↔ AI collaboration
- Shared project awareness
- Workspace-native conversations
- Voice-first support
- Persistent memory
- Artifact generation

---

## Participants

```text
👤 Human

🟦 Planner

🟩 Engineer

🟨 Reviewer

🟥 Verifier

🟪 Context
```

Each participant has:

- Identity
- Role
- Personality
- Capabilities
- Memory
- Voice
- Presence
- Permissions

---

## Communication Model

```text
Human
     │
     ▼
Conversation Runtime
     │
     ├── Planner
     ├── Engineer
     ├── Reviewer
     ├── Verifier
     └── Context
```

Messages are published to the Conversation Runtime. Each agent decides independently whether to respond.

---

## Attention Model

Not every message requires every agent.

**Example:**

```text
@engineer
Implement authentication.
```

Only Engineer responds.

```text
@planner
Design EV-004.
```

Planner becomes active.

```text
@all
Morning standup.
```

Everyone joins.

---

## Agent-to-Agent Collaboration

Agents may communicate without direct human intervention.

```text
Planner → Engineer
Engineer → Reviewer
Reviewer → Planner
Verifier → Human
```

The user receives the final recommendation while retaining full visibility into the discussion.

---

## Voice Collaboration

Future versions support spoken conversations.

```text
Eddie → "Planner, what's today's priority?"
```

Planner responds. Engineer may interrupt if work has completed. Verifier may confirm completion. Conversation behaves like an engineering standup rather than a voice assistant.

---

## Presence

Agents maintain observable state.

| Agent | Status |
|-------|--------|
| Planner | Thinking |
| Engineer | Implementing Runtime |
| Reviewer | Reviewing Pull Request |
| Verifier | Running Tests |
| Context | Searching Knowledge |

Presence is visible across Workspace, Dashboard, and Chat.

---

## Workspace Awareness

Every conversation belongs to a Workspace. Agents automatically understand:

- Repository
- Current branch
- Active project
- Open tasks
- Knowledge Graph
- Memory
- Previous decisions

The user never needs to repeatedly explain context.

---

## Long-Term Vision

> Vestara's goal is not to create a chatbot with multiple personalities. Its goal is to create an AI engineering organization where humans and specialized AI agents collaborate naturally through conversation, shared context, and coordinated execution.
