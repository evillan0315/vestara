---
title: "Platform Architecture — Complete Platform Specification"
volume: "04-platform"
book: "Book 2: Platform Architecture"
version: "1.0.0"
status: "approved"
owner: "@chief-architect"
last-reviewed: "2025-07-23"
next-review: "2026-01-23"
tags: ["platform", "architecture", "layers", "modules"]
---

# Platform Architecture
## The Complete Specification for the Vestara Platform

> **This document defines the layered architecture, module boundaries, dependency rules, and key data flows of the Vestara platform. Every service, app, and package in the ecosystem implements these specifications.**

---

## ═══════════════════════════════════════════════════════════════════
### 🏗️ PLATFORM LAYERS
### ═══════════════════════════════════════════════════════════════════

```
┌─────────────────────────────────────────────────────┐
│                    APPLICATIONS                       │
│  Dashboard (React)  │  CLI (Node)  │  Mobile (RN)    │
├─────────────────────────────────────────────────────┤
│                  WORKSPACE SERVICES                    │
│  Projects │ Knowledge │ Memory │ Notifications      │
├─────────────────────────────────────────────────────┤
│                      AI CORE                          │
│  Agents │ Providers │ Prompts │ Evaluation │ Safety  │
├─────────────────────────────────────────────────────┤
│                  PLATFORM SERVICES                     │
│  Auth │ Settings │ Events │ Filesystem │ Sync       │
├─────────────────────────────────────────────────────┤
│                   INFRASTRUCTURE                       │
│  SQLite │ Immutable OS │ Network │ Security        │
└─────────────────────────────────────────────────────┘
```

---

## ═══════════════════════════════════════════════════════════════════
### 📦 MODULE SPECIFICATIONS
### ═══════════════════════════════════════════════════════════════════

### Identity Module (`04-platform/identity/`)
**Purpose**: User identity, authentication, authorization, roles

| Aspect | Specification |
|--------|---------------|
| Auth methods | OS login (username/password), JWT, SSO (Gen 3) |
| Roles | `admin`, `editor`, `user` |
| Storage | SQLite (users, sessions tables) |
| API | `POST /auth/os-login`, `POST /auth/os-auto-login`, `GET /auth/me`, `DELETE /auth/logout` |
| Boundaries | Zod schemas: `LoginSchema`, `RegisterSchema` |
| Future | MFA, passkeys, OAuth2, SAML (Gen 2+) |

### Organizations Module (`04-platform/organizations/`)
**Purpose**: Multi-tenant organization management (Gen 3+)

| Aspect | Specification |
|--------|---------------|
| Scope | Team workspace, org-level knowledge |
| Storage | SQLite (organizations, memberships tables) |
| API | `POST /orgs`, `GET /orgs/:id`, `PATCH /orgs/:id`, `DELETE /orgs/:id` |
| Permissions | Role-based (owner, admin, member, viewer) |
| Future | SAML/SCIM, billing, audit logs |

### Projects Module (`04-platform/projects/`)
**Purpose**: Project and task management

| Aspect | Specification |
|--------|---------------|
| Storage | SQLite (projects, tasks tables) |
| Features | CRUD, Kanban, sub-tasks, tags, time tracking, archive |
| API | `GET /projects`, `POST /projects`, `GET /projects/:id`, `PATCH /projects/:id`, `DELETE /projects/:id` |
| Sync | `.vestara/` folder, cloud sync (Gen 3) |
| Events | `project:created`, `project:archived`, `task:updated` |

### Workspace Module (`04-platform/workspace/`)
**Purpose**: Workspace orchestration and state management

| Aspect | Specification |
|--------|---------------|
| Scopes | User settings, project settings, workspace layout |
| Storage | SQLite (settings table) + in-memory state |
| Features | Layout persistence, panel management, multi-window |
| Events | `workspace:layout-changed`, `workspace:theme-changed` |

### Plugins Module (`04-platform/plugins/`)
**Purpose**: Plugin system and SDK

| Aspect | Specification |
|--------|---------------|
| Language | TypeScript |
| Runtime | Isolated VM with permissions |
| API | PluginSDK class with typed hooks |
| Permissions | Filesystem, network, AI, UI, storage (granular) |
| Marketplace | Discover, install, version, update |
| Events | `plugin:installed`, `plugin:activated`, `plugin:error` |

