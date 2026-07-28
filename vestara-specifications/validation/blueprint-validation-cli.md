---
id: "SPEC-VAL-000"
title: "Blueprint Validation CLI — Making the Blueprint Executable"
owner: "@chief-architect"
status: "draft"
blueprint-ref: "14-engineering/RELEASE_PROCESS.md"
version: "1.0.0"
---

# Blueprint Validation CLI
## Making the Blueprint Testable, Not Just Readable

> **The Blueprint should be verifiable by machine, not just read by humans. This specification defines the `vestara blueprint validate` command that checks Blueprint integrity, consistency, and completeness — ensuring that documentation, architecture, and implementation stay synchronized.**

---

## Command Interface

```bash
# Validate entire Blueprint
vestara blueprint validate

# Validate specific volume
vestara blueprint validate --volume=05-ai-core

# Validate against a specific repository
vestara blueprint validate --repo=vestara-ai-core

# Check cross-references between Blueprint and specs
vestara blueprint validate --cross-ref

# Output as JSON for CI consumption
vestara blueprint validate --format=json

# Fix auto-fixable issues
vestara blueprint validate --fix

# Watch mode (re-validate on file changes)
vestara blueprint validate --watch
```

## Exit Codes

| Code | Meaning |
|------|---------|
| 0 | All checks pass |
| 1 | Warnings found (non-blocking) |
| 2 | Errors found (blocking) |
| 3 | Fatal — cannot parse Blueprint |

---

## Validation Checks

### 1. Structural Integrity

| Check | Description | Severity |
|-------|-------------|----------|
| Required files exist | All files listed in `README.md` actually exist | Error |
| Frontmatter validity | Every `.md` file has valid YAML frontmatter | Error |
| No orphaned files | Files not referenced by any index are flagged | Warning |
| Directory completeness | All required subdirectories exist per volume spec | Error |
| Symlinks valid | No broken symlinks | Error |

### 2. Cross-Reference Integrity

| Check | Description | Severity |
|-------|-------------|----------|
| Internal links valid | All `vestara-blueprint/` links resolve to existing files | Error |
| Blueprint refs valid | All `blueprint-ref:` values in specs point to existing blueprint files | Error |
| ADR references valid | All ADR references (`ADR-XXX`) exist in decision log | Error |
| Event references valid | All emitted/received events are in event catalog | Warning |
| API references valid | All API endpoints referenced are in API contract library | Warning |
| No dead links | No 404s in external references | Warning |

### 3. Capability Coverage

| Check | Description | Severity |
|-------|-------------|----------|
| Capability index complete | Every CAP-XXX has a specification | Warning |
| Capability owners assigned | Every CAP-XXX has an owner | Warning |
| Capability status current | No CAP-XXX stuck in draft >90 days | Warning |
| Capability → Blueprint mapping | Every CAP-XXX maps to a Blueprint volume | Error |
| Gen compatibility | Capabilities don't reference Gen 3 APIs in Gen 1 | Error |

### 4. API Contract Consistency

| Check | Description | Severity |
|-------|-------------|----------|
| Endpoint uniqueness | No duplicate API endpoints | Error |
| Pagination consistency | All list endpoints have pagination defined | Warning |
| Error coverage | All error codes have documentation | Warning |
| Auth consistency | Auth requirements match security model | Error |
| Rate limit coverage | All endpoints have rate limits defined | Warning |
| DTO completeness | All request/response types defined | Error |

### 5. Event Catalog Coverage

| Check | Description | Severity |
|-------|-------------|----------|
| Event uniqueness | No duplicate event types | Error |
| Publisher defined | Every event has a publisher | Warning |
| Subscriber defined | Every event has at least one subscriber | Warning |
| Payload defined | Every event has a payload schema | Error |
| Events emitted | Every emitted event is in catalog | Error |
| Events consumed | Every consumed event is in catalog | Warning |

### 6. Data Model Consistency

| Check | Description | Severity |
|-------|-------------|----------|
| Entity uniqueness | No duplicate entity definitions | Error |
| Field type consistency | Same field name has same type across entities | Warning |
| FK relationships valid | All foreign keys reference existing entities | Error |
| Index completeness | All FK columns have indexes | Warning |
| Migration safety | No columns defined without defaults | Warning |
| Naming conventions | All tables/columns follow snake_case | Error |

### 7. Architecture Compliance

| Check | Description | Severity |
|-------|-------------|----------|
| Layer compliance | No circular dependencies between layers | Error |
| Module boundaries | No cross-module direct dependencies | Warning |
| ADR coverage | Architectural decisions have ADRs | Warning |
| ADR staleness | ADRs older than 6 months flagged for review | Warning |
| Security review | Security-impacting changes have security ADR | Error |

