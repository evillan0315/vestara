# The Vestara Blueprint

> **The complete architectural, business, and operational blueprint for the Vestara AI Platform.**

---

## 📚 Blueprint Structure

This repository contains **24 volumes** organized into **6 Books**, forming the complete specification for Vestara AI OS — from vision to implementation to future technologies.

```text
vestara-blueprint/
│
├── 📖 BOOK 1: Vision & Business
│   ├── 00-governance/         # Governance, constitution, decision-making
│   ├── 01-company/            # Company identity, mission, values, culture
│   ├── 02-business/           # Business model, market, pricing, metrics
│   └── 03-product/            # Product strategy, roadmap, positioning
│
├── 📖 BOOK 2: Platform Architecture
│   ├── 04-platform/           # Platform architecture, layers, modules
│   ├── 06-workspace/          # Workspace, projects, knowledge, memory
│   ├── 10-developer-platform/ # SDK, plugins, marketplace, APIs
│   └── 12-data/               # Data architecture, sync, privacy, ownership
│
├── 📖 BOOK 3: AI Architecture
│   ├── 05-ai-core/            # Core AI: conversation, memory, planning, reasoning
│   ├── 08-cloud/              # Cloud AI, distributed inference, swarms
│   ├── 14-conversation/       # Conversation runtime, agent collaboration, voice
│   └── 21-research/           # AI research, evaluation, safety, future
│
├── 📖 BOOK 4: Engineering
│   ├── 13-design-system/      # VDS — visual language, tokens, components, accessibility
│   ├── 14-engineering/        # Engineering standards, practices, conventions
│   └── 15-devops/             # Build, deploy, CI/CD, infrastructure
│
├── 📖 BOOK 5: Operations
│   ├── 07-operating-system/   # Immutable OS, A/B, Secure Boot, portable SSD
│   ├── 09-mobile/             # Mobile companion, sync, offline-first
│   ├── 11-security/           # Security model, threat model, compliance
│   └── 16-operations/         # Operations, support, incident response
│
├── 📖 BOOK 6: Future Technologies
│   ├── 17-marketing/          # Brand, positioning, community, content
│   ├── 18-sales/              # Sales motion, enterprise, partnerships
│   ├── 19-investor/           # Fundraising, metrics, board, governance
│   ├── 20-roadmaps/           # Generation roadmaps, milestones, horizons
│   └── 99-appendix/           # Glossary, references, archives, decisions
```

---

## ⚡ Important Notes

- **Implementation lives in `vestara-ai-core/`** — this repository contains the frozen architecture specification
- **Development methodology**: Vestara Specification-Driven Engineering (VSDE) — see `vestara-ai-core/docs/VSDE/`
- **Architecture is frozen** per ADR-016 — no changes to architecture documents without a new ADR

---

## 🎯 Quick Navigation

| Need | Go To |
|------|-------|
| **Understand the Vision** | [01-company/01-mission-vision-values.md](01-company/01-mission-vision-values.md) |
| **Understand the Architecture** | [04-platform/01-platform-overview.md](04-platform/01-platform-overview.md) |
| **Understand the AI** | [05-ai-core/01-ai-architecture-overview.md](05-ai-core/01-ai-architecture-overview.md) |
| **Engineering Standards** | [14-engineering/01-engineering-standards.md](14-engineering/01-engineering-standards.md) |
| **OS Architecture** | [07-operating-system/01-os-architecture.md](07-operating-system/01-os-architecture.md) |
| **Roadmaps** | [20-roadmaps/01-generation-roadmap.md](20-roadmaps/01-generation-roadmap.md) |
| **AI Development Workflow** | [00-governance/03-ai-development-lifecycle.md](00-governance/03-ai-development-lifecycle.md) |
| **Security Model** | [11-security/01-security-architecture.md](11-security/01-security-architecture.md) |

---

## 🏛️ Governance

This blueprint is the **supreme authority** for all Vestara decisions.

| Document | Purpose |
|----------|---------|
| [AI Constitution](00-governance/01-ai-constitution.md) | Master prompt for all AI agents |
| [Engineering Rules](00-governance/02-engineering-rules.md) | Non-negotiable engineering standards |
| [AI Development Lifecycle](00-governance/03-ai-development-lifecycle.md) | AIDL — AI Development Lifecycle |
| [Decision Log](00-governance/04-decision-log.md) | Architectural Decision Records (ADRs) |
| [Compatibility](00-governance/05-compatibility.md) | AI agent compatibility (Claude, OpenCode, etc.) |
| VSDE Standard | `vestara-ai-core/docs/standards/VSDE.md` — Specification-Driven Engineering methodology |

---

## 📖 The Six Books

### Book 1: Vision & Business (Volumes 00–03)
*Why Vestara exists, what it stands for, how it wins.*

- **00-governance** — Constitution, rules, decision-making, AI agent roles
- **01-company** — Mission, vision, values, founder story, culture
- **02-business** — Business model, pricing, market, metrics, competitive advantage
- **03-product** — Product strategy, positioning, roadmap, feature framework

### Book 2: Platform Architecture (Volumes 04, 06, 10, 12)
*How the platform fits together — layers, modules, data, extensibility.*

- **04-platform** — Platform layers, module registry, dependencies, APIs
- **06-workspace** — Projects, knowledge, memory, filesystem, sync
- **10-developer-platform** — SDK, plugins, marketplace, API gateway
- **12-data** — Data ownership, privacy, sync, local-first, encryption

### Book 3: AI Architecture (Volumes 05, 08, 14, 21)
*The brain of Vestara — every AI subsystem specified.*

