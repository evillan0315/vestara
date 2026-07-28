---
id: "RT-003"
title: "Logging Architecture — Structured, Observable, Diagnosable"
owner: "@devops-engineer"
status: "ratified"
blueprint-ref: "15-devops/LOGGING.md"
runtime-version: "1.0.0"
---

# Logging Architecture
## Not console.log — Real Structured Logging for a Production Platform

> **Every component in Vestara logs through a structured logging pipeline. No exceptions. Logs are structured (JSON), level-controlled, sink-routed, and queryable — from development laptops to enterprise deployments.**

---

## Logging Pipeline

```
Application Code
    ↓
Structured Logger (pino / winston)
    ↓
Level Manager (debug < info < warn < error < fatal)
    ↓
Enricher (add service, request, correlation IDs)
    ↓
Sink Router
    ├── Console (development, always on)
    ├── File (production, rotated)
    ├── Event Bus (diagnostics panel)
    └── Network (future: Loki, ELK, DataDog)
```

---

## Log Format

Every log entry is structured JSON:

```json
{
  "timestamp": "2025-07-23T12:00:00.000Z",
  "level": "info",
  "message": "Service started successfully",
  "service": {
    "id": "memory-service",
    "version": "1.0.0"
  },
  "context": {
    "requestId": "req-abc123",
    "correlationId": "cor-xyz789",
    "userId": "user-456",
    "component": "MemoryConsolidation"
  },
  "error": null,
  "metrics": {
    "duration": 145,
    "itemsProcessed": 250
  }
}
```

---

## Log Levels

| Level | Value | Usage | Production |
|-------|-------|-------|------------|
| `fatal` | 0 | Unrecoverable error, system will shut down | ✅ Always |
| `error` | 1 | Recoverable error, degraded functionality | ✅ Always |
| `warn` | 2 | Unexpected but handled, potential issue | ✅ Always |
| `info` | 3 | Normal operational messages | ✅ Default |
| `debug` | 4 | Detailed diagnostic information | ❌ Toggle on |
| `trace` | 5 | Extremely verbose, per-function entry/exit | ❌ Debug only |

---

## Logger Interface

```typescript
interface Logger {
  fatal(message: string, context?: LogContext): void;
  error(message: string, context?: LogContext & { error?: Error }): void;
  warn(message: string, context?: LogContext): void;
  info(message: string, context?: LogContext): void;
  debug(message: string, context?: LogContext): void;
  trace(message: string, context?: LogContext): void;

  /** Create child logger with inherited context */
  child(context: LogContext): Logger;

  /** Set minimum log level at runtime */
  setLevel(level: LogLevel): void;

  /** Flush pending log entries */
  flush(): Promise<void>;

  /** Get logger metrics */
  getMetrics(): LoggerMetrics;
}

interface LogContext {
  requestId?: string;
  correlationId?: string;
  userId?: string;
  component?: string;
  [key: string]: unknown;
}

type LogLevel = 'fatal' | 'error' | 'warn' | 'info' | 'debug' | 'trace';
```

---

## Log Enrichment

Every log entry is automatically enriched with:

| Field | Source | Example |
|-------|--------|---------|
| `service.id` | Kernel Service Registry | `memory-service` |
| `service.version` | Service manifest | `1.2.3` |
| `host.name` | OS | `vestara-dev-01` |
| `host.platform` | OS | `linux-x64` |
| `runtime.version` | Kernel | `1.0.0` |
| `trace.id` | AsyncLocalStorage | `trace-abc` |
| `span.id` | AsyncLocalStorage | `span-def` |

---

## Log Sinks

| Sink | Environment | Retention | Configuration |
|------|-------------|-----------|---------------|
| **Console** | Development | N/A | Always on, pretty-print |
| **File** | Production | 30 days, 1GB max | Path, rotation schedule |
| **Event Bus** | All | Last 1000 in memory | On by default for diagnostics |
| **Network** | Enterprise | External system dependent | URL, API key, buffer size |

---

## Log Security Rules

| Rule | Enforcement |
|------|-------------|
| **No secrets in logs** | Automatic PII/secret redaction |
| **No user content in error logs** | User data redacted, ID only |
| **Audit logs are immutable** | Append-only, signed |
| **Log level is runtime-configurable** | No restart needed |
| **Production default level: info** | debug/trace only on-demand |

---

## Logger Metrics

```typescript
interface LoggerMetrics {
  totalLogged: number;
  byLevel: Record<LogLevel, number>;
  errorsLastMinute: number;
  avgLogLatency: number;    // ms
  sinkStatus: Record<string, 'healthy' | 'degraded' | 'failed'>;
  bufferUsage: number;       // % of buffer
}
```

---

## Log Query API

```typescript
interface LogQueryService {
  /** Query logs with filters */
  query(filters: LogFilters): Promise<LogEntry[]>;
  
  /** Stream logs in real-time */
  stream(filters?: LogFilters): AsyncIterable<LogEntry>;
  
  /** Get log statistics */
  stats(timeRange: TimeRange): Promise<LogStats>;
}

interface LogFilters {
  level?: LogLevel | LogLevel[];
  service?: string | string[];
  userId?: string;
  requestId?: string;
  correlationId?: string;
  timeRange?: TimeRange;
  search?: string;
  limit?: number;
  offset?: number;
}
```

---

*The Logging Architecture ensures every line of code produces observable output. No console.log. No silent failures. Every component is diagnosable.*
