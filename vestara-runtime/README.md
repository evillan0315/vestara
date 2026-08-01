# Vestara Runtime

OS-0 host integration is implemented in the Core repository at `579df3f`.
Linux/systemd provide the machine plane; Host Runtime supplies read-only machine
inspection and Boot Runtime supplies durable ordered boot evidence. See
`kernel/VESTARA-KERNEL.md`. This milestone does not produce a bootable image.

## The Execution Engine — Infrastructure That Never Cares About AI

> **This repository defines the runtime infrastructure of the Vestara platform. The Kernel, lifecycle, configuration, logging, metrics, diagnostics — everything that makes the platform run but has nothing to do with AI features. The Runtime is a stable platform that could host any services, not just AI.**

---

## 🏛️ Evolutionary Role

```
vestara-blueprint/       ← WHY: Governance, vision, architecture
vestara-specifications/  ← WHAT: Implementation contracts
vestara-foundation/      ← THE LANGUAGE: Core primitives
vestara-runtime/         ← THE ENGINE: Infrastructure, lifecycle, observability
vestara-ai-core/         ← INTELLIGENCE: AI features (Phase 3 — follows)
vestara-workspace/       ← UI: User experience
vestara-os/              ← OS: Operating system
vestara-cloud/           ← CLOUD: Distributed services
```

**AI Core should never care how services start, how logs work, or how configuration is loaded. Those belong here — in the Runtime.**

---

## 📦 Runtime Contents

| Directory | Contents |
|-----------|----------|
| `kernel/` | **Vestara Kernel** — Boot manager, configuration, lifecycle orchestrator |
| `boot/` | **Boot Sequence** — Complete startup/shutdown specification |
| `lifecycle/` | **Lifecycle Specifications** — Service, Agent, Plugin, Mission state machines |
| `scheduler/` | **Scheduler** — Task scheduling, cron, intervals |
| `execution/` | **Execution Engine** — Sandbox, resource limits, timeouts |
| `dependency-injection/` | **DI Container** — Service graph, resolution, scopes |
| `configuration/` | **Configuration Manager** — Sources, validation, hot-reload |
| `logging/` | **Logging Architecture** — Structured logging, levels, sinks |
| `metrics/` | **Metrics Architecture** — Collection, aggregation, export |
| `tracing/` | **Distributed Tracing** — Request tracing across services |
| `observability/` | **Observability** — Health, readiness, liveness, dashboard |
| `state/` | **State Management** — In-memory, persistent, distributed |
| `health/` | **Health Monitoring** — Checks, circuit breakers, thresholds |
| `diagnostics/` | **Diagnostics / Vestara Doctor** — System health inspection |
| `recovery/` | **Recovery** — Crash recovery, state restoration, graceful degradation |
| `architecture-v1.0/` | **Architecture Freeze** — Vestara Architecture v1.0 declaration |

---

## 🔗 Dependency Graph (Immutable)

```
Kernel
  ↓
Runtime
  ↓
Foundation
  ↓
Services
  ↓
Capabilities
  ↓
Workspace
  ↓
Applications
```

**Rule**: Dependencies flow downward. Never: `Workspace → Kernel`.

---

## 🎯 Runtime Principles

| Principle | Description |
|-----------|-------------|
| **Observability by Default** | Every component emits metrics, logs, traces without opt-in |
| **Lifecycle Governance** | Every component follows the same lifecycle state machine |
| **Fail Fast, Fail Clean** | Errors are detected at boot, not at runtime |
| **Graceful Degradation** | Partial failures don't cascade — non-critical services degrade independently |
| **Configuration as Code** | All configuration is validated, versioned, auditable |
| **Self-Healing** | The Runtime detects and recovers from common failure modes automatically |
| **Hot-Reload Capable** | Configuration and plugins can be updated without restart |
| **Deterministic Startup** | Boot order is explicit, dependency-driven, and repeatable |

---

*Vestara Runtime — The execution engine that makes everything else possible. AI Core will be a client of this Runtime, not an owner of infrastructure.*
