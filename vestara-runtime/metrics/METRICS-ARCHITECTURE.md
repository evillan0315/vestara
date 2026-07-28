---
id: "RT-004"
title: "Metrics Architecture — Everything Observable"
owner: "@devops-engineer"
status: "ratified"
blueprint-ref: "15-devops/OBSERVABILITY.md"
runtime-version: "1.0.0"
---

# Metrics Architecture
## Every Service Reports — CPU, Memory, Latency, Errors, Tokens, Cost, Health

> **Every component in Vestara emits metrics by default. No opt-in. Metrics are collected, aggregated, and exported for dashboards, alerting, and capacity planning. A platform with 250 plugins and 120 agents cannot be debugged by guessing.**

---

## Metrics Pipeline

```
Service Code
    ↓
Metrics API (counter, gauge, histogram)
    ↓
Metrics Registry (all metrics declared here)
    ↓
Aggregation Pipeline (rate, percentile, max)
    ↓
Export Interface
    ├── In-memory (for diagnostics API)
    ├── Prometheus (pull endpoint)
    ├── Event Bus (real-time dashboard)
    └── Filesystem (for diagnostics)
```

---

## Metric Types

```typescript
interface MetricsCollector {
  /** Monotonically increasing counter */
  increment(name: string, value?: number, tags?: TagMap): void;
  
  /** Point-in-time value */
  gauge(name: string, value: number, tags?: TagMap): void;
  
  /** Distribution of values (p50, p95, p99, max) */
  record(name: string, value: number, tags?: TagMap): void;
  
  /** Time a function execution */
  time<T>(name: string, fn: () => Promise<T>, tags?: TagMap): Promise<T>;
  
  /** Declare a metric for documentation */
  declare(definition: MetricDefinition): void;
  
  /** Create child collector with default tags */
  child(tags: TagMap): MetricsCollector;
  
  /** Get snapshot of all metrics */
  snapshot(): MetricsSnapshot;
}

type TagMap = Record<string, string>;
```

---

## Standard Metrics

Every service reports these metrics automatically:

### Resource Metrics (Auto-collected by Runtime)

| Metric | Type | Tags | Description |
|--------|------|------|-------------|
| `vestara.runtime.cpu` | gauge | `service` | CPU usage % |
| `vestara.runtime.memory` | gauge | `service` | Memory usage bytes |
| `vestara.runtime.memory.percent` | gauge | `service` | Heap usage % |
| `vestara.runtime.gc.duration` | histogram | `service` | GC pause time ms |
| `vestara.runtime.uptime` | gauge | `service` | Uptime seconds |

### Service Metrics (Every Service)

| Metric | Type | Tags | Description |
|--------|------|------|-------------|
| `vestara.service.status` | gauge | `service` | 1=healthy, 0=degraded, -1=down |
| `vestara.service.requests.total` | counter | `service,method` | Total requests |
| `vestara.service.requests.active` | gauge | `service` | Currently active |
| `vestara.service.requests.duration` | histogram | `service,method` | Request latency ms |
| `vestara.service.requests.errors` | counter | `service,method,error` | Error count |
| `vestara.service.requests.rate` | gauge | `service` | Requests/sec |

### AI-Specific Metrics (AI Core)

| Metric | Type | Tags | Description |
|--------|------|------|-------------|
| `vestara.ai.tokens.input` | counter | `provider,model` | Input tokens |
| `vestara.ai.tokens.output` | counter | `provider,model` | Output tokens |
| `vestara.ai.tokens.total` | counter | `provider,model` | Total tokens |
| `vestara.ai.cost` | counter | `provider,model` | Cost in USD cents |
| `vestara.ai.latency.ttft` | histogram | `provider,model` | Time to first token ms |
| `vestara.ai.latency.total` | histogram | `provider,model` | Total response time ms |
| `vestara.ai.requests.active` | gauge | `provider` | Active AI requests |
| `vestara.ai.provider.health` | gauge | `provider` | 1=healthy, 0=down |

### Agent Metrics (Agent Runtime)

| Metric | Type | Tags | Description |
|--------|------|------|-------------|
| `vestara.agent.active` | gauge | `agent` | Active agents |
| `vestara.agent.executions.total` | counter | `agent` | Total executions |
| `vestara.agent.executions.active` | gauge | `agent` | Currently executing |
| `vestara.agent.executions.duration` | histogram | `agent,status` | Execution duration ms |
| `vestara.agent.tools.invoked` | counter | `agent,tool` | Tool invocations |
| `vestara.agent.tools.errors` | counter | `agent,tool` | Tool errors |

### Plugin Metrics (Plugin Runtime)

| Metric | Type | Tags | Description |
|--------|------|------|-------------|
| `vestara.plugin.active` | gauge | `plugin` | Active plugins |
| `vestara.plugin.executions.total` | counter | `plugin` | Tool/command invocations |
| `vestara.plugin.executions.duration` | histogram | `plugin` | Execution duration ms |
| `vestara.plugin.errors` | counter | `plugin,error` | Error count |

### Business Metrics (Application)

| Metric | Type | Tags | Description |
|--------|------|------|-------------|
| `vestara.users.active` | gauge | `tier` | Active users |
| `vestara.projects.created` | counter | — | Projects created |
| `vestara.tasks.completed` | counter | `project` | Tasks completed |
| `vestara.memories.stored` | counter | — | Memories stored |
| `vestara.conversations.started` | counter | — | Conversations started |
| `vestara.missions.active` | gauge | — | Active missions |

---

## Metrics Export

```yaml
prometheus:
  endpoint: "/api/v1/metrics"
  format: "Prometheus text format"
  enabled: true
  scrape_interval: "15s"

event_bus:
  enabled: true
  interval: "60s"
  event_type: "metrics:snapshot"

diagnostics_api:
  enabled: true
  max_points: 1000
  retention: "1 hour"
```

---

## Metrics Dashboard Specification

```yaml
dashboard:
  name: "Vestara Platform Overview"
  
  panels:
    - title: "Service Health"
      metrics: ["vestara.service.status"]
      type: "status-grid"
    
    - title: "Active AI Requests"
      metrics: ["vestara.ai.requests.active"]
      type: "time-series"
    
    - title: "Token Usage (7d)"
      metrics: ["vestara.ai.tokens.total"]
      type: "area"
      breakdown: "provider"
    
    - title: "AI Cost (7d)"
      metrics: ["vestara.ai.cost"]
      type: "area"
      breakdown: "provider"
    
    - title: "API Latency (p95)"
      metrics: ["vestara.service.requests.duration"]
      type: "heatmap"
      breakdown: "service"
    
    - title: "Error Rate"
      metrics: ["vestara.service.requests.errors"]
      type: "time-series"
      breakdown: "service"
    
    - title: "Active Users"
      metrics: ["vestara.users.active"]
      type: "gauge"
    
    - title: "Active Missions"
      metrics: ["vestara.missions.active"]
      type: "gauge"
```

---

## Alerting Thresholds

| Condition | Severity | Action |
|-----------|----------|--------|
| Service status = 0 (degraded) | Warning | Notify team |
| Service status = -1 (down) | Critical | Pager, auto-recovery |
| Error rate > 5% over 5min | Warning | Notify team |
| Error rate > 10% over 5min | Critical | Pager |
| p95 latency > 2x baseline | Warning | Investigate |
| Memory > 90% | Warning | Scale or investigate |
| Monthly AI cost > 80% budget | Warning | Notify admin |
| Provider unavailable > 5min | Critical | Failover |

---

*The Metrics Architecture ensures every component produces measurable output. No guessing. No "it worked on my machine." Every behavior is quantified.*
