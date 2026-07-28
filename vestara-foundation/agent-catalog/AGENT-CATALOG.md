---
id: "FND-005"
title: "Agent Catalog — Runtime Agent Definitions"
owner: "@ai-engineer"
status: "ratified"
blueprint-ref: "05-ai-core/AI_OVERVIEW.md"
foundation-version: "1.0.0"
---

# Agent Catalog

## Every Runtime Agent Type in the Vestara Platform

> **Unlike the engineering agent roles defined in the Blueprint (Chief Architect, Product Manager, etc.), these are the runtime agents that execute within the Agent Runtime. They are the autonomous actors that users interact with and that perform work in the platform.**

---

## Agent Contract Template

Every agent implements:

```typescript
interface VestaraAgent {
  id: string;
  type: AgentType;
  name: string;
  description: string;
  
  initialize(config: AgentConfig): Promise<void>;
  execute(task: Task, context: ExecutionContext): Promise<ExecutionResult>;
  stream(task: Task, context: ExecutionContext): AsyncIterable<ExecutionChunk>;
  cancel(): Promise<void>;
  pause(): Promise<void>;
  resume(): Promise<void>;
  getStatus(): AgentStatus;
  dispose(): Promise<void>;
}
```

---

## 📋 Agent Catalog

### AGT-001: Conversation Agent

```yaml
id: "AGT-001"
name: "Conversation Agent"
purpose: "Primary human-AI interaction. Multi-turn dialogue, tool use, memory, knowledge."
status: "standard"

capabilities:
  - "Multi-turn dialogue with context preservation"
  - "Tool orchestration within conversation"
  - "Memory access (read + write)"
  - "Knowledge search and RAG"
  - "Streaming responses"
  - "Message editing and correction"

default_model: "opencode/deepseek-v4-flash-free"
default_tools:
  - "vestara.knowledge.search"
  - "vestara.memory.search"
  - "vestara.filesystem.read"
  
events:
  emits:
    - "conversation:message.sent"
    - "conversation:tool.executing"
    - "conversation:response.complete"
  consumes:
    - "memory:consolidated"  # Refresh context

security:
  - "Tool execution requires user confirmation for writes"
  - "Memory access scoped to current user"
  - "Context isolated to current conversation"
```

### AGT-002: Planning Agent

```yaml
id: "AGT-002"
name: "Planning Agent"
purpose: "Decomposes high-level goals into structured plans with tasks, dependencies, and milestones."
status: "gen-2"

capabilities:
  - "Goal decomposition into sub-tasks"
  - "Dependency analysis and ordering"
  - "Resource estimation (time, cost, models)"
  - "Risk identification"
  - "Schedule generation"
  - "Replanning on failure"

default_model: "openai/gpt-4o"  # Requires stronger reasoning
default_tools:
  - "vestara.task.create"
  - "vestara.knowledge.search"
  - "vestara.memory.search"

input:
  - "Mission objective"
  - "Constraints (time, resources, quality)"
  - "Available agents and tools"
  - "Prior knowledge and context"

output:
  - "Structured plan with tasks and dependencies"
  - "Risk assessment"
  - "Resource allocation"
  - "Timeline estimate"

state_machine:
  - Analyzing: "Understanding the goal and constraints"
  - Decomposing: "Breaking down into sub-tasks"
  - Ordering: "Determining dependencies and sequence"
  - Reviewing: "Validating plan completeness"
  - Approved: "Plan ready for execution"
  - Replanning: "Adjusting based on execution feedback"
```

### AGT-003: Reasoning Agent

```yaml
id: "AGT-003"
name: "Reasoning Agent"
purpose: "Deep analytical reasoning, chain-of-thought, constraint solving, validation."
status: "gen-2"

capabilities:
  - "Chain-of-thought reasoning"
  - "Constraint satisfaction"
  - "Logical deduction and inference"
  - "Hypothesis testing"
  - "Decision analysis with trade-offs"
  - "Validation and consistency checking"

default_model: "anthropic/claude-3.5-sonnet"  # Strong reasoning

input:
  - "Problem statement"
  - "Constraints and criteria"
  - "Available data and context"
  - "Hypotheses or options"

output:
  - "Reasoned conclusion with chain-of-thought"
  - "Confidence assessment"
  - "Alternative options considered"
  - "Uncertainty quantification"
```

### AGT-004: Research Agent

```yaml
id: "AGT-004"
name: "Research Agent"
purpose: "Investigates topics, searches knowledge, analyzes findings, produces research reports."
status: "gen-1"

capabilities:
  - "Web search and information gathering"
  - "Knowledge base search"
  - "Source evaluation and citation"
  - "Cross-reference validation"
  - "Summary and report generation"
  - "Competitive analysis"

default_model: "opencode/deepseek-v4-flash-free"
default_tools:
  - "vestara.web.search"
  - "vestara.knowledge.search"
  - "vestara.filesystem.read"

input:
  - "Research question or topic"
  - "Scope and depth requirements"
  - "Sources to include/exclude"

output:
  - "Research report with citations"
  - "Key findings summary"
  - "Knowledge gaps identified"
  - "Recommendations"

rule: "Outputs research reports only. Never implements code."
```

### AGT-005: Knowledge Agent

```yaml
id: "AGT-005"
name: "Knowledge Agent"
purpose: "Manages the knowledge base — adds, indexes, organizes, and maintains knowledge."
status: "gen-1"

capabilities:
  - "Document ingestion and indexing"
  - "Automatic categorization and tagging"
  - "Duplicate detection"
  - "Knowledge graph construction (Gen 3)"
  - "Content summarization"
  - "Knowledge quality scoring"

default_tools:
  - "vestara.filesystem.read"
  - "vestara.knowledge.search"

events:
  consumes:
    - "filesystem:file.changed"  # Auto-index
    - "conversation:response.complete"  # Extract knowledge

output:
  - "Indexed knowledge entries"
  - "Content summaries"
  - "Knowledge quality reports"
```