### Providers Module (`04-platform/providers/`)
**Purpose**: External service provider management

| Aspect | Specification |
|--------|---------------|
| Types | AI providers, storage providers, identity providers |
| Config | `@vestara/config` with env + defaults |
| Health | Periodic health checks, status tracking |
| Fallback | Configurable provider chains |

### Filesystem Module (`04-platform/filesystem/`)
**Purpose**: File abstraction and project storage

| Aspect | Specification |
|--------|---------------|
| Structure | `.vestara/` folder per project |
| Contents | SQLite DB, knowledge files, memory, settings |
| Portability | Self-contained, movable, Git-friendly |
| Sync | `rsync`-based, cloud sync protocol (Gen 3) |

### Marketplace Module (`04-platform/marketplace/`)
**Purpose**: Plugin and extension marketplace (Gen 3+)

| Aspect | Specification |
|--------|---------------|
| Scope | Plugins, themes, templates, AI providers |
| Revenue | 30% commission, subscription passthrough |
| Review | Automated scanning + human review |
| Versioning | Semantic versioning, compatibility matrix |

### Automation Module (`04-platform/automation/`)
**Purpose**: Workflow automation and scheduled tasks

| Aspect | Specification |
|--------|---------------|
| Triggers | Time, event, webhook, AI-decision |
| Actions | Service methods, API calls, shell commands |
| Storage | SQLite (automations, logs tables) |
| Security | Sandboxed execution, permission scopes |

### Notifications Module (`04-platform/notifications/`)
**Purpose**: User notification system

| Aspect | Specification |
|--------|---------------|
| Types | In-app, push (Gen 3), email (Gen 3) |
| Priorities | `low`, `normal`, `high`, `critical` |
| Channels | Activity log, WebSocket, notification center |
| Storage | SQLite (notifications, activity_log tables) |
| Events | `notification:new`, `notification:read` |

### Analytics Module (`04-platform/analytics/`)
**Purpose**: Platform analytics and instrumentation (Opt-in)

| Aspect | Specification |
|--------|---------------|
| Opt-in | Required explicit user consent |
| Data | Usage patterns, performance, errors |
| Storage | Local SQLite (exportable) |
| Privacy | No PII, no IP tracking, aggregated only |

### Licensing Module (`04-platform/licensing/`)
**Purpose**: License management and entitlements (Gen 2+)

| Aspect | Specification |
|--------|---------------|
| Tiers | Community, Pro, Team, Enterprise |
| Enforcement | Local validation, periodic check |
| Features | Feature flags per tier |
| Offline | Graceful degradation without validation |

### Synchronization Module (`04-platform/synchronization/`)
**Purpose**: Cross-device and cross-org data sync

| Aspect | Specification |
|--------|---------------|
| Protocol | CRDT-based, last-writer-wins, conflict resolution |
| Transport | Local file → mDNS → WebSocket → HTTPS |
| Storage | SQLite + filesystem watcher |
| Events | `sync:started`, `sync:progress`, `sync:completed`, `sync:conflict` |

---

## ═══════════════════════════════════════════════════════════════════
### 🔗 DEPENDENCY GRAPH
### ═══════════════════════════════════════════════════════════════════

```
Platform Dependencies (Inward-pointing)

Apps ──→ Workspace Services ──→ AI Core ──→ Platform Services ──→ Infrastructure
  │            │                   │                │                    │
  └────────────┴───────────────────┴────────────────┴────────────────────┘
                           (All point inward)
```

**Constraint**: A module at layer N may only depend on modules at layer N+1 (inward). Circular dependencies are FORBIDDEN and will be caught by CI (`madge --circular`).

---

## ═══════════════════════════════════════════════════════════════════
### 🔄 PLATFORM LIFECYCLE
### ═══════════════════════════════════════════════════════════════════

The platform evolves through semantic versioning (MAJOR.MINOR.PATCH):

| Bump | When | Migration Required |
|------|------|-------------------|
| MAJOR | Breaking API change | Full migration guide + deprecation window |
| MINOR | New feature, backward compatible | Additive only |
| PATCH | Bug fix, no API change | None |

**All platform modules follow the same version number.** Major releases align with Vestara generations.

---

**END OF PLATFORM OVERVIEW**

*The platform is the foundation upon which all Vestara products are built. Its stability and clarity determine the quality of everything above it.*
