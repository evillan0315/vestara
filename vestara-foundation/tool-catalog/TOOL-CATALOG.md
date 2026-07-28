---
id: "FND-004"
title: "Tool Catalog — Standard Tool Contracts"
owner: "@ai-engineer"
status: "ratified"
blueprint-ref: "05-ai-core/agents/"
foundation-version: "1.0.0"
---

# Tool Catalog
## Every Tool in the Vestara Platform — Standard Contract Before Implementation

> **Tools are how agents interact with the world. Every tool has a standard contract: parameters, returns, permissions, sandboxing, timeout, and security model. This catalog defines every tool before any agent uses it.**

---

## Tool Contract Template

```typescript
interface Tool {
  id: string;                              // Well-known string ID (e.g., "vestara.read_file")
  name: string;                            // Human-readable name
  description: string;                     // What the tool does
  version: string;                         // Semantic version
  
  parameters: JSONSchema;                  // JSON Schema for invocation parameters
  returns: JSONSchema;                     // JSON Schema for return value
  
  permissionLevel: 'read-only' | 'user-confirm' | 'admin-only';
  requires: string[];                      // Required capabilities (network, filesystem, etc.)
  timeout: number;                         // Max execution time in ms
  sandbox: boolean;                        // Isolated execution required
  retryOnTimeout: boolean;                 // Can be retried on timeout
  
  execute(params: Record<string, unknown>, context: ToolContext): Promise<ToolResult>;
  validate(params: Record<string, unknown>): ValidationResult;
  getSchema(): ToolSchema;
}
```

## Permission Levels

| Level | Behavior | Examples |
|-------|----------|---------|
| `read-only` | Executes automatically, no user prompt | `search_knowledge`, `read_file` |
| `user-confirm` | Requires explicit user approval | `write_file`, `execute_command` |
| `admin-only` | Requires admin role | `system.configure`, `user.impersonate` |

---

## 📋 Built-in Tools

### T-001: Filesystem — Read

```yaml
id: "vestara.filesystem.read"
name: "Read File"
description: "Read the contents of a file at the specified path"
version: "1.0.0"

parameters:
  type: object
  properties:
    path:
      type: string
      description: "Project-relative file path"
      example: "src/index.ts"
  required: ["path"]

returns:
  type: object
  properties:
    content: { type: string, description: "File contents" }
    path: { type: string, description: "Resolved absolute path" }
    size: { type: number, description: "File size in bytes" }
    mime_type: { type: string, description: "Detected MIME type" }

permissionLevel: "read-only"
requires: ["filesystem"]
timeout: 5000
sandbox: false

error_cases:
  - "File not found"
  - "Path traversal detected"
  - "Binary file (size exceeds limit)"
  - "Permission denied"
```

### T-002: Filesystem — Write

```yaml
id: "vestara.filesystem.write"
name: "Write File"
description: "Write content to a file at the specified path (creates directories if needed)"
version: "1.0.0"

parameters:
  type: object
  properties:
    path: { type: string, description: "Project-relative file path" }
    content: { type: string, description: "File content to write" }
    mode: { type: string, enum: [write, append], default: "write" }
  required: ["path", "content"]

returns:
  type: object
  properties:
    path: { type: string, description: "Written file path" }
    size: { type: number, description: "Written bytes" }
    created: { type: boolean, description: "Whether file was created new" }

permissionLevel: "user-confirm"
requires: ["filesystem"]
timeout: 5000
sandbox: true

error_cases:
  - "Path traversal detected"
  - "Disk full"
  - "Permission denied"
  - "File locked by another process"
```

### T-003: Shell — Execute

```yaml
id: "vestara.shell.execute"
name: "Execute Command"
description: "Execute a shell command and capture output"
version: "1.0.0"

parameters:
  type: object
  properties:
    command: { type: string, description: "Shell command to execute" }
    working_directory: { type: string, description: "Working directory (default: project root)" }
    timeout: { type: number, description: "Custom timeout in ms", maximum: 60000 }
  required: ["command"]

returns:
  type: object
  properties:
    stdout: { type: string, description: "Standard output" }
    stderr: { type: string, description: "Standard error" }
    exit_code: { type: number, description: "Process exit code" }
    duration: { type: number, description: "Execution duration in ms" }

permissionLevel: "user-confirm"
requires: ["shell", "sandbox"]
timeout: 30000
sandbox: true
max_timeout: 60000

restricted_commands:
  - "rm -rf /"
  - "sudo"
  - ":(){ :|:& };:"  # Fork bomb
  - "dd if=/dev/..."
  - "mkfs.*"
  - "> /dev/sd*"

error_cases:
  - "Command not found"
  - "Timeout exceeded"
  - "Permission denied"
  - "Forbidden command detected"
```

### T-004: Knowledge — Search

```yaml
id: "vestara.knowledge.search"
name: "Search Knowledge"
description: "Search the knowledge base using full-text and semantic search"
version: "1.0.0"

parameters:
  type: object
  properties:
    query: { type: string, description: "Search query" }
    project_id: { type: string, description: "Scope search to project (optional)" }
    type: { type: string, enum: [document, note, code, reference], description: "Filter by type (optional)" }
    limit: { type: number, default: 10, maximum: 50 }
  required: ["query"]

returns:
  type: object
  properties:
    results:
      type: array
      items:
        type: object
        properties:
          id: { type: string }
          title: { type: string }
          content: { type: string, description: "Snippet with highlights" }
          type: { type: string }
          score: { type: number, description: "Relevance score 0-1" }
    total: { type: number, description: "Total results" }

permissionLevel: "read-only"
requires: ["knowledge"]
timeout: 10000
sandbox: false
```