### AGT-006: Memory Agent

```yaml
id: "AGT-006"
name: "Memory Agent"
purpose: "Manages memory consolidation, importance scoring, pruning, and retrieval optimization."
status: "gen-1"

capabilities:
  - "Memory importance scoring"
  - "Memory consolidation and summarization"
  - "Memory pruning and archiving"
  - "Cross-session memory linking (Gen 2)"
  - "Memory quality optimization"

default_tools: []
# Runs as a background process, not user-interactive

events:
  consumes:
    - "conversation:response.complete"
    - "conversation:message.sent"

schedule:
  - trigger: "every 50 interactions"
    task: "consolidate"
  - trigger: "daily"
    task: "prune_old_memories"
  - trigger: "weekly"
    task: "quality_report"

output:
  - "Consolidated memory summaries"
  - "Importance distribution reports"
  - "Pruning statistics"
```

### AGT-007: Coding Agent

```yaml
id: "AGT-007"
name: "Coding Agent"
purpose: "Generates, reviews, and refactors code. Understands project structure and patterns."
status: "gen-1"

capabilities:
  - "Code generation from specifications"
  - "Code review and quality analysis"
  - "Refactoring and optimization"
  - "Test generation"
  - "Documentation generation"
  - "Dependency management"

default_model: "opencode/deepseek-v4-flash-free"
default_tools:
  - "vestara.filesystem.read"
  - "vestara.filesystem.write"
  - "vestara.code.read"
  - "vestara.shell.execute"
  - "vestara.git.status"
  - "vestara.git.commit"

security:
  - "Write operations require user confirmation"
  - "Command execution sandboxed"
  - "Generated code scanned for security issues"
  - "No auto-commit without user approval"

output:
  - "Source code files"
  - "Test files"
  - "Documentation"
  - "Code review reports"
```

### AGT-008: Automation Agent

```yaml
id: "AGT-008"
name: "Automation Agent"
purpose: "Executes automated workflows, monitors triggers, manages scheduled tasks."
status: "gen-2"

capabilities:
  - "Workflow execution from definitions"
  - "Trigger monitoring (time, event, condition)"
  - "Step-by-step execution with state"
  - "Error handling and retry"
  - "Workflow status reporting"
  - "Parallel step execution"

default_tools:
  - All tools (scoped to workflow permissions)

events:
  consumes:
    - "*"  # Can be triggered by any event

output:
  - "Workflow execution results"
  - "Step completion status"
  - "Error reports with retry suggestions"
```

### AGT-009: Voice Agent

```yaml
id: "AGT-009"
name: "Voice Agent"
purpose: "Voice interaction — speech-to-text, text-to-speech, voice commands."
status: "gen-2"

capabilities:
  - "Speech-to-text (STT) — Web Speech API + Whisper fallback"
  - "Text-to-speech (TTS) — Web Speech API + ElevenLabs"
  - "Voice command parsing"
  - "Voice activity detection"
  - "Speaker identification (Gen 3)"

default_providers:
  stt: "browser/webspeech"
  tts: "browser/webspeech"

events:
  emits:
    - "voice:activated"
    - "voice:deactivated"
    - "voice:command.received"
    - "voice:response.started"
    - "voice:response.completed"
```

### AGT-010: Organization Agent

```yaml
id: "AGT-010"
name: "Organization Agent"
purpose: "Multi-user coordination, team knowledge, organizational memory, cross-project insights."
status: "gen-4"

capabilities:
  - "Cross-user memory consolidation (with permission)"
  - "Team knowledge base management"
  - "Organizational pattern detection"
  - "Cross-project dependency analysis"
  - "Team productivity insights"
  - "Onboarding assistance"

security:
  - "Requires organization admin consent"
  - "Cross-user data access is opt-in"
  - "All cross-user operations are audited"
```

---

## Agent Capability Matrix

| Agent | Gen | Model Need | Tools | Memory | Knowledge | Voice | Vision |
|-------|-----|------------|-------|--------|-----------|-------|--------|
| Conversation | 1 | Medium | Yes | R/W | Search | — | — |
| Planning | 2 | High | Read | Read | Search | — | — |
| Reasoning | 2 | High | Read | Read | Search | — | — |
| Research | 1 | Medium | Web | Read | R/W | — | — |
| Knowledge | 1 | Low | File | — | R/W | — | — |
| Memory | 1 | Low | — | R/W | — | — | — |
| Coding | 1 | High | Full | Read | Search | — | — |
| Automation | 2 | Medium | Full | Read | Read | — | — |
| Voice | 2 | Low | — | — | — | R/W | — |
| Organization | 4 | High | Full | Multi | Multi | — | — |

---

## Agent Execution Flow

```
User/Mission → AgentRuntime.execute(agentId, task)
                    ↓
          Agent receives task + context
                    ↓
          Agent loads memories + knowledge
                    ↓
          Agent plans execution steps
                    ↓
          For each step:
            ├─ Invoke tool (if needed)
            ├─ Wait for user approval (if permission required)
            └─ Process result
                    ↓
          Agent produces final result
                    ↓
          Agent emits execution.completed event
```

---

*The Agent Catalog defines every runtime agent before it is implemented. Each agent has a clear purpose, capabilities, tool access, and security model. New agents are added by creating their contract here first.*