### 8. AI Contract Coverage

| Check | Description | Severity |
|-------|-------------|----------|
| Contract completeness | Every AI-CON-XXX has a specification | Error |
| Interface defined | Every contract has TypeScript interface | Warning |
| Performance targets | Every contract has performance targets | Warning |
| Dependencies documented | Every contract lists dependencies | Warning |

### 9. Design System Consistency

| Check | Description | Severity |
|-------|-------------|----------|
| Token uniqueness | No duplicate design tokens | Error |
| Token usage | All tokens referenced in component specs exist | Error |
| Accessibility compliance | WCAG contrast ratios met | Warning |
| Responsive coverage | All components have mobile spec | Warning |

### 10. Generation Compatibility

| Check | Description | Severity |
|-------|-------------|----------|
| Gen compatibility | Gen 1 specs don't reference Gen 2+ features | Error |
| Roadmap alignment | Capabilities match roadmap generation | Warning |
| Deprecation tracking | Superseded specs marked deprecated | Warning |
| Breaking changes | Breaking changes have migration guide | Error |

---

## Output Format

### Console Output (Default)
```text
vestara blueprint validate --volume=04-platform
═════════════════════════════════════════════════════
Validating: 04-platform/ (Platform Architecture)
═════════════════════════════════════════════════════

✅ Structural Integrity       — 3 checks passed
⚠️  Cross-Reference Integrity — 2 warnings
   • 04-platform/README.md references 12-data which is empty
   • 04-platform/PLATFORM_OVERVIEW.md references CAP-060 which has no spec
✅ Capability Coverage       — 4 checks passed
✅ Event Catalog Coverage    — 5 checks passed
⚠️  Data Model Consistency    — 1 warning
   • projects.metadata lacks index
✅ Architecture Compliance   — 6 checks passed

═════════════════════════════════════════════════════
Result: PASSED with 3 warnings
Time: 0.847s
═════════════════════════════════════════════════════
```

### JSON Output (CI)
```json
{
  "version": "1.0.0",
  "timestamp": "2025-07-23T12:00:00Z",
  "status": "passed",
  "checks": {
    "passed": 18,
    "warnings": 3,
    "errors": 0
  },
  "details": [
    {
      "check": "cross-reference-integrity",
      "status": "warning",
      "message": "04-platform/README.md references 12-data which is empty",
      "file": "04-platform/README.md",
      "line": 42
    }
  ],
  "duration_ms": 847
}
```

---

## Configuration

The validator reads configuration from `vestara-blueprint/.blueprint-validator.yml`:

```yaml
# .blueprint-validator.yml
validator:
  version: "1"

  severity:
    missing-frontmatter: error
    broken-link: error
    missing-adr: warning
    stale-adr-days: 180
    draft-spec-stale-days: 90
    missing-owner: warning
    missing-performance-targets: warning

  paths:
    blueprint: "vestara-blueprint/"
    specifications: "vestara-specifications/"
    implementation:
      - "services/"
      - "packages/"
      - "apps/"

  rules:
    - "no-broken-links"
    - "no-missing-frontmatter"
    - "all-capabilities-have-owners"
    - "all-adrs-have-blueprint-ref"
    - "all-events-have-publisher"
    - "all-entities-have-fields"
    - "all-apis-have-rate-limits"
    - "layer-compliance-no-circular"

  ci-mode: false
  fix-mode: false
```

---

## Implementation Plan

| Phase | Scope | Timeline |
|-------|-------|----------|
| **Phase 1** | Structural integrity + cross-reference validation | Gen 1 Alpha |
| **Phase 2** | Capability coverage + API contract consistency | Gen 1 Beta |
| **Phase 3** | Event catalog + data model consistency | Gen 1 GA |
| **Phase 4** | Architecture compliance + AI contracts | Gen 1.5 |
| **Phase 5** | Design system + generation compatibility | Gen 2 |

---

## Integration with CI

The validation command runs in CI on every PR that modifies `vestara-blueprint/` or `vestara-specifications/`:

```yaml
# .github/workflows/blueprint-ci.yml (extended)
- name: Validate Blueprint
  run: vestara blueprint validate --format=json
  continue-on-error: true  # Warnings don't block

- name: Check Architecture Compliance
  run: vestara blueprint validate --check=architecture
  # Errors DO block
```

---

*The Blueprint Validation CLI makes the Blueprint testable, accountable, and self-healing. When documentation and implementation drift, the validator catches it.*