### T-005: Memory — Search

```yaml
id: "vestara.memory.search"
name: "Search Memory"
description: "Search user memories using full-text and importance ranking"
version: "1.0.0"

parameters:
  type: object
  properties:
    query: { type: string, description: "Search query" }
    type: { type: string, enum: [fact, preference, event, decision], description: "Filter by type (optional)" }
    limit: { type: number, default: 20, maximum: 100 }
  required: ["query"]

returns:
  type: object
  properties:
    results:
      type: array
      items:
        type: object
        properties:
          id: { type: string }
          content: { type: string }
          type: { type: string }
          importance: { type: number, description: "0-10 importance score" }
          score: { type: number, description: "Search relevance 0-1" }
    total: { type: number }

permissionLevel: "read-only"
requires: ["memory"]
timeout: 5000
sandbox: false
```

### T-006: Web — Search

```yaml
id: "vestara.web.search"
name: "Web Search"
description: "Search the web for current information"
version: "1.0.0"

parameters:
  type: object
  properties:
    query: { type: string, description: "Web search query" }
    num_results: { type: number, default: 5, maximum: 10 }
  required: ["query"]

returns:
  type: object
  properties:
    results:
      type: array
      items:
        type: object
        properties:
          title: { type: string }
          url: { type: string }
          snippet: { type: string }
          source: { type: string }

permissionLevel: "user-confirm"
requires: ["network"]
timeout: 15000
sandbox: false
```

### T-007: Task — Create

```yaml
id: "vestara.task.create"
name: "Create Task"
description: "Create a task in the current project"
version: "1.0.0"

parameters:
  type: object
  properties:
    title: { type: string, description: "Task title", maxLength: 500 }
    description: { type: string, description: "Task description", maxLength: 10000 }
    status: { type: string, enum: [todo, in_progress, review, done], default: "todo" }
    tags: { type: array, items: { type: string }, maxItems: 10 }
    estimated_hours: { type: number, minimum: 0, maximum: 10000 }
  required: ["title"]

returns:
  type: object
  properties:
    id: { type: string }
    title: { type: string }
    status: { type: string }
    created_at: { type: string }

permissionLevel: "user-confirm"
requires: ["project"]
timeout: 5000
sandbox: false
```

### T-008: Code — Read

```yaml
id: "vestara.code.read"
name: "Read Code"
description: "Read and analyze source code with syntax awareness"
version: "1.0.0"

parameters:
  type: object
  properties:
    path: { type: string, description: "File path or directory to explore" }
    include_patterns: { type: array, items: { type: string }, description: "Glob patterns to include" }
    line_start: { type: number, description: "Start line (1-indexed)" }
    line_end: { type: number, description: "End line" }
  required: ["path"]

returns:
  type: object
  properties:
    content: { type: string }
    language: { type: string, description: "Detected programming language" }
    line_count: { type: number }
    size: { type: number }

permissionLevel: "read-only"
requires: ["filesystem"]
timeout: 5000
sandbox: false
```

### T-009: Git — Status

```yaml
id: "vestara.git.status"
name: "Git Status"
description: "Get the current git repository status"
version: "1.0.0"

parameters:
  type: object
  properties:
    path: { type: string, description: "Repository path (default: project root)" }
  required: []

returns:
  type: object
  properties:
    branch: { type: string }
    changes: { type: number }
    staged: { type: number }
    untracked: { type: number }
    ahead: { type: number, description: "Commits ahead of remote" }
    behind: { type: number, description: "Commits behind remote" }

permissionLevel: "read-only"
requires: ["filesystem", "git"]
timeout: 5000
sandbox: false
```

### T-010: Git — Commit

```yaml
id: "vestara.git.commit"
name: "Git Commit"
description: "Stage and commit changes to the git repository"
version: "1.0.0"

parameters:
  type: object
  properties:
    message: { type: string, description: "Commit message", minLength: 1 }
    files: { type: array, items: { type: string }, description: "Files to stage (default: all)" }
    allow_empty: { type: boolean, default: false }
  required: ["message"]

returns:
  type: object
  properties:
    commit_hash: { type: string }
    files_changed: { type: number }
    insertions: { type: number }
    deletions: { type: number }

permissionLevel: "user-confirm"
requires: ["filesystem", "git"]
timeout: 10000
sandbox: true
```

---

## Tool Permission Groups

| Group | Tools | Default Permission |
|-------|-------|-------------------|
| **Read** | read_file, search_knowledge, search_memory, read_code, git_status | Auto-approve |
| **Write** | write_file, git_commit, task_create | User-confirm |
| **Dangerous** | execute_command | User-confirm (with highlighted warning) |
| **External** | web_search | User-confirm (first use) |

---

## Tool Registration Flow

```
Plugin / Built-in → Register Tool → Tool Runtime
                                         ↓
Agent Runtime ← Tool Registry ← Tool Catalog
                         ↓
              Permission Service ← User Approval UI
```

---

## Tool Security Model

| Threat | Mitigation |
|--------|------------|
| Path traversal | Path validation (reject `/../`), sandbox to project root |
| Command injection | Parameterized execution, no shell injection possible |
| Resource exhaustion | Per-tool timeout, memory limit, concurrent execution limit |
| Data exfiltration | Output inspection, rate limiting, audit logging |
| Unauthorized file access | Permission levels, user confirmation for writes |
| Malicious plugins | Sandboxed execution, capability-based permissions |

---

*The Tool Catalog defines every tool before it is implemented. No tool is registered without a contract. Every tool has the same interface, the same security model, and the same execution path.*
