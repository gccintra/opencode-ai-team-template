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
  - [Environment Variables](#environment-variables)
- [Hotfix Flow](#hotfix-flow)
- [Customization](#customization)
- [Troubleshooting](#troubleshooting)

---

## Overview

This template gives you a complete AI engineering team operating inside [OpenCode](https://opencode.ai). Each agent has a specific role, a defined model, and strict quality gates that prevent the workflow from advancing until requirements are met.

**The team:**

| Agent | Role | Model | Mode |
|-------|------|-------|------|
| `project-setup` | Entry point for new projects — analyzes codebase, discusses architecture, creates PROJECT_CONTEXT.md | claude-sonnet-4-6 | primary |
| `issue-crafter` | Discusses requirements and creates GitHub issues | claude-sonnet-4-6 | primary |
| `orchestrator` | Reads issues or prompts, discusses approach, plans everything, routes to executor | gemini-3-pro | primary |
| `executor` | Implements code with mandatory tests and security checks | claude-sonnet-4-6 | subagent |
| `tester` | Runs tests, generates coverage reports, logs results | minimax-m2.5 | subagent |
| `reviewer` | Code review, final security scan, marks READY_TO_COMMIT | minimax-m2.5 | subagent |
| `committer` | Creates commit, pushes branch, opens PR (manual trigger only) | minimax-m2.5 | subagent |
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
 │
 │     [Technical Discussion — skipped for hotfix/docs/chore/test]
 │     Asks what the user has in mind for the approach
 │     Evaluates user's idea against architecture — validates, challenges, or refines
 │     If user has no opinion: decides autonomously with justification
 │
 │     Creates agents/tasks/<id>.md (SINGLE FILE with spec + plan + tasks + evidence)
 │     [Gate G1]
 │     └── Delegates directly to executor
 │
 ├── @executor (called by orchestrator via task())
 │     Reads agents/tasks/<id>.md + PROJECT_CONTEXT.md
 │     Implements each task from the ### Tasks section
 │     Uses test-generator skill after each implementation
 │     Runs security-checker skill on changed files
 │     Marks checkboxes as complete in task file
 │     [Gate G3]
 │     └── Hands off to @tester
 │
 ├── @tester (called by executor via task())
 │     Reads PROJECT_CONTEXT.md for test commands and thresholds
 │     Runs full test suite (unit + integration + E2E)
 │     Uses coverage-reporter skill (threshold: 80%)
 │     Uses test-logger skill → agents/logs/
 │     Updates Evidence section in task file
 │     [Gate G4]
 │     ├── PASS → @reviewer
 │     └── FAIL → back to @executor
 │
 ├── @reviewer (called by tester via task())
 │     Reads all changed files vs main
 │     Uses quick-review skill (code quality, architecture, performance)
 │     Runs security-checker skill (final pass)
 │     Verifies test logs exist and pass
 │     Uses lessons-writer skill for patterns discovered
 │     [Gate G5]
 │     ├── APPROVED → marks task as READY_TO_COMMIT, notifies user, STOPS
 │     └── CHANGES REQUESTED → back to @executor
 │
 └── @committer agents/tasks/<id>.md   (MANUAL — user-triggered ONLY)
       Verifies task status = READY_TO_COMMIT
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
          ┌──────────▼──────────┐
          │  CODEBASE RESEARCH  │
          │  grep / glob / read │
          └──────────┬──────────┘
                     │
                     ▼
         ┌───────────────────────┐
         │      ORCHESTRATOR     │
         │  Plans EVERYTHING     │
         │  (no separate planners)│
         │  Creates single file: │
         │  agents/tasks/<id>.md │
         │  [Gate G1]            │
         └──────────┬────────────┘
                    │
                    ▼
         ┌───────────────────────┐
         │       EXECUTOR        │
         │  Implements all tasks │
         │  test-generator       │
         │  security-checker     │
         │  [Gate G3]            │
         └──────────┬────────────┘
                    │
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
         │  READY_TO_COMMIT      │
         │  → notify user        │
         │  → STOP (no auto-commit)│
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

### Step 0: Project Setup (First Time Only)

**Initialize your PROJECT_CONTEXT.md:**

```
@project-setup
```

The agent analyzes your codebase, detects the tech stack from config files (package.json, go.mod, requirements.txt, etc.), and guides you through an interactive conversation to populate `PROJECT_CONTEXT.MD` — the single source of truth that all other agents read before acting.

**Quick mode (auto-fill with sensible defaults):**
```
@project-setup --quick
```

**What it detects automatically:**
- Frontend: Next.js, React, Vue, Angular, Svelte, Vite, TailwindCSS, etc.
- Backend: Go (Gin/Echo/Chi), Python (FastAPI/Django), Node.js (Express), etc.
- Database: PostgreSQL, MySQL, MongoDB, Redis, etc.
- Testing: Vitest, Jest, Playwright, Cypress, pytest, go test
- CI/CD: GitHub Actions, GitLab CI, Jenkins
- Build commands: from package.json, Makefile, docker-compose

---

### Option A: Full issue-tracked flow

**1. Create and craft an issue**

```
@issue-crafter
```

**2. Start the automated workflow**

```
@orchestrator #<issue-number>
```

The orchestrator parses the issue, discusses approach, creates a unified task file, and delegates to executor. The flow continues automatically through executor → tester → reviewer.

**3. Commit and open the PR**

When the reviewer marks the task as `READY_TO_COMMIT`:

```
@committer agents/tasks/<id>.md
```

---

### Option B: Prompt-only flow (no GitHub issue)

```
@orchestrator add password reset flow to the auth module
```

The orchestrator asks 4 clarifying questions, creates a unified task file, and continues through the full workflow.

---

## Agents

### project-setup

**Purpose:** Entry point for new projects. Analyzes the codebase, detects the tech stack, discusses architecture, creates `PROJECT_CONTEXT.MD`.

**How to invoke:**
```
@project-setup
```

---

### issue-crafter

**Purpose:** Conducts a conversation to gather requirements, proposes technical approaches, creates a GitHub issue in the standard format.

**How to invoke:**
```
@issue-crafter
```

---

### orchestrator

**Purpose:** Staff engineer coordinator AND planner. Does not write code. Plans ALL implementation details and delegates to executor.

**How to invoke:**
```
@orchestrator #<issue-number>           # issue-based
@orchestrator <plain text description>  # prompt-based
```

**Key behavior:**
- NEVER writes code — only plans and delegates
- Creates a single unified task file: `agents/tasks/<id>.md`
- Delegates to executor via `task()` with appropriate category and skills

---

### executor

**Purpose:** Staff engineer implementer. Works from the unified task file.

**How to invoke:** Called automatically by orchestrator via `task()`

**Key behavior:**
- Reads `agents/tasks/<id>.md` for all tasks
- Marks checkboxes as complete in the task file
- Mandatory test generation and security checks
- Hands off to tester when done

---

### tester

**Purpose:** Executes the full test suite.

**How to invoke:** Called automatically by executor via `task()`

**Key behavior:**
- Runs all tests, generates coverage reports
- Updates Evidence section in task file
- PASS → reviewer | FAIL → back to executor

---

### reviewer

**Purpose:** Senior code review. Marks task as `READY_TO_COMMIT` and STOPS. Does NOT auto-commit.

**How to invoke:** Called automatically by tester via `task()`

**Key behavior:**
- Reviews code, runs security scan
- APPROVED → marks READY_TO_COMMIT, notifies user, **STOPS**
- CHANGES → returns to executor

---

### committer

**Purpose:** Creates commit, pushes branch, opens PR. **Manual trigger only.**

**How to invoke:** (user-triggered)
```
@committer agents/tasks/<id>.md
```

---

### hotfix

**Purpose:** Emergency bypass for critical production issues.

**How to invoke:**
```
@hotfix #<issue-number>
```

---

---

## Quality Gates

| Gate | Owner | Requirements |
|------|-------|-------------|
| **G1** | Orchestrator | Task file exists, problem clear, criteria defined, tasks listed |
| **G3** | Executor | All checkboxes `[x]`, tests created, security passed |
| **G4** | Tester | All tests pass, coverage >= 80%, logs saved, evidence updated |
| **G5** | Reviewer | Review complete, security passed, no HIGH issues |

**Status lifecycle:**
```
PLANNING → IN_PROGRESS → TESTING → REVIEW → READY_TO_COMMIT → DONE
```

---

## GitHub Issues & Labels Standard

### Issue Title Format

```
<type>: <short imperative description>
```

### Issue Body Format

```markdown
## Problem Statement
## Acceptance Criteria
## Technical Requirements
## Design References
## Dependencies
## Notes
```

### Labels

| Group | Labels |
|-------|--------|
| **Type** | `feature`, `bug`, `refactor`, `docs`, `test`, `chore` |
| **Scope** | `scope:frontend`, `scope:backend`, `scope:full-stack`, `scope:infrastructure` |
| **Priority** | `priority:high`, `priority:medium`, `priority:low` |
| **Special** | `hotfix`, `urgent`, `blocked`, `needs-clarification` |

### Branch Naming

```
<type>/<id>-<short-desc>
```

### Commit Convention

[Conventional Commits](https://www.conventionalcommits.org): `feat:`, `fix:`, `refactor:`, `docs:`, `test:`, `chore:`

---

## Skills Reference

| Skill | Used By | What It Does |
|-------|---------|-------------|
| `issue-reader` | orchestrator | Parses GitHub issues |
| `todo-manager` | All agents | Task tracking via unified task file |
| `lessons-writer` | executor, reviewer | Appends learnings to PROJECT_CONTEXT.md |
| `test-generator` | executor | Generates tests |
| `test-runner` | tester | Executes test suite |
| `test-logger` | tester | Saves results to `agents/logs/` |
| `coverage-reporter` | tester | Generates coverage reports |
| `security-checker` | executor, reviewer | OWASP checks |
| `hotfix-mode` | hotfix agent | Expedited fix workflow, skips planning |
| `figma-implement-design` | executor | Translates Figma designs into production code |
| `html-to-figma` | executor | Builds HTML screens with market-standard design and inserts into Figma |
| `commit-changes` | committer | Creates conventional commit |
| `push-changes` | committer | Creates branch, pushes |
| `create-pr` | committer | Opens PR with evidence |
| `pr-description` | committer | Formats PR body |
| `frontend-design` | executor | Design direction, accessibility |
| `db-migrator` | executor | Database migration planning |
| `quick-review` | reviewer | Structured code review |

---

## File Structure

```
.env.example          # Template for environment variables (copy to .env)
agents/
├── tasks/
│   └── <id>.md                 # UNIFIED task file (spec + plan + todos + evidence)
└── logs/
    ├── test-run-<id>-<ts>.md   # Test execution results
    ├── coverage-<id>-<ts>.md   # Coverage report
    └── security-<id>-<ts>.md   # Security scan report
```

**Identifier convention:**

| Trigger | `<id>` | Example |
|---------|--------|---------|
| `@orchestrator #42` | `issue-42` | `agents/tasks/issue-42.md` |
| `@orchestrator <prompt>` | `task-<slug>` | `agents/tasks/task-add-jwt-auth.md` |

---

## Configuration

### Quick Setup

```
@project-setup
```

### Environment Variables

This project uses environment variables to keep sensitive credentials out of the repository.

**1. Copy the example file:**

```bash
cp .env.example .env
```

**2. Fill in your credentials in `.env`:**

| Variable | Description | How to get it |
|----------|-------------|---------------|
| `FIGMA_CLIENT_ID` | Figma OAuth client ID | [Figma Developer Apps](https://www.figma.com/developers/apps) |
| `FIGMA_CLIENT_SECRET` | Figma OAuth client secret | Same as above |
| `FIRECRAWL_API_KEY` | Firecrawl MCP API key | [Firecrawl](https://www.firecrawl.dev/) |

> **Note:** The `.env` file is already in `.gitignore` and will not be committed. Never commit real credentials.

### opencode.json

The MCP configuration in `.opencode/opencode.json` references environment variables instead of hardcoded secrets:

```json
{
  "permission": {
    "skill": { "*": "allow" }
  },
  "mcp": {
    "figma": {
      "type": "remote",
      "url": "https://mcp.figma.com/mcp",
      "enabled": true,
      "oauth": {
        "clientId": "${FIGMA_CLIENT_ID}",
        "clientSecret": "${FIGMA_CLIENT_SECRET}"
      }
    },
    "firecrawl": {
      "type": "remote",
      "url": "https://mcp.firecrawl.dev/${FIRECRAWL_API_KEY}/v2/mcp",
      "enabled": true
    }
  }
}
```

### GitHub CLI

```bash
gh auth login
gh auth status
```

---

## Hotfix Flow

```
@hotfix #<issue-number>
```

**Abbreviated flow:** executor (minimal fix + regression test) → tester (affected tests only) → user triggers `@committer`

---

## Customization

### Add a New Agent

Create `.opencode/agents/my-agent.md` with frontmatter:

```markdown
---
description: What this agent does.
mode: subagent
model: anthropic/claude-sonnet-4-6
---
```

### Add a New Skill

Create `.opencode/skills/my-skill/SKILL.md`

### Change an Agent's Model

Edit the `model:` field in the agent's frontmatter.

---

## Troubleshooting

### Orchestrator is implementing instead of delegating

The orchestrator MUST NOT write code. Check that it only uses `read`, `glob`, `grep`, and `task` tools.

### Tests failing at Gate G4

Check `agents/logs/test-run-<id>-*.md` for details. The tester returns to executor automatically.

### Task not reaching READY_TO_COMMIT

Check `agents/tasks/<id>.md` — verify all checkboxes are `[x]` and Evidence section is filled.

### PR creation fails

Run `gh auth status`. Confirm task status is `READY_TO_COMMIT` before `@committer`.
