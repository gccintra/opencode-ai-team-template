# OpenCode AI Team Template

A robust template for AI agent orchestration for full-stack development with complete coding vibe flow: planning → implementation → tests → review → commit.

---

## 🎯 Overview

This template implements a complete workflow where specialized agents collaborate to deliver quality features:

1. **Orchestrator** (Lead) - Reads issues, creates specifications, routes to planners
2. **Planner Frontend/Backend** - Plans architecture and decomposes tasks
3. **Executor** (Claude Opus) - Implements code with mandatory tests
4. **Tester** (Qwen local) - Executes tests and generates reports
5. **Reviewer** (Qwen local) - Final code review, marks as ready
6. **Committer** (Manual) - Creates commit and PR when triggered by the user

---

## 🚀 How to Use

### Start a new feature/fix

```bash
# 1. Create issue on GitHub
# 2. Invoke the orchestrator
@orchestrator #<issue-number>

# 3. Monitor the automatic flow (Orchestrator → Planners → Executor → Tester → Reviewer)

# 4. When spec is READY_TO_COMMIT, create commit and PR
@committer agents/specs/issue-<num>-spec.md
```

### Automatic Flow

```
┌─────────────────────────────────────────────┐
│ GITHUB ISSUE                                │
└──────────────────────┬──────────────────────┘
                       │
                       ▼
            ┌──────────────────────┐
            │   ORCHESTRATOR       │
            │ - Parse issue        │
            │ - Create spec        │
            │ - Route to planners  │
            └──────────┬───────────┘
                   ┌───┴────┐
                   ▼        ▼
        ┌──────────────┐  ┌──────────────┐
        │ PLANNER-FE   │  │ PLANNER-BE   │
        │ Frontend     │  │ Backend      │
        │ planning     │  │ planning     │
        └──────┬───────┘  └────────┬─────┘
               └────────┬─────────┘
                        ▼
            ┌──────────────────────┐
            │    EXECUTOR          │
            │ - Implement code     │
            │ - Generate tests     │
            │ - Security check     │
            └──────────┬───────────┘
                       ▼
            ┌──────────────────────┐
            │     TESTER           │
            │ - Run tests          │
            │ - Generate coverage  │
            │ - Log results        │
            └──────────┬───────────┘
               Pass │ Fail
                    │   └──→ EXECUTOR (fix)
                    ▼
            ┌──────────────────────┐
            │    REVIEWER          │
            │ - Code review        │
            │ - Security final     │
            │ - Mark READY_COMMIT  │
            └──────────────────────┘
```

### Manual Flow (Commit/PR)

```
USER: @committer agents/specs/issue-<num>-spec.md
                        │
                        ▼
            ┌──────────────────────┐
            │    COMMITTER         │
            │ - Create commit      │
            │ - Push to remote     │
            │ - Create PR          │
            └──────────────────────┘
                        │
                        ▼
            ┌──────────────────────┐
            │  GITHUB PULL REQUEST │
            │  (with test logs)    │
            └──────────────────────┘
```

---

## 📁 Generated File Structure

For each issue, the system creates:

```
agents/
├── specs/
│   ├── issue-42-intake.md          # Issue parse by issue-reader
│   ├── issue-42-spec.md            # Technical specification
│   ├── issue-42-frontend-plan.md   # Frontend plan (if applicable)
│   └── issue-42-backend-plan.md    # Backend plan (if applicable)
├── tasks/
│   ├── todo.md                     # Main tasks
│   ├── frontend-todo.md            # Frontend tasks
│   ├── backend-todo.md             # Backend tasks
│   └── lessons.md                  # Lessons learned
└── logs/
    ├── test-run-42-<timestamp>.md  # Test results
    ├── coverage-42-<timestamp>.md  # Coverage report
    └── security-42-<timestamp>.md  # Security scan
```

---

## ⚙️ Project Configuration

### 1. Customize PROJECT_CONTEXT.md

The `PROJECT_CONTEXT.md` is the **single source of truth** for the agent. Customize with your technologies and standards:

```markdown
## Overview
- Project description
- Main objectives
- Target audience

## Technology Stack
- Frontend: React, Vue, etc.
- Backend: Node, Python, Go, etc.
- Database: PostgreSQL, MongoDB, etc.
- Testing: Jest, Pytest, Playwright, etc.

## Architecture and Patterns
- Architectural pattern (Clean Architecture, etc.)
- State management
- API patterns
- Authentication and authorization
- Folder structure

## Coding Standards and Conventions
- Linting and formatting
- Error handling
- Naming conventions
- Commit convention
```

### 2. Configure .opencode/opencode.json

Adjust the models and MCPs:

```json
{
  "settings": {
    "autoCompact": true,
    "autoHandoff": true,
    "maxTokensPerAgent": 80000
  },
  "mcp": {
    "git-integration": {...},
    "testing-tools": {...},
    "db-query": {...}
  },
  "agents": {
    "orchestrator": {...},
    "planner-frontend": {...},
    "executor": {...}
  }
}
```

---

## 🎓 Detailed Workflow

