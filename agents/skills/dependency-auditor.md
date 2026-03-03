---
name: dependency-auditor
description: Audits project dependencies for security vulnerabilities, outdated packages, and license compliance.
---
## Dependency Auditor Skill

Comprehensive dependency analysis for security, updates, and compliance.

### When to Use
- At the start of any new issue (proactive)
- When adding new dependencies
- Before major releases
- As part of security review
- Periodically (weekly/monthly maintenance)

### Prerequisites
**CRITICAL**: Read `PROJECT_CONTEXT.md` for:
- Allowed licenses
- Dependency management approach
- Update policies

### Audit Workflow

```
┌─────────────────────────────────────────────────────┐
│  1. Security Audit                                   │
│     - Check for known vulnerabilities                │
│     - Flag CVEs by severity                          │
└───────────────────────────┬─────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────┐
│  2. Outdated Check                                   │
│     - Identify outdated packages                     │
│     - Check for major version updates               │
└───────────────────────────┬─────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────┐
│  3. License Compliance                               │
│     - Verify license compatibility                   │
│     - Flag restrictive licenses                      │
└───────────────────────────┬─────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────┐
│  4. Generate Report                                  │
│     - Summary with action items                      │
│     - Prioritized fix list                           │
└─────────────────────────────────────────────────────┘
```

### Step 1: Security Audit

#### JavaScript/TypeScript (npm)
```bash
# Run npm audit
npm audit --json > audit-results.json

# Parse results
npm audit --audit-level=moderate
```

#### Python (pip)
```bash
# Using pip-audit
pip-audit --format=json > audit-results.json

# Using safety
safety check --json > audit-results.json
```

#### Go
```bash
# Using nancy
go list -m -json all | nancy sleuth

# Using govulncheck
govulncheck ./...
```

### Step 2: Outdated Check

#### JavaScript/TypeScript
```bash
# Check outdated
npm outdated --json > outdated.json

# Interactive update
npx npm-check-updates
```

#### Python
```bash
# Check outdated
pip list --outdated --format=json > outdated.json

# Using pip-tools
pip-compile --upgrade
```

#### Go
```bash
# Check for updates
go list -m -u all
```

### Step 3: License Compliance

#### Allowed Licenses (typical)
- MIT
- Apache-2.0
- BSD-2-Clause
- BSD-3-Clause
- ISC

#### Restricted Licenses (require review)
- GPL-2.0, GPL-3.0 (copyleft)
- LGPL-2.0, LGPL-3.0 (copyleft)
- AGPL-3.0 (strong copyleft)

#### Check Licenses
```bash
# JavaScript
npx license-checker --json > licenses.json

# Python
pip-licenses --format=json > licenses.json

# Go
go-licenses check ./...
```

### Step 4: Generate Report

Save to `agents/logs/dependency-audit-<timestamp>.md`:

```markdown
# Dependency Audit Report

## Metadata
- **Date:** <ISO timestamp>
- **Project:** <project name>
- **Package Manager:** npm/pip/go mod

---

## Summary

| Category | Critical | High | Medium | Low |
|----------|----------|------|--------|-----|
| Security | 0 | 2 | 5 | 12 |
| Outdated | - | 3 | 8 | 15 |

### Overall Status: NEEDS_ATTENTION

---

## Security Vulnerabilities

### Critical (0)
No critical vulnerabilities found.

### High (2)

#### 1. lodash - Prototype Pollution
- **Package:** lodash@4.17.15
- **CVE:** CVE-2021-23337
- **Severity:** High (7.5)
- **Fixed in:** 4.17.21
- **Path:** lodash > package-a > package-b

**Action Required:**
```bash
npm update lodash
```

#### 2. axios - SSRF Vulnerability
- **Package:** axios@0.21.0
- **CVE:** CVE-2021-3749
- **Severity:** High (7.5)
- **Fixed in:** 0.21.2

**Action Required:**
```bash
npm update axios
```

### Medium (5)
<list of medium severity issues>

---

## Outdated Packages

### Major Updates Available (Breaking Changes)

| Package | Current | Latest | Type |
|---------|---------|--------|------|
| react | 17.0.2 | 18.2.0 | major |
| typescript | 4.9.5 | 5.3.3 | major |

**Note:** Major updates require testing and may have breaking changes.

### Minor/Patch Updates (Safe to Update)

| Package | Current | Latest | Type |
|---------|---------|--------|------|
| eslint | 8.45.0 | 8.56.0 | minor |
| jest | 29.5.0 | 29.7.0 | minor |

**Quick Update:**
```bash
npm update
```

---

## License Compliance

### Approved Licenses
- MIT: 145 packages
- Apache-2.0: 23 packages
- BSD-3-Clause: 12 packages

### Requires Review

| Package | License | Risk |
|---------|---------|------|
| package-x | GPL-3.0 | High |
| package-y | LGPL-2.1 | Medium |

**Action:** Review license compatibility before release.

### Unknown Licenses

| Package | License |
|---------|---------|
| package-z | UNKNOWN |

**Action:** Investigate and document license.

---

## Unused Dependencies

These packages appear unused:
- `moment` (consider removing or replacing with date-fns)
- `underscore` (duplicate of lodash)

```bash
# Check usage
npx depcheck
```

---

## Recommendations

### Immediate (High Priority)
1. Update lodash to 4.17.21
2. Update axios to 0.21.2+

### Short-term (Medium Priority)
1. Review GPL-3.0 licensed package
2. Plan React 18 migration
3. Remove unused dependencies

### Long-term (Low Priority)
1. TypeScript 5 migration
2. License documentation update

---

## Update Commands

### Safe Updates (Minor/Patch)
```bash
npm update
```

### Security Fixes Only
```bash
npm audit fix
```

### Force Security Fixes (May Break)
```bash
npm audit fix --force
```

---

*Generated by dependency-auditor skill*
```

### Automation Integration

Add to CI pipeline:
```yaml
# GitHub Actions example
- name: Security Audit
  run: |
    npm audit --audit-level=high
    if [ $? -ne 0 ]; then
      echo "Security vulnerabilities found"
      exit 1
    fi
```

### Output Format

```
## Dependency Audit Complete

**Date:** 2024-03-15
**Status:** NEEDS_ATTENTION

### Security
- Critical: 0
- High: 2 (requires fix)
- Medium: 5 (review recommended)

### Outdated
- Major updates: 3 (requires planning)
- Minor updates: 12 (safe to update)

### License Issues
- GPL packages: 1 (review needed)
- Unknown licenses: 1 (investigate)

### Immediate Actions
1. `npm update lodash axios` - Fix high severity CVEs
2. Review package-x GPL-3.0 license

**Report:** agents/logs/dependency-audit-20240315.md
```

### Integration
- Triggered by: Maintenance schedule, security reviews
- Reports to: `lessons-writer` (for dependency patterns)
- Blocks: Release if critical vulnerabilities