- **05-ai-core** — Memory, Planning, Reasoning, Knowledge, Agents, Providers, Safety, Evaluation
- **08-cloud** — Distributed inference, swarms, cloud workers, edge-cloud hybrid
- **14-conversation** — Conversation runtime, agent collaboration, group chat, voice, streaming, artifacts
- **21-research** — Model eval, prompt engineering, safety, emerging capabilities

### Book 4: Engineering (Volumes 13, 14, 15)
*How we build — standards, practices, tooling, quality.*

- **13-design-system** — VDS: visual language, design tokens, components, accessibility, theme engine
- **14-engineering** — TypeScript, React, Fastify, SQLite, testing, linting, architecture
- **15-devops** — Docker, CI/CD, releases, monitoring, portable builds, SSD images

> **Note**: Volume `13-user-experience` is superseded by `13-design-system` (VDS). The design system is the canonical visual specification. UX patterns are now covered across VDS and the conversation/workspace volumes.

### Book 5: Operations (Volumes 07, 09, 11, 16)
*How Vestara runs — OS, mobile, security, operations.*

- **07-operating-system** — Immutable A/B OS, Secure Boot, verified boot, SSD builder
- **09-mobile** — Companion app, sync, offline-first, push notifications
- **11-security** — Threat model, encryption, keys, supply chain, audits, compliance
- **16-operations** — Incident response, support, SLOs, disaster recovery, on-call

### Book 6: Future Technologies (Volumes 17, 18, 19, 20, 99)
*Where Vestara goes next — business, research, horizons.*

- **17-marketing** — Brand, positioning, content, community, developer relations
- **18-sales** — Enterprise sales, partnerships, channel, pricing, contracts
- **19-investor** — Fundraising, board, metrics, governance, cap table
- **20-roadmaps** — Generation 1–5 roadmaps, milestones, capability maps
- **99-appendix** — Glossary, references, decisions archive, research notes

---

## 🔄 How to Use This Blueprint

### For AI Agents (Claude, OpenCode, Codex, Cursor, Copilot, etc.)

```markdown
1. READ: 00-governance/01-ai-constitution.md  (THIS IS YOUR MASTER PROMPT)
2. READ: 00-governance/02-engineering-rules.md
3. READ: 00-governance/03-ai-development-lifecycle.md
4. READ: 00-governance/04-decision-log.md
5. READ: The relevant volume for your task
6. FOLLOW: The AIDL workflow for your assigned phase
7. UPDATE: 00-governance/04-decision-log.md with any architectural decisions
```

### For Human Engineers

```markdown
1. READ: 14-engineering/01-engineering-standards.md
2. READ: 04-platform/01-platform-overview.md
3. READ: The relevant module specification in 04-platform/modules/
4. FOLLOW: Conventional commits, type-safe TypeScript, Zod validation
5. RUN: pnpm lint && pnpm typecheck && pnpm build && pnpm test before commit
```

### For Product & Leadership

```markdown
1. READ: 01-company/01-mission-vision-values.md
2. READ: 02-business/01-business-model.md
3. READ: 20-roadmaps/01-generation-roadmap.md
4. REFERENCE: 03-product/ for feature prioritization framework
```

---

## 📋 Document Standards

Every document in this blueprint follows:

| Standard | Requirement |
|----------|-------------|
| **Format** | Markdown with frontmatter metadata |
| **Versioning** | Semantic version in frontmatter (v1.0.0) |
| **Ownership** | `owner: @team/role` in frontmatter |
| **Status** | `status: draft | review | approved | deprecated` |
| **Links** | Relative links only, no external dependencies |
| **Diagrams** | Mermaid.js for architecture diagrams |
| **Code** | TypeScript snippets with `typescript` fence |
| **Updates** | `last-reviewed` and `next-review` dates |

**Frontmatter Template:**
```markdown
---
title: "Document Title"
volume: "XX-volume-name"
book: "Book N: Title"
version: "1.0.0"
status: "approved"
owner: "@team/role"
last-reviewed: "2025-01-15"
next-review: "2025-07-15"
tags: ["tag1", "tag2"]
---
```

---

## 🔗 Cross-References

| Source | References |
|--------|------------|
| AGENTS.md (root) | This blueprint — AI instructions |
| CLAUDE.md | This blueprint — Claude-specific |
| blueprints/00-07 | Superseded by this blueprint |
| services/*/src | Implementation follows these specs |
| apps/dashboard/src | UI implements VDS (13-design-system) specs |
| packages/conversation-runtime | Implements 14-conversation runtime spec |
| packages/* | Shared types follow 14-engineering standards |

---

## 📈 Version History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 2.0.0 | 2025-07-30 | Chief Architect | Promoted VDS to 13-design-system, Conversation to 14-conversation; added 11-language governance doc |
| 1.0.0 | 2025-07-23 | Chief Architect | Initial blueprint creation |

---

## 🤝 Contributing to the Blueprint

The blueprint **is** the product. Changes follow the **AIDL** (AI Development Lifecycle):

```
Proposal → Research → Architecture Review → Blueprint Approval → Implementation → Documentation
```

1. Create ADR in `00-governance/04-decision-log/`
2. Propose changes via PR with `docs:` prefix
3. Require Architecture Review (Chief Architect approval)
4. Update all affected volumes atomically
5. Update `last-reviewed` dates

---

**The Blueprint is the source of truth. Code implements the Blueprint. The Blueprint evolves through disciplined governance.**

---

*Vestara AI OS — Empowering people to transform ideas into products, businesses, and lifelong skills.*
