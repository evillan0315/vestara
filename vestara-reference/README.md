# Vestara Reference
## Canonical Implementation Examples for Every Contract

> **Documentation tells people what to build. Reference implementations show them how.**

---

## Purpose

`vestara-reference` contains canonical implementations of every contract defined in the Vestara architecture. Each example is self-contained, compilable, and traceable to the specific contract it implements.

## Contents

| Directory | Contract Reference | Example |
|-----------|-------------------|---------|
| `providers/` | Provider SDK (FND-007) | Building a custom AI provider |
| `plugins/` | Plugin SDK (FND-008) | Creating a Vestara plugin |
| `tools/` | Tool Catalog (FND-004) | Implementing a custom tool |
| `services/` | Service Catalog (FND-003) | Creating a VestaraService |
| `agents/` | Agent Catalog (FND-005) | Building a runtime agent |
| `memory/` | Memory Engine (AI-CON-001) | Memory integration patterns |
| `knowledge/` | Knowledge Engine (AI-CON-002) | Knowledge ingestion examples |
| `reasoning/` | Reasoning Runtime (BRAIN-4) | Custom reasoning strategies |
| `events/` | Event Catalog (SPEC-EVT-000) | Event-driven patterns |
| `workspace/` | Workspace API (PLATFORM-API) | Workspace integration examples |

## How to Use

1. Find the contract you want to implement
2. Open the corresponding directory
3. Read the canonical example
4. Use it as a starting point for your implementation

## Traceability

Every example includes a header tracing back to the specific contract:

```typescript
/**
 * Architecture Traceability:
 *   Foundation: PROVIDER-SDK.md → AIProvider
 *   Specification: AI-CON-004 → Provider Manager
 *   Runtime: LIFECYCLE-SPECIFICATION.md → Provider Lifecycle
 */
```