### Gate 1: Orchestrator
- ✅ Issue parse
- ✅ Spec created in `agents/specs/issue-<num>-spec.md`
- ✅ Tasks initialized in `agents/tasks/todo.md`

### Gate 2: Planners
- ✅ Frontend/backend plan completed
- ✅ Task decomposition finalized
- ✅ Architecture validated against PROJECT_CONTEXT.md

### Gate 3: Executor
- ✅ Implementation completed
- ✅ Tests generated mandatorily
- ✅ No TODO comments without issue
- ✅ Security check passed

### Gate 4: Tester
- ✅ All tests pass
- ✅ Coverage >= 80%
- ✅ Logs saved in `agents/logs/`

### Gate 5: Reviewer
- ✅ Code review completed
- ✅ Final security scan passed
- ✅ Spec marked as READY_TO_COMMIT

### Manual: Committer
- ✅ Conventional commit created
- ✅ Push to remote
- ✅ PR created with test evidence

---

## 🛠️ Available Skills

### Base Flow
| Skill | Agent | Function |
|-------|--------|--------|
| `issue-reader` | Orchestrator | Parse GitHub issues |
| `todo-manager` | All | Task tracking |
| `lessons-writer` | Reviewer | Document learnings |

### Tests
| Skill | Agent | Function |
|-------|--------|--------|
| `test-generator` | Executor | Generate tests |
| `test-runner` | Tester | Execute tests |
| `test-logger` | Tester | Log results |
| `coverage-reporter` | Tester | Coverage report |

### Git/GitHub
| Skill | Agent | Function |
|-------|--------|--------|
| `commit-changes` | Committer | Create commit |
| `push-changes` | Committer | Push to remote |
| `create-pr` | Committer | Create Pull Request |
| `pr-description` | Committer | Format description |

### Security & Extras
| Skill | Agent | Function |
|-------|--------|--------|
| `security-checker` | Executor/Reviewer | Verify OWASP |
| `hotfix-mode` | Executor | Quick mode for emergencies |
| `db-migrator` | Planner Backend | Manage migrations |
| `dependency-auditor` | Any | Audit dependencies |

### Design & Review
| Skill | Agent | Function |
|-------|--------|--------|
| `frontend-design` | Planner Frontend | Design guidelines |
| `brand-guidelines` | Planner Frontend | Brand tokens |
| `quick-review` | Reviewer | Quick code review |
| `golang-pro` | Executor | Go best practices |

---

## 💾 Configured MCPs

| MCP | Type | Purpose |
|-----|------|----------|
| `git-integration` | remote | GitHub API access |
| `github-cli` | local | gh CLI commands |
| `testing-tools` | local | Test execution |
| `test-coverage` | local | Coverage reports |
| `db-query` | remote | Database queries |
| `browser-inspiration` | remote | Design references |
| `linter-security` | local | Linting and security |
| `commit-tools` | local | Git operations |

---

## 🔧 Common Customizations

### Add New Skill

1. Create `agents/skills/new-skill.md`
2. Add to agent in `.opencode/opencode.json`:
```json
"executor": {
  "skills": ["test-generator", "new-skill"]
}
```

### Change Agent Model

```json
"executor": {
  "model": "anthropic/claude-opus-4.6",  // or other model
  "maxTokensPerAgent": 80000
}
```

### Add New MCP

```json
"mcp": {
  "my-mcp": {
    "type": "local",
    "command": ["my-command"],
    "enabled": true
  }
}
```

---

## 📋 Checklist for New Project

- [ ] Clone template
- [ ] Customize `PROJECT_CONTEXT.md` with tech stack and standards
- [ ] Configure `.opencode/opencode.json` with desired models
- [ ] Create first issue on GitHub
- [ ] Invoke `@orchestrator` to test flow
- [ ] Monitor progress through logs
- [ ] Review `agents/tasks/lessons.md` and adjust if necessary

---

## 🚨 Troubleshooting

### Issue not being parsed correctly

→ Check issue format on GitHub
→ Confirm that `issue-reader` is being invoked

### Failing Tests

→ Check logs in `agents/logs/test-run-<num>-*.md`
→ Review error messages
→ Executor will be called again for fix

### Spec not marking as READY_TO_COMMIT

→ Check gate G5 in `agents/tasks/todo.md`
→ Confirm that security-checker passed
→ Code review may have requested changes

### PR not creating

→ Check gh CLI authentication
→ Confirm that spec status is READY_TO_COMMIT
→ Check committer logs

---

## 📚 Additional Reading

- `PROJECT_CONTEXT.MD` - Project technical configurations
- `agents/skills/` - Documentation for each skill
- `agents/tasks/lessons.md` - Discovered patterns and gotchas
- `.opencode/opencode.json` - Agents and MCPs configuration

---

## 📝 Notes

- Each issue generates its own specs and logs
- Tasks are tracked with `todo-manager`
- Lessons learned are documented in `lessons.md`
- The flow is automatic except for commit/PR (manual via `@committer`)
- Spec must be in `READY_TO_COMMIT` before invoking committer
