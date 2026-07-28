---
id: "RT-005"
title: "Vestara Doctor — Platform Diagnostics and Health Inspection"
owner: "@devops-engineer"
status: "ratified"
blueprint-ref: "16-operations/SUPPORT.md"
runtime-version: "1.0.0"
---

# Vestara Doctor
## Diagnose the Entire Platform with a Single Command

> **Instead of reading logs, ask: "Diagnose my workspace." Vestara Doctor inspects every layer — Kernel, services, plugins, providers, agents, filesystem, GPU, network, OS — and returns a health score with actionable findings.**

---

## Philosophy

Traditional debugging requires grepping logs, checking metrics, and correlating symptoms across services. Vestara Doctor reverses this: **it knows how the platform should work and checks everything against that expectation.**

---

## Interface

```bash
# Quick health check
vestara doctor

# Detailed diagnosis
vestara doctor --verbose

# Diagnose specific component
vestara doctor --service=memory
vestara doctor --plugin=* 
vestara doctor --provider=ollama

# Export diagnosis report
vestara doctor --output=report.json

# Continuous monitoring
vestara doctor --watch
```

---

## Output Format

```text
═══════════════════════════════════════════════
  VESTARA DOCTOR — System Diagnosis
═══════════════════════════════════════════════

  Overall Health: ● 92%  (Good)
  Timestamp:     2025-07-23T12:00:00Z
  Uptime:        14d 3h 22m

─────────────────────────────────────────────
  🔷 Kernel              ● 100%
  ✔ Boot sequence        Complete (1.2s)
  ✔ Configuration        Valid, 127 keys loaded
  ✔ Service count        12/12 running
  ✔ Memory usage         342MB / 8GB

─────────────────────────────────────────────
  🔷 Services            ● 95%
  ✔ Identity             Running (24ms latency)
  ✔ Workspace            Running (12ms latency)
  ✔ Memory               Running (8ms latency)
  ✔ Knowledge            Running (15ms latency)
  ✔ Provider             Running, 4 providers
  ✔ Agent Runtime        Running, 8 agents available
  ✔ Plugin Loader        5 plugins active
  ⚠ Conversation         Degraded (high latency: 2.3s p95)

─────────────────────────────────────────────
  🔷 AI Providers        ● 100%
  ✔ OpenCode             Available (10 models)
  ✔ Ollama               Available (2 models, 1.2GB VRAM)
  ✔ OpenAI               Available (8 models)
  ✔ Anthropic            Available (3 models)

─────────────────────────────────────────────
  🔷 Plugins             ● 100%
  ✔ git-integration      v1.2.0 — Active
  ✔ terminal-helper      v0.9.0 — Active
  ✔ web-search           v1.0.0 — Active
  ✔ code-formatter       v2.1.0 — Active
  ✔ theme-pack           v1.0.0 — Active

─────────────────────────────────────────────
  🔷 Filesystem          ● Warning
  ✔ .vestara/            Present, 2.3GB used
  ⚠ Disk usage           87% (230GB / 256GB SSD)

─────────────────────────────────────────────
  🔷 Hardware            ● 100%
  ✔ CPU                  Intel i7-13700H (16 cores)
  ✔ Memory               8GB (342MB used)
  ✔ GPU                  NVIDIA RTX 4060 (8GB VRAM)
  ✔ Disk                 NVMe SSD (256GB)

─────────────────────────────────────────────
  🔷 Network             ● 100%
  ✔ Connectivity         Online (34ms to opencode.ai)
  ✔ DNS                  Resolving
  ✔ mDNS                 Local peer discovery active

─────────────────────────────────────────────
  🔷 Security            ● 100%
  ✔ Secure Boot          Enabled
  ✔ Disk Encryption      LUKS2 active
  ✔ No known CVEs        Up to date
  ✔ Certificate          Valid (expires 2026-01-15)

═══════════════════════════════════════════════
  RECOMMENDATIONS:
  1. ⚠ Disk near capacity — free 30GB or upgrade SSD
  2. ⚠ Conversation service high latency — check provider routing
═══════════════════════════════════════════════
```

---

## Health Checks by Layer

```yaml
kernel:
  - check: "Boot sequence complete"
    command: "Check Kernel.status === 'running'"
  - check: "Configuration valid"
    command: "Validate all config schemas"
  - check: "All services running"
    command: "Check service registry counts"

services:
  - check: "Each service status"
    command: "service.health() for every registered service"
  - check: "Service dependencies"
    command: "Verify dependency graph is acyclic and all deps available"
  - check: "Service latency"
    command: "health() latency < 100ms"

providers:
  - check: "Provider health"
    command: "provider.healthCheck() for each registered provider"
  - check: "Model availability"
    command: "provider.listModels() returns expected models"
  - check: "Fallback chain valid"
    command: "No circular or broken fallback references"

plugins:
  - check: "Manifest valid"
    command: "Re-verify manifests against schema"
  - check: "Permissions valid"
    command: "Granted permissions still match requested"
  - check: "No crash loops"
    command: "Check error metrics for repetition"

filesystem:
  - check: ".vestara/ exists and writable"
    command: "stat() + access(W_OK)"
  - check: "Disk usage < 90%"
    command: "df .vestara/"
  - check: "Database integrity"
    command: "PRAGMA integrity_check"

hardware:
  - check: "CPU available"
    command: "os.cpus()"
  - check: "Memory sufficient"
    command: "freemem > 512MB"
  - check: "GPU available (if needed)"
    command: "nvidia-smi or vulkaninfo"
  - check: "Disk healthy"
    command: "SMART status via udisks2"

network:
  - check: "Connectivity"
    command: "HTTP HEAD to known endpoints"
  - check: "DNS resolution"
    command: "dns.resolve()"
  - check: "Local network"
    command: "mDNS broadcast"

security:
  - check: "Secure Boot enabled"
    command: "mokutil --sb-state"
  - check: "Disk encryption active"
    command: "cryptsetup status"
  - check: "No known vulnerabilities"
    command: "Check CVE database for installed versions"
  - check: "Certificate expiry"
    command: "openssl x509 -checkend"
```

---

## Health Score Calculation

```yaml
scoring:
  each_check:
    pass: 1.0
    warning: 0.5
    fail: 0.0
  
  layer_weight:
    kernel: 25%
    services: 25%
    providers: 15%
    plugins: 10%
    filesystem: 10%
    hardware: 5%
    network: 5%
    security: 5%
  
  thresholds:
    critical: "Any failure in Kernel, Security, or critical service"
    warning: "Score < 90% or any warning"
    healthy: "Score >= 90% and no failures"
```

---

## Vestara Doctor Commands

```bash
vestara doctor                    # Full diagnosis
vestara doctor --quick            # Quick health check (10s)
vestara doctor --watch            # Continuous monitoring
vestara doctor --service=memory   # Single service
vestara doctor --output=json      # Machine-readable
vestara doctor --fix              # Auto-fix common issues
vestara doctor --history          # Health history
```

---

*Vestara Doctor makes every deployment diagnosable by a single command. Instead of "It doesn't work," users can run `vestara doctor` and get actionable results.*
