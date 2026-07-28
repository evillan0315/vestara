# Vestara Labs
## Experimental Ground — Protecting the Frozen Architecture

> **Nothing experimental belongs in the production repositories. If an experiment succeeds, it goes through: Labs → ADR → Specification → Foundation → Implementation.**

---

## Purpose

`vestara-labs` exists to protect the frozen architecture. Every production repository (`vestara-blueprint`, `vestara-specifications`, `vestara-foundation`, `vestara-runtime`, `vestara-ai-core`) follows strict governance. Labs has no governance — it is the place to explore, prototype, and fail without consequences.

## Policy

| Action | Rule |
|--------|------|
| **Experiment** | Anything goes in Labs |
| **Production promotion** | Requires ADR + Specification + Foundation contract |
| **Architecture freeze** | Labs cannot modify frozen architecture |
| **Code quality** | No standards required (experimental) |
| **Data** | No user data in experiments |

## Directory

| Directory | Purpose |
|-----------|---------|
| `experiments/` | Quick prototypes and exploratory coding |
| `prototypes/` | More structured proofs of concept |
| `research/` | Research reports, model evaluations, benchmarks |
| `benchmarks/` | Performance and capability benchmarks |
| `future-models/` | Next-generation AI model integration |
| `reasoning/` | Novel reasoning strategies and architectures |
| `robotics/` | Physical hardware integration (future) |
| `vision/` | Image/video understanding (future) |
| `voice/` | Speech recognition and synthesis (future) |

## Promotion Path

```
Labs Experiment
    ↓
Successful?
    ↓ YES
ADR → Specification → Foundation Contract → Implementation
    ↓
Production Repository
```
