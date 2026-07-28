# Vestara Foundation
## The Core Language of the Vestara AI Operating Platform

> **This repository defines the foundational primitives, contracts, and architecture of the Vestara platform. Before any implementation, before any AI feature, before any user interface — the Foundation defines the language Vestara speaks.**

---

## 🏛️ Three-Layer Architecture (Complete)

```
vestara-blueprint/          ← WHY: Governance, vision, architecture, standards
vestara-specifications/     ← WHAT: Implementation specs, contracts, data models
vestara-foundation/         ← THE LANGUAGE: Core primitives, runtime, contracts
vestara-ai-core/            ← HOW: AI implementation (Phase 3)
```

## 📦 Foundation Contents

| Directory | Contents |
|-----------|----------|
| `object-model/` | **VOM** — Vestara Object Model: Every core primitive defined |
| `runtime-contracts/` | **VRA** — Vestara Runtime Architecture: What runs inside Vestara |
| `service-contracts/` | **Service Catalog** — Every service with API, events, SLA |
| `tool-catalog/` | **Tool Catalog** — Every tool with standard interface contract |
| `agent-catalog/` | **Agent Catalog** — Every runtime agent type defined |
| `provider-sdk/` | **Provider SDK** — Provider abstraction and integration contract |
| `plugin-sdk/` | **Plugin SDK** — Plugin architecture, manifest, lifecycle |
| `workflow-engine/` | **Workflow Engine** — Trigger → Condition → Plan → Execute |
| `mission-engine/` | **Mission Engine** — Long-lived mission objects across sessions |
| `interfaces/` | **Universal Interface** — Every runtime component's contract |
| `core-types/` | **Type System** — Foundational TypeScript types and schemas |
| `event-contracts/` | **Event Architecture** — Event publishing, subscription, routing |
| `capability-definitions/` | **Capability Model** — What a capability is and how it's defined |
| `shared-schemas/` | **Shared Zod Schemas** — Cross-cutting validation schemas |
| `protocols/` | **Communication Protocols** — Inter-component message formats |

---

## 🎯 Dependency Graph

```
Blueprint (WHY)
    ↓
Specifications (WHAT)
    ↓
Foundation (THE LANGUAGE)
    ├── Object Model
    ├── Runtime Architecture
    ├── Service Contracts
    ├── Agent/Provider/Plugin SDKs
    └── Universal Interface
        ↓
AI Core → Workspace → OS → Cloud → Applications
             (HOW)
```

**The Foundation is the stable base. Higher layers depend on it. It rarely changes.**

---

## 🔗 Relationship to Blueprint and Specs

| Foundation Concept | Blueprint Ref | Specs Ref |
|-------------------|---------------|-----------|
| Object Model (VOM) | `04-platform/PLATFORM_OVERVIEW.md` | `capabilities/INDEX.md` |
| Runtime Architecture (VRA) | `04-platform/01-platform-overview.md` | `events/EVENT-CATALOG.md` |
| Service Catalog | `04-platform/PLATFORM_OVERVIEW.md` | `apis/API-CONTRACT-INDEX.md` |
| Tool Catalog | `05-ai-core/agents/` | `ai-capabilities/AI-CONTRACT-INDEX.md` |
| Agent Catalog | `05-ai-core/AI_OVERVIEW.md` | `ai-capabilities/AI-CONTRACT-INDEX.md` |
| Provider SDK | `05-ai-core/providers/` | `ai-capabilities/AI-CONTRACT-INDEX.md` |
| Plugin SDK | `10-developer-platform/` | `apis/API-CONTRACT-INDEX.md` |
| Universal Interface | `14-engineering/ARCHITECTURE.md` | `validation/blueprint-validation-cli.md` |

---

*Vestara Foundation — Defining the language of the platform so every line of code speaks the same dialect.*
