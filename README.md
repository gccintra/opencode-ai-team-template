# OpenCode AI Team Template

A production-ready template for orchestrating a team of specialized AI agents that collaborates to deliver features end-to-end — from requirements to Pull Request — with automated planning, implementation, testing, security checks, and code review.

---

## Table of Contents

- [Overview](#overview)
- [Full Workflow](#full-workflow)
- [Quick Start](#quick-start)
- [Agents](#agents)
- [Quality Gates](#quality-gates)
- [GitHub Issues & Labels Standard](#github-issues--labels-standard)
- [Skills Reference](#skills-reference)
- [File Structure](#file-structure)
- [Configuration](#configuration)
- [Hotfix Flow](#hotfix-flow)
- [Customization](#customization)
- [Troubleshooting](#troubleshooting)

---

## Overview

This template gives you a complete AI engineering team operating inside [OpenCode](https://opencode.ai). Each agent has a specific role, a defined model, and strict quality gates that prevent the workflow from advancing until requirements are met.

**The team:**

| Agent | Role | Model | Mode |
|-------|------|-------|------|
| `issue-crafter` | Discusses requirements and creates GitHub issues | claude-sonnet-4-6 | primary |
| `orchestrator` | Reads issues or prompts, discusses approach, creates specs, routes to planners | gemini-3-pro | primary |
| `planner-frontend` | Plans UI, components, state, and accessibility | gemini-3-pro | subagent |
| `planner-backend` | Plans APIs, database, services, and migrations | gemini-3-pro | subagent |
| `executor` | Implements code with mandatory tests and security checks | claude-sonnet-4-6 | subagent |
| `tester` | Runs tests, generates coverage reports, logs results | qwen2.5-coder:14b | subagent |
| `reviewer` | Code review, final security scan, marks READY_TO_COMMIT | qwen2.5-coder:14b | subagent |
| `committer` | Creates commit, pushes branch, opens PR (manual trigger) | gemini-3-pro | subagent |
| `hotfix` | Emergency bypass flow for critical production issues | claude-sonnet-4-6 | primary |

---

## Full Workflow

### Entry Points

There are two ways to start the workflow:

**Option A — Via GitHub issue (recommended for team visibility):**
```
@issue-crafter          # create a structured GitHub issue
@orchestrator #<num>    # start the workflow from the issue
```

**Option B — Via plain text prompt (fast, no GitHub issue needed):**
```
@orchestrator add JWT authentication to the API
```
The orchestrator detects the input type automatically.

---

### Standard Flow

```
USER
 │
 ├── [Option A] @issue-crafter
 │     Discusses requirements conversationally
 │     Proposes technical approaches
 │     Drafts issue in standard format
 │     Creates GitHub issue with labels + @me assignee
 │     └── Outputs: GitHub issue URL + issue number
 │
 ├── @orchestrator #<num>  OR  @orchestrator <plain text prompt>
 │     Reads PROJECT_CONTEXT.md
 │
 │     [If issue number] Uses issue-reader skill to parse GitHub issue
 │     [If plain prompt] Asks 4 clarifying questions in one message, waits for response
 │     └── Creates agents/specs/<id>-intake.md
 │
 │     [Technical Discussion — skipped for hotfix/docs/chore/test]
 │     Asks what the user has in mind for the approach
 │     Evaluates user's idea against architecture — validates, challenges, or refines
 │     If user has no opinion: decides autonomously with justification
 │     └── Creates agents/specs/<id>-approach.md
 │
 │     Creates agents/specs/<id>-spec.md (references approved approach)
 │     Initializes task tracking via todo-manager
 │     [Gate G1]
 │     └── Routes to planners based on scope
 │
 ├── @planner-frontend agents/specs/<id>-spec.md  (if frontend/full-stack)
 │     Reads PROJECT_CONTEXT.md + spec + approach document
 │     Plans component hierarchy and state
 │     Applies frontend-design + brand-guidelines skills
 │     Creates agents/specs/<id>-frontend-plan.md
 │     Creates agents/tasks/frontend-todo.md
 │     [Gate G2]
 │     └── Hands off to @executor
 │
 ├── @planner-backend agents/specs/<id>-spec.md   (if backend/full-stack)
 │     Reads PROJECT_CONTEXT.md + spec + approach document
 │     Plans API endpoints, database schema, service layers
 │     Uses db-migrator skill for schema changes
 │     Creates agents/specs/<id>-backend-plan.md
 │     Creates agents/tasks/backend-todo.md
 │     [Gate G2]
 │     └── Hands off to @executor
 │
 ├── @executor agents/specs/<id>-{frontend,backend}-plan.md
 │     Reads plan + spec + PROJECT_CONTEXT.md
 │     Implements code (minimal impact, no over-engineering)
 │     Uses test-generator skill after each implementation
 │     Runs security-checker skill on changed files
 │     [Gate G3]
 │     └── Hands off to @tester
 │
 ├── @tester agents/specs/<id>-spec.md
 │     Reads PROJECT_CONTEXT.md for test commands and thresholds
 │     Runs full test suite (unit + integration + E2E)
 │     Uses coverage-reporter skill (threshold: 80%)
 │     Uses test-logger skill → agents/logs/
 │     [Gate G4]
 │     ├── PASS → @reviewer
 │     └── FAIL → back to @executor
 │
 ├── @reviewer agents/specs/<id>-spec.md
 │     Reads all changed files vs main
 │     Uses quick-review skill (code quality, architecture, performance)
 │     Runs security-checker skill (final pass)
 │     Verifies test logs exist and pass
 │     Uses lessons-writer skill for patterns discovered
 │     [Gate G5]
 │     ├── APPROVED → marks spec as READY_TO_COMMIT, notifies user
 │     └── CHANGES REQUESTED → back to @executor
 │
 └── @committer agents/specs/<id>-spec.md   (manual, user-triggered)
       Verifies spec status = READY_TO_COMMIT
       Uses commit-changes skill (conventional commit format)
       Uses push-changes skill (creates feature branch, pushes)
       Uses create-pr skill (links issue, attaches test evidence)
       └── Outputs: PR URL
```

### Visual Diagram

```
  [Option A]                        [Option B]
@issue-crafter                  plain text prompt
      │                                 │
      │ GitHub Issue #<num>             │
      └──────────────┬──────────────────┘
                     │
                     ▼
         ┌───────────────────────┐
         │      ORCHESTRATOR     │
         │  issue-reader (A)     │
         │  inline intake (B)    │
         │  tech discussion      │
         │  todo-manager         │
         │  [Gate G1]            │
         └──────────┬────────────┘
               ┌────┴─────┐
               ▼           ▼
   ┌───────────────┐  ┌───────────────┐
   │  PLANNER-FE   │  │  PLANNER-BE   │
   │ frontend-des. │  │  db-migrator  │
   │ brand-guide.  │  │  golang-pro   │
   │  [Gate G2]    │  │  [Gate G2]    │
   └──────┬────────┘  └────────┬──────┘
          └──────────┬─────────┘
                     ▼
         ┌───────────────────────┐
         │       EXECUTOR        │
         │  test-generator       │
         │  security-checker     │
         │  todo-manager         │
         │  [Gate G3]            │
         └──────────┬────────────┘
                    ▼
         ┌───────────────────────┐
         │        TESTER         │
         │  test-runner          │
         │  coverage-reporter    │
         │  test-logger          │
         │  [Gate G4]            │
         └──────────┬────────────┘
               Pass │ Fail
                    │   └──→ EXECUTOR (fix loop)
                    ▼
         ┌───────────────────────┐
         │       REVIEWER        │
         │  quick-review         │
         │  security-checker     │
         │  lessons-writer       │
         │  [Gate G5]            │
         └──────────┬────────────┘
            Approved │ Changes
                     │   └──→ EXECUTOR (revision loop)
                     ▼
         ┌───────────────────────┐
         │  spec: READY_TO_COMMIT│
         │  → notify user        │
         └──────────┬────────────┘
                    │ USER: @committer
                    ▼
         ┌───────────────────────┐
         │       COMMITTER       │
         │  commit-changes       │
         │  push-changes         │
         │  create-pr            │
         │  pr-description       │
         └───────────────────────┘
                    │
                    ▼
         GitHub Pull Request (with test evidence)
```

---

## Quick Start

### Option A: Full issue-tracked flow

**1. Create and craft an issue**

```
@issue-crafter
```

The agent starts a conversation, discusses your requirements, proposes technical approaches, and creates a structured GitHub issue. At the end it outputs the issue URL and the command to run next.

**2. Start the automated workflow**

```
@orchestrator #<issue-number>
```

The orchestrator parses the issue, opens a discussion about the technical approach, creates a specification, initializes task tracking, and delegates to the correct planners. The flow continues automatically through planners → executor → tester → reviewer.

**3. Commit and open the PR**

When the reviewer marks the spec as `READY_TO_COMMIT`, trigger the final step:

```
@committer agents/specs/<id>-spec.md
```

---

### Option B: Prompt-only flow (no GitHub issue)

```
@orchestrator add password reset flow to the auth module
```

The orchestrator asks 4 clarifying questions in one message (scope, acceptance criteria, constraints, priority), creates an intake document locally, and continues through the full workflow. No GitHub issue is created.

---

## Agents

### issue-crafter

**Purpose:** Entry point for the issue-tracked flow. Conducts a focused conversation to gather requirements, proposes 1–2 technical approaches grounded in `PROJECT_CONTEXT.md`, and creates a GitHub issue with the exact structure that `issue-reader` expects.

**How to invoke:**
```
@issue-crafter
```

**What it does:**
1. Reads `PROJECT_CONTEXT.md` and detects the authenticated GitHub user (`gh api user --jq .login`)
2. Opens conversational discovery (problem, impact, scope, acceptance criteria, constraints, dependencies, references)
3. Classifies type/scope/priority and proposes technical approaches
4. Drafts the full issue — waits for explicit approval before creating
5. Ensures all required labels exist, then creates the issue with `--assignee @me`

**Issue body format it creates:**
```markdown
## Problem Statement
## Acceptance Criteria
## Technical Requirements
## Design References
## Dependencies
## Notes
```

**Output:** Issue URL + number + suggested next command (`@orchestrator #<num>`)

---

### orchestrator

**Purpose:** Staff engineer coordinator. Does not write code. Accepts either a GitHub issue number or a plain text prompt, discusses the technical approach with the user, and routes to the correct planners.

**How to invoke:**
```
@orchestrator #<issue-number>           # issue-based
@orchestrator <plain text description>  # prompt-based
```

**What it does:**

1. **Reads `PROJECT_CONTEXT.md`** (mandatory)

2. **Intake (two paths):**
   - *Issue path:* Uses `issue-reader` to parse GitHub issue → `<id>-intake.md`
   - *Prompt path:* Asks 4 clarifying questions in one message, waits for response, then creates intake document locally. Identifier becomes `task-<slug>` (e.g., `task-add-jwt-auth`)

3. **Technical Solutions Discussion** *(skipped for hotfix/docs/chore/test)*
   - Asks what the user has in mind before writing any spec
   - If user shares an idea: evaluates it against the architecture — validates if solid, raises concerns if not, proposes refinements if partially good
   - If user has no opinion: decides autonomously with a brief justification
   - Caps at 3 exchanges; if no consensus, makes the final call
   - Creates `<id>-approach.md` documenting the decision (origin: user-driven / orchestrator-decided / collaborative)

4. **Creates `<id>-spec.md`** with architecture fit, affected files, testing strategy, and a reference to the approved approach

5. **Initializes task tracking** with `todo-manager`, passes Gate G1, delegates to planners

**Identifier convention:**

| Input type | `<id>` format | Example |
|-----------|--------------|---------|
| GitHub issue `#42` | `issue-42` | `agents/specs/issue-42-spec.md` |
| Plain prompt | `task-<slug>` | `agents/specs/task-add-jwt-auth-spec.md` |

**Special cases:**
- `hotfix` or `urgent` label → routes to `@hotfix`, skips discussion
- `docs` only → routes directly to `@executor --docs-only`, skips discussion

---

### planner-frontend

**Purpose:** Plans all frontend work: component hierarchy, state management, routing, accessibility, and integration points.

**How to invoke:** (called automatically by orchestrator)
```
@planner-frontend agents/specs/<id>-spec.md
```

**What it does:**
1. Reads `PROJECT_CONTEXT.md` and the approach document — must use the defined component library, state management, and styling approach
2. Uses `browser-inspiration` MCP for design references
3. Uses `figma` MCP for design assets
4. Applies `frontend-design` and `brand-guidelines` skills
5. Plans responsive breakpoints, accessibility (WCAG 2.1 AA), animations, and loading/error states
6. Creates `<id>-frontend-plan.md` and `agents/tasks/frontend-todo.md`
7. Passes Gate G2, then hands off to `@executor`

---

### planner-backend

**Purpose:** Plans all backend work: API endpoints, database schema, service layers, error handling, and security.

**How to invoke:** (called automatically by orchestrator)
```
@planner-backend agents/specs/<id>-spec.md
```

**What it does:**
1. Reads `PROJECT_CONTEXT.md` and the approach document — must use the defined framework, ORM, and architectural patterns
2. Inspects the current database schema via Docker exec commands defined in `PROJECT_CONTEXT.md` (DB Access section)
3. Uses `db-migrator` skill for any schema changes (with rollback plans)
4. Uses stack-specific skills (e.g., `golang-pro` for Go projects)
5. Documents full API contracts (request/response shapes, HTTP status codes, error codes)
6. Creates `<id>-backend-plan.md` and `agents/tasks/backend-todo.md`
7. Passes Gate G2, then hands off to `@executor`

---

### executor

**Purpose:** Staff engineer implementer. Works strictly from plans. Never skips tests or security checks.

**How to invoke:** (called automatically by planners)
```
@executor agents/specs/<id>-backend-plan.md
```

**What it does:**
1. Reads plan files + spec + `PROJECT_CONTEXT.md`
2. Updates task status via `todo-manager`
3. Implements each task with minimal code impact
4. After each implementation, runs `test-generator` skill (mandatory)
5. Runs `security-checker` skill before marking any task complete
6. Self-verifies: compiles, tests pass, diff looks correct
7. Passes Gate G3, then hands off to `@tester`

**Self-improvement:** After any correction from user or reviewer, updates `agents/tasks/lessons.md` with the root cause and prevention strategy.

---

### tester

**Purpose:** Executes the full test suite — no simulations, no shortcuts.

**How to invoke:** (called automatically by executor)
```
@tester agents/specs/<id>-spec.md
```

**What it does:**
1. Reads `PROJECT_CONTEXT.md` for test framework, commands, and coverage threshold
2. Resets test database (if applicable) and runs migrations
3. Runs unit + integration + E2E tests via `test-runner` skill
4. Generates coverage report via `coverage-reporter` skill (default threshold: 80%)
5. Logs all results via `test-logger` skill → `agents/logs/`
6. Gate G4: **PASS** → `@reviewer` | **FAIL** → back to `@executor` with failure details

---

### reviewer

**Purpose:** Senior code review with security focus. The only agent that can mark a spec `READY_TO_COMMIT`.

**How to invoke:** (called automatically by tester on pass)
```
@reviewer agents/specs/<id>-spec.md
```

**What it does:**
1. Reads all changed files vs `main`
2. Uses `quick-review` skill (code quality, architecture, performance, error handling, test quality)
3. Runs `security-checker` skill for final security pass
4. Verifies test logs and coverage reports exist and pass
5. Uses `lessons-writer` skill to document any patterns or anti-patterns found
6. Gate G5: **APPROVED** → updates spec to `READY_TO_COMMIT`, notifies user | **CHANGES** → returns to `@executor`

**Does not auto-commit.** The commit step is always manual.

---

### committer

**Purpose:** Final step. Creates a conventional commit, pushes the feature branch, and opens a PR with full test evidence attached.

**How to invoke:** (manual, user-triggered)
```
@committer agents/specs/<id>-spec.md
```

**What it does:**
1. Verifies spec status = `READY_TO_COMMIT` — stops if not
2. Reviews all staged/unstaged changes (never commits `.env`, secrets, or binaries)
3. Uses `commit-changes` skill → conventional commit format (`feat:`, `fix:`, `refactor:`, etc.)
4. Uses `push-changes` skill → creates feature branch, pushes with upstream tracking
5. Uses `create-pr` + `pr-description` skills → PR with test logs, coverage, and security scan linked

---

### hotfix

**Purpose:** Emergency bypass for critical production issues. Skips orchestrator discussion and planners entirely.

**How to invoke:**
```
@hotfix #<issue-number>
```
Or automatically triggered by orchestrator when issue has `hotfix` or `urgent` label.

See [Hotfix Flow](#hotfix-flow) for the full bypass workflow.

---

## Quality Gates

Gates are enforced by the `todo-manager` skill. A gate failure blocks the workflow from advancing.

| Gate | Owner | Requirements |
|------|-------|-------------|
| **G1** | Orchestrator | Spec file exists, intake file exists, approach file exists (feature/bug/refactor only), tasks initialized |
| **G2** | Planners | All planning tasks complete, plan files exist, no blockers |
| **G3** | Executor | All implementation tasks complete, tests created, security check passed, no unresolved TODOs |
| **G4** | Tester | All tests pass, coverage >= threshold (default 80%), test logs saved |
| **G5** | Reviewer | Code review complete, security scan passed, no HIGH severity issues, all tasks done |

**Status lifecycle:**
```
PLANNING → IN_PROGRESS → TESTING → REVIEW → READY_TO_COMMIT → DONE
```

---

## GitHub Issues & Labels Standard

This section defines the conventions used by `issue-crafter`, `orchestrator`, and `committer` for all GitHub issues, labels, branches, and PRs.

### Issue Title Format

```
<type>: <short imperative description>
```

Examples:
```
feat: add JWT authentication to the auth module
fix: resolve race condition in payment processor
refactor: extract user service from monolithic controller
docs: add API authentication guide
chore: upgrade dependencies to latest minor versions
```

Rules:
- Use imperative mood ("add", "fix", "remove" — not "added" or "adding")
- Keep under 72 characters
- No period at the end
- `type` must match one of the issue type labels below

---

### Issue Body Format

All issues created by `issue-crafter` or manually must follow this structure:

```markdown
## Problem Statement
<What is broken or missing? What is the user story?>

## Acceptance Criteria
- [ ] Criterion 1
- [ ] Criterion 2
- [ ] Criterion 3

## Technical Requirements
<Architectural constraints, things to avoid, compliance requirements>

## Design References
<Figma links, mockups, screenshots — or N/A>

## Dependencies
- Related to: #<num>
- Blocked by: #<num>

## Notes
<Edge cases, rollback plan, additional context — or N/A>
```

Rules:
- Acceptance criteria are **mandatory** — never leave this section empty
- Technical Requirements must reference actual constraints from `PROJECT_CONTEXT.md`
- Never skip sections; use "N/A" for empty ones
- `issue-reader` parses this exact structure to build the intake document

---

### Labels

Labels are organized into four namespaced groups. `issue-crafter` creates all labels automatically.

#### Type (what kind of work)

| Label | Color | When to use |
|-------|-------|-------------|
| `feature` | `#0075ca` | New functionality |
| `bug` | `#d73a4a` | Something is broken |
| `refactor` | `#e4e669` | Code improvement without behavior change |
| `docs` | `#0075ca` | Documentation only |
| `test` | `#cfd3d7` | Test additions or improvements |
| `chore` | `#cfd3d7` | Maintenance, dependency updates, tooling |

#### Scope (what area is affected)

| Label | Color | When to use |
|-------|-------|-------------|
| `scope:frontend` | `#bfd4f2` | UI, components, styling only |
| `scope:backend` | `#d4c5f9` | API, database, business logic only |
| `scope:full-stack` | `#d4c5f9` | Both frontend and backend |
| `scope:infrastructure` | `#f9d0c4` | DevOps, CI/CD, deployment |

#### Priority

| Label | Color | When to use |
|-------|-------|-------------|
| `priority:high` | `#b60205` | Must ship this sprint |
| `priority:medium` | `#e99695` | Normal backlog priority |
| `priority:low` | `#f9d0c4` | Nice-to-have, no deadline |

#### Special / Status

| Label | Color | When to use |
|-------|-------|-------------|
| `hotfix` | `#b60205` | Production-critical, triggers bypass flow |
| `urgent` | `#d73a4a` | Same as hotfix — triggers bypass flow |
| `blocked` | `#e4e669` | Waiting on external dependency |
| `needs-clarification` | `#cfd3d7` | Requirements unclear, needs discussion |

---

### Branch Naming

Branches are created automatically by `committer` via the `push-changes` skill.

```
<type>/<id>-<short-desc>
```

| Scenario | Branch format | Example |
|----------|--------------|---------|
| Feature from issue | `feat/issue-<num>-<slug>` | `feat/issue-42-jwt-auth` |
| Bug fix from issue | `fix/issue-<num>-<slug>` | `fix/issue-17-login-race` |
| Refactor from issue | `refactor/issue-<num>-<slug>` | `refactor/issue-55-user-service` |
| Feature from prompt | `feat/task-<slug>` | `feat/task-add-jwt-auth` |
| Hotfix | `hotfix/issue-<num>-<short-desc>` | `hotfix/issue-99-payment-crash` |
| Docs | `docs/issue-<num>-<slug>` | `docs/issue-33-auth-guide` |

Rules:
- Always branch from `main` (or the configured base branch)
- Never push directly to `main` or `master`
- Delete branches after merge

---

### Commit Convention

All commits use the [Conventional Commits](https://www.conventionalcommits.org) format:

```
<type>[!]: <short description>

[optional body]

[optional footer: Closes #<num>]
```

| Type | When to use |
|------|-------------|
| `feat:` | New feature |
| `fix:` | Bug fix |
| `refactor:` | Code change without behavior change |
| `docs:` | Documentation only |
| `test:` | Adding or updating tests |
| `chore:` | Tooling, dependencies, build |
| `perf:` | Performance improvement |
| `ci:` | CI/CD changes |
| `fix!:` / `feat!:` | Breaking change (adds `BREAKING CHANGE` footer) |

Example:
```
feat: add JWT authentication to auth module

Implements RS256 token signing with 15-minute expiry.
Refresh tokens stored in HttpOnly cookies.

Closes #42
```

---

### PR Format

PRs are created automatically by `committer` via `create-pr` + `pr-description` skills.

**Title:** Same as the conventional commit title
**Body structure:**
```markdown
## Summary
- <bullet point 1>
- <bullet point 2>

## Changes
- <file or module changed>: <what changed>

## Test Evidence
- Test run: agents/logs/test-run-<id>-<timestamp>.md
- Coverage: agents/logs/coverage-<id>-<timestamp>.md
- Security: agents/logs/security-<id>-<timestamp>.md

## Checklist
- [ ] Tests pass (Gate G4)
- [ ] Coverage >= 80%
- [ ] Security scan clean (Gate G5)
- [ ] No HIGH severity findings
- [ ] Code reviewed (Gate G5)

Closes #<num>
```

---

### Milestones

Use milestones to group issues by sprint or release:

```
v1.0.0 — Initial release
v1.1.0 — Auth improvements
Sprint 3 — 2026-04-14
```

---

## Skills Reference

Skills are reusable capabilities invoked by agents. All skills read `PROJECT_CONTEXT.md` before acting.

### Setup

| Skill | Used By | What It Does |
|-------|---------|-------------|
| `context-setup` | User (manual) | Interactive conversation to populate `PROJECT_CONTEXT.MD` |

### Core Workflow

| Skill | Used By | What It Does |
|-------|---------|-------------|
| `issue-reader` | orchestrator | Parses GitHub issues into structured intake documents |
| `todo-manager` | All agents | Task tracking, gate verification, status transitions |
| `lessons-writer` | executor, reviewer | Appends learnings to `agents/tasks/lessons.md` |

### Testing

| Skill | Used By | What It Does |
|-------|---------|-------------|
| `test-generator` | executor | Generates unit, integration, and E2E tests for changed files |
| `test-runner` | tester | Executes the full test suite using commands from `PROJECT_CONTEXT.md` |
| `test-logger` | tester | Saves test run results to `agents/logs/test-run-<id>-<timestamp>.md` |
| `coverage-reporter` | tester | Generates coverage reports, flags gaps (threshold from `PROJECT_CONTEXT.md`) |

### Security

| Skill | Used By | What It Does |
|-------|---------|-------------|
| `security-checker` | executor, reviewer | OWASP Top 10 check + automated scan, saves to `agents/logs/security-<id>.md` |
| `dependency-auditor` | Any agent | CVE scan, outdated packages, license compliance |

### Git & GitHub

| Skill | Used By | What It Does |
|-------|---------|-------------|
| `commit-changes` | committer | Stages files safely, creates conventional commit with issue reference |
| `push-changes` | committer | Creates feature branch, syncs with remote, pushes safely (never to main) |
| `create-pr` | committer | Opens PR via `gh` CLI with full test evidence and checklist |
| `pr-description` | committer | Formats PR body (summary, changes, test results, security, checklist) |

### Planning & Design

| Skill | Used By | What It Does |
|-------|---------|-------------|
| `frontend-design` | planner-frontend | Design direction, typography, color, motion, accessibility checklist |
| `brand-guidelines` | planner-frontend | CSS design tokens (colors, spacing, radius, shadows, transitions) |
| `db-migrator` | planner-backend | Safe database migration planning with rollback procedures |
| `quick-review` | reviewer | Structured code review with severity classification (CRITICAL/HIGH/MEDIUM/LOW) |

### Stack-Specific

| Skill | Used By | What It Does |
|-------|---------|-------------|
| `golang-pro` | planner-backend, executor | Go 1.21+ idioms, concurrency patterns, table-driven tests, race detection |
| `hotfix-mode` | hotfix agent | Abbreviated workflow: minimal fix + regression test + fast-track commit |
| `senior-engineer-executor` | (skill variant) | Same implementation workflow as executor agent |

---

## File Structure

For each task, the workflow generates:

```
agents/
├── specs/
│   ├── <id>-intake.md          # Parsed issue or inline requirements
│   ├── <id>-approach.md        # Technical approach decision record
│   ├── <id>-spec.md            # Technical specification
│   ├── <id>-frontend-plan.md   # Frontend implementation plan
│   └── <id>-backend-plan.md    # Backend implementation plan
├── tasks/
│   ├── todo.md                 # Main task tracker (all agents)
│   ├── frontend-todo.md        # Frontend-specific tasks
│   ├── backend-todo.md         # Backend-specific tasks
│   ├── lessons.md              # Patterns and pitfalls discovered
│   └── migration-plan-<id>.md  # DB migration plan (if applicable)
└── logs/
    ├── test-run-<id>-<ts>.md   # Test execution results
    ├── coverage-<id>-<ts>.md   # Coverage report
    └── security-<id>-<ts>.md   # Security scan report
```

**Identifier (`<id>`) convention:**

| Trigger | `<id>` | Example files |
|---------|--------|--------------|
| `@orchestrator #42` | `issue-42` | `issue-42-spec.md`, `issue-42-approach.md` |
| `@orchestrator <prompt>` | `task-<slug>` | `task-add-jwt-auth-spec.md` |

The spec file (`<id>-spec.md`) acts as the single source of truth for each task. Its `## Status:` field drives the workflow:

```
PLANNING → IN_PROGRESS → TESTING → REVIEW → READY_TO_COMMIT → DONE
```

---

## Configuration

### Quick Setup (Recommended)

Instead of filling `PROJECT_CONTEXT.md` manually, use the `context-setup` skill:

```
context-setup
```

The skill opens an interactive conversation, asks about your stack, architecture, and tooling, offers smart suggestions, and writes the file for you after explicit approval. Run it once when starting a new project, or whenever the tech stack changes.

### Step 1: Fill in PROJECT_CONTEXT.md

This is the **single source of truth** for all agents. Every agent reads it before acting.

```markdown
## 1. Project Overview
- Description, objectives, target audience

## 2. Technology Stack
### Frontend
- Framework, State Management, Styling, UI Components, Build Tool, Testing
- Test Command, E2E Command, Coverage Command

### Backend
- Language, Framework, ORM/Query, Auth, Testing
- Test Command, Run Migrations, Test DB Reset, Security Scanner, Coverage Command

## 3. Architecture and Patterns
- Architectural pattern (Clean Architecture, MVC, etc.)
- Folder structure, API patterns, error handling conventions

## 4. Coding Standards
- Linting and formatting commands, naming conventions, commit convention

## 10. Lessons Learned
- Populated automatically by lessons-writer skill
```

### Step 2: Configure .opencode/opencode.json

```json
{
  "settings": {
    "autoCompact": true,
    "autoHandoff": true,
    "maxTokensPerAgent": 80000,
    "plugin": ["oh-my-opencode"]
  },
  "permission": {
    "skill": { "*": "allow" }
  },
  "mcp": {
    "figma": {
      "type": "http",
      "url": "https://mcp.figma.com/mcp",
      "enabled": true
    },
    "browser-inspiration": {
      "type": "remote",
      "url": "https://firecrawl.dev/mcp",
      "enabled": true
    }
  }
}
```

Disable MCPs you don't need by setting `"enabled": false`.

### Step 3: Authenticate GitHub CLI

All agents that interact with GitHub require the `gh` CLI:

```bash
gh auth login
gh auth status  # verify
```

---

## Hotfix Flow

Use this path for **production-critical** issues only: service down, critical security vulnerability, data loss, imminent SLA breach.

```
@hotfix #<issue-number>
```

**What gets bypassed:** orchestrator technical discussion, full planning (no spec review, no plan files)

**Abbreviated flow:**

```
hotfix triggered
     │
     ▼
EXECUTOR (direct)
  - Read issue description
  - Investigate root cause (15-minute time-box)
  - Implement MINIMAL fix (no refactoring, no feature additions)
  - Create regression test that reproduces the bug
  - Run abbreviated security check on changed files
     │
     ▼
TESTER (fast-track)
  - Run affected test suite only
  - Run the new regression test
  - Smoke test critical paths
     │
     ▼
COMMITTER (immediate)
  - Branch: hotfix/issue-<num>-<short-desc>
  - Commit: fix!: <description>
  - PR: labelled hotfix, marked for priority review
```

**Hotfix quality gates (abbreviated but mandatory):**
- Regression test exists and passes
- No new security vulnerabilities introduced
- Affected tests pass
- Code compiles/builds

**After merge:**
1. Monitor metrics for 30 minutes
2. Create follow-up issues for proper fix, root cause analysis, and process improvement

---

## Customization

### Add a New Agent

Create `.opencode/agents/my-agent.md` with frontmatter:

```markdown
---
description: What this agent does.
mode: subagent  # or: primary
model: anthropic/claude-sonnet-4-6
---
## My Agent Workflow
...
```

### Add a New Skill

Create `.opencode/skills/my-skill/SKILL.md`:

```markdown
---
name: my-skill
description: What this skill does.
---
## My Skill
...
```

### Change an Agent's Model

Edit the `model:` field in the agent's frontmatter:

```yaml
---
model: anthropic/claude-opus-4-6   # more capable
model: anthropic/claude-haiku-4-5  # faster, cheaper
model: local/qwen2.5-coder:14b     # local model
---
```

### Add a New MCP

Add to `.opencode/opencode.json`:

```json
"mcp": {
  "my-tool": {
    "type": "local",
    "command": ["my-command", "--arg"],
    "enabled": true,
    "description": "What this MCP provides"
  }
}
```

### Adjust Coverage Threshold

In `PROJECT_CONTEXT.md`, set the coverage threshold in the Technology Stack section. The `test-runner` and `coverage-reporter` skills read this value.

---

## Troubleshooting

### Issue not being parsed correctly

- Verify the issue body uses the standard format (sections: Problem Statement, Acceptance Criteria, Technical Requirements, Design References, Dependencies, Notes)
- Use `@issue-crafter` to create the next issue — it produces exactly the format `issue-reader` expects
- Check that `gh auth status` is authenticated

### Tests failing at Gate G4

- Check `agents/logs/test-run-<id>-*.md` for the failure details
- The tester provides probable causes and suggested fixes
- The executor will be re-invoked automatically with the failure report

### Coverage below threshold

- Check `agents/logs/coverage-<id>-*.md` for which lines are uncovered
- The tester returns to the executor with a coverage report
- The executor will add tests for the identified gaps

### Security scan blocking Gate G3 or G5

- Check `agents/logs/security-<id>-*.md` for findings
- HIGH and CRITICAL findings block the gate — they must be fixed
- MEDIUM findings are logged but not blocking by default

### Spec not reaching READY_TO_COMMIT

- Check Gate G5 status in `agents/tasks/todo.md`
- Check if the reviewer requested changes (executor loop)
- Confirm the security scan is passing
- All tasks in todo.md must be marked complete

### PR creation fails

- Run `gh auth status` to verify authentication
- Confirm the spec status is `READY_TO_COMMIT` before invoking `@committer`
- Check if a PR already exists for the branch (`gh pr list`)

---

## Project Checklist

- [ ] Clone the template
- [ ] Run `context-setup` skill (or fill `PROJECT_CONTEXT.md` manually)
- [ ] Configure `.opencode/opencode.json` (adjust models, enable/disable MCPs)
- [ ] Authenticate GitHub CLI: `gh auth login`
- [ ] Create first issue with `@issue-crafter` or go directly with `@orchestrator <prompt>`
- [ ] Run `@orchestrator #<num>` (or `@orchestrator <prompt>`) and monitor the flow
- [ ] When spec is `READY_TO_COMMIT`, trigger `@committer`
- [ ] Review `agents/tasks/lessons.md` after each cycle

---

## Additional Reference

| File | Purpose |
|------|---------|
| `PROJECT_CONTEXT.md` | Single source of truth for all agents — fill this out first |
| `.opencode/agents/` | Agent definitions (workflows, skills available, handoff rules) |
| `.opencode/skills/` | Skill definitions (reusable capabilities invoked by agents) |
| `.opencode/opencode.json` | Runtime configuration (models, MCPs, settings) |
| `agents/tasks/lessons.md` | Patterns and pitfalls discovered across issues |
| `agents/logs/` | Test, coverage, and security scan artifacts |
