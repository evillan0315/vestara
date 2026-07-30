---
title: "Conversation Runtime"
volume: "14-conversation"
book: "Book 3: AI Architecture"
version: "1.0.0"
status: "draft"
owner: "@ai-engineer"
last-reviewed: "2025-07-30"
next-review: "2026-01-30"
tags: ["conversation", "runtime", "session", "message"]
---

# Conversation Runtime

## Session Model

Every conversation begins as a `Session` — a scoped container with:

- **Participants** (human and/or agent)
- **Context** (files, workspace state, memory snapshots)
- **Message history** (ordered sequence of messages)
- **Lifecycle state** (active, archived, resolved)

## Message Lifecycle

```
Author creates → Validated → Routed → Processed → Streamed → Persisted
```

| Stage | Responsibility |
|-------|---------------|
| Create | Message typed/spoken by any participant |
| Validate | Schema check, permission check, rate limit |
| Route | Deliver to correct handler (agent, command, group) |
| Process | Generate response (AI inference, tool call, lookup) |
| Stream | Real-time delivery to all participants |
| Persist | Store in conversation history, index for retrieval |

## State Machine

```
Active → [Timeout] → Idle → [Resume] → Active
Active → [Resolved] → Archived
Active → [Cancelled] → Archived
```
