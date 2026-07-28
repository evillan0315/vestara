---
id: "CAP-001"
title: "Workspace.Chat — Conversational AI Interface"
owner: "@ai-core-team"
status: "draft"
blueprint-ref: "05-ai-core/conversation/"
version: "1.0.0"
---

# CAP-001: Workspace.Chat
## Conversational AI Interface — The Primary User Interaction Mode

---

## Purpose
Provide a multi-turn conversational interface where users interact with AI agents, manage tasks, search knowledge, and control the workspace through natural language.

## Business Value
Chat is the primary interface for AI collaboration at Vestara. It is the highest-frequency touchpoint between users and the platform. Quality here defines the entire product experience.

---

## Specification

### Conversation Model

```typescript
interface Conversation {
  id: string;                    // UUID v7
  userId: string;                // Owner
  projectId?: string;            // Optional project scope
  title: string;                 // Auto-generated or user-set
  messages: Message[];
  metadata: {
    model: string;               // AI model used
    provider: string;            // AI provider used
    tokens: number;              // Total tokens
    cost: number;                // Estimated cost
    latency: number;             // Total response time
  };
  status: 'active' | 'archived' | 'deleted';
  createdAt: string;             // ISO 8601
  updatedAt: string;             // ISO 8601
}

interface Message {
  id: string;                    // UUID v7
  role: 'user' | 'assistant' | 'system' | 'tool';
  content: string;               // Markdown-formatted
  toolCalls?: ToolCall[];
  toolResults?: ToolResult[];
  attachments?: Attachment[];
  metadata: {
    tokens: number;
    latency: number;
    cost: number;
  };
  createdAt: string;
}
```

### Features

| Feature | Status | Priority | Notes |
|---------|--------|----------|-------|
| Multi-turn conversation | ✅ Gen 1 | P0 | Core capability |
| Markdown rendering | ✅ Gen 1 | P0 | Code blocks, tables, images |
| Streaming responses | ✅ Gen 1 | P0 | Real-time token display |
| Message editing | ✅ Gen 1 | P1 | Edit and re-submit |
| Conversation history | ✅ Gen 1 | P1 | Searchable history |
| Conversation branching | 🔄 Gen 2 | P2 | Branch from any message |
| Voice input/output | 🔮 Gen 2 | P2 | STT/TTS integration |
| Multi-agent conversations | 🔮 Gen 3 | P3 | Multiple AI agents in one chat |
| Avatar/visual presence | 🔮 Gen 3 | P3 | AI visual representation |

### Interactions

```
User sends message
  → Message validated (Zod)
  → Context assembled (memory + knowledge + project)
  → Provider selected (based on user preference)
  → Request streamed to provider
  → Response tokens streamed to UI
  → Response rendered (markdown)
  → Memory updated with interaction
  → If tools: execute, collect results, continue
```

### Tool Integration

When the AI detects a tool call during conversation:

```typescript
interface ToolCall {
  id: string;
  tool: 'search_knowledge' | 'search_memory' | 'create_task' | 
        'read_file' | 'write_file' | 'execute_command' | 'web_search';
  arguments: Record<string, unknown>;
  status: 'pending' | 'executing' | 'completed' | 'failed';
  result?: unknown;
  error?: Error;
}
```

**Tool confirmation**: High-risk tools (write_file, execute_command) require explicit user approval. Read-only tools execute automatically.

### UI States

| State | Visual | Behavior |
|-------|--------|----------|
| **Idle** | Empty input, conversation history visible | Waiting for user input |
| **Composing** | Text in input, send button active | User typing |
| **Streaming** | Animated cursor, response appearing | Tokens arriving from provider |
| **Tool executing** | Progress indicator, tool name shown | Waiting for tool result |
| **Error** | Error banner with retry option | Provider failure or tool error |
| **Offline** | Warning badge, model selector limited | Only local models available |

---

## Dependencies
- CAP-020: AI.Conversation — Conversation engine
- CAP-021: AI.Memory — Memory integration
- CAP-022: AI.Knowledge — Knowledge search
- CAP-023: AI.Providers — Provider routing
- CAP-062: Platform.Auth — Authentication
- CAP-063: Platform.Filesystem — File operations

## Interfaces

### API
- `POST /api/v1/chat` — Send message
- `GET /api/v1/chat/history` — Get conversation history
- `DELETE /api/v1/chat/:id` — Delete conversation
- `WebSocket /ws` — Streaming responses

### Events
- `chat:message.sent` — User sent a message
- `chat:response.start` — AI started responding
- `chat:response.complete` — AI finished responding
- `chat:tool.executing` — Tool execution started
- `chat:tool.completed` — Tool execution finished
- `chat:error` — Error occurred

### UI Components
- `ChatPanel` — Main chat container
- `MessageBubble` — Individual message display
- `MessageInput` — Input with send, attach, voice
- `ConversationList` — History sidebar
- `ModelSelector` — Provider/model picker
- `ToolConfirmationDialog` — Confirmation for tool execution

---

## Performance Targets

| Metric | Target | Measurement |
|--------|--------|-------------|
| Time to first token (OpenCode) | <500ms p95 | Real-user monitoring |
| Time to first token (Ollama) | <2s p95 | Real-user monitoring |
| Message send latency | <100ms p95 | From click to provider request |
| Streaming throughput | >50 tokens/s | Client-side measurement |
| Conversation load time | <500ms for 1000 messages | Pagination + virtualization |
| Memory per conversation | <10MB for 500 messages | Client-side tracking |

---

## Security Considerations

- All messages authenticated via JWT
- Message content sanitized (XSS prevention)
- File attachments scanned for malware (Gen 2)
- Prompt injection detection on user input
- PII detection on provider output
- Rate limiting: 100 messages/min per user
- Conversation data isolated per user

---

## Testing Requirements

| Test Type | Scope | Tool |
|-----------|-------|------|
| Unit | Message validation, formatting, state machine | Vitest |
| Integration | Full send → stream → render cycle | Vitest + Fastify inject |
| E2E | Complete user conversation flow | Playwright |
| Load | 100 concurrent conversations, streaming | k6 |
| Security | Prompt injection, XSS, rate limiting | OWASP ZAP |

---

## Future Roadmap

| Version | Features |
|---------|----------|
| Gen 1 | Multi-turn chat, markdown, streaming, tools, history |
| Gen 2 | Conversation branching, voice input/output, message reactions |
| Gen 3 | Multi-agent chat, avatar presence, persistent personas |
| Gen 4 | Autonomous conversation, proactive suggestions, cross-session memory |
| Gen 5 | Predictive conversation, emotional awareness, lifelong memory |

---

## Related Specifications

- CAP-020: AI.Conversation — Backend conversation engine
- CAP-021: AI.Memory — Memory and context integration
- SPEC-EVT-001: Chat Events — Event definitions for chat
- SPEC-API-001: Chat API — API contract for chat endpoints
