---
name: todo-manager
description: Manages task tracking in todo.md files, verifies completion gates, and blocks progression when tasks are incomplete.
---
## Todo Manager Skill

Central task tracking system for the development workflow. Manages tasks, verifies completion, and enforces gates.

### Files Managed
- `agents/tasks/todo.md` - Main task list
- `agents/tasks/frontend-todo.md` - Frontend-specific tasks
- `agents/tasks/backend-todo.md` - Backend-specific tasks

### Task Format
```markdown
## Issue #<num>: <title>

### Status: <PLANNING|IN_PROGRESS|TESTING|REVIEW|READY_TO_COMMIT|DONE>

### Tasks
- [ ] Task 1 description
- [ ] Task 2 description
- [x] Task 3 (completed)

### Subtasks (if needed)
#### Frontend
- [ ] Component implementation
- [ ] Styling

#### Backend
- [ ] API endpoint
- [ ] Database migration

### Blockers
- <blocker description if any>

### Notes
- <relevant notes>

---
Last updated: <timestamp>
Updated by: <agent-name>
```

### Operations

#### Initialize Tasks
When starting a new issue:
```
todo-manager init --issue <num> --title "<title>"
```
Creates the base structure in `agents/tasks/todo.md`

#### Add Task
```
todo-manager add --task "Description" [--category frontend|backend]
```

#### Complete Task
```
todo-manager complete --task "Description"
```
Marks task as done with timestamp

#### Check Status
```
todo-manager status --issue <num>
```
Returns completion percentage and pending tasks

#### Verify Gate
```
todo-manager gate --check <gate-name>
```
Returns PASS/FAIL based on gate criteria

### Gates

#### G1: Orchestrator Gate
- Spec file exists: `agents/specs/issue-<num>-spec.md`
- Intake file exists: `agents/specs/issue-<num>-intake.md`
- Tasks initialized in todo.md

#### G2: Planner Gate
- All planning tasks complete
- Plan files exist (frontend-plan.md and/or backend-plan.md)
- No blockers flagged

#### G3: Executor Gate
- All implementation tasks complete
- Tests created for new code
- No TODO comments left in code (except intentional)

#### G4: Tester Gate
- All tests pass
- Coverage >= 80% (or project threshold)
- Test logs saved to `agents/logs/`

#### G5: Reviewer Gate
- Code review complete
- Security scan passed
- Spec status = READY_TO_COMMIT

### Status Transitions
```
PLANNING → IN_PROGRESS → TESTING → REVIEW → READY_TO_COMMIT → DONE
```

Only allowed transitions:
- PLANNING → IN_PROGRESS (when G2 passes)
- IN_PROGRESS → TESTING (when G3 passes)
- TESTING → IN_PROGRESS (when tests fail, return to executor)
- TESTING → REVIEW (when G4 passes)
- REVIEW → IN_PROGRESS (when changes requested)
- REVIEW → READY_TO_COMMIT (when G5 passes)
- READY_TO_COMMIT → DONE (after commit/PR created)

### Blocking Behavior
**CRITICAL**: If a gate check fails, the workflow MUST stop.

Output format for blocked gate:
```
## GATE BLOCKED: <gate-name>

**Missing requirements:**
- [ ] <requirement 1>
- [ ] <requirement 2>

**Action required:** <what needs to be done>
**Return to:** <agent that should fix this>
```

### Update Spec Status
When changing spec status, update the spec file:
```markdown
<!-- In agents/specs/issue-<num>-spec.md -->
## Status: READY_TO_COMMIT
```

### Usage Examples

**Orchestrator initializing:**
```
Use todo-manager to init issue #42 with title "Add user authentication"
```

**Executor completing task:**
```
Use todo-manager to complete "Implement login endpoint"
```

**Tester checking gate:**
```
Use todo-manager to verify gate G4
```

### Output
Always output current status after any operation:
```
## Todo Status: Issue #<num>

Progress: [████████░░] 80% (8/10 tasks)
Current Phase: TESTING
Pending Tasks:
- [ ] Write E2E tests
- [ ] Update documentation

Gate G4: PENDING (awaiting test completion)
```
