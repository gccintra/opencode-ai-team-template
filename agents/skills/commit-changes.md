---
name: commit-changes
description: Creates standardized conventional commits following project conventions and best practices.
---
## Commit Changes Skill

Create well-structured, conventional commits that follow project standards and provide clear history.

### When to Use
- After code review approval (spec status = READY_TO_COMMIT)
- When committer agent is invoked
- Never automatically - always requires explicit trigger

### Prerequisites
**CRITICAL**: Read `PROJECT_CONTEXT.md` for:
- Commit message conventions
- Branching strategy
- Required sign-offs or co-authors

### Step 1: Verify Ready State
```bash
# Check spec status
grep -l "READY_TO_COMMIT" agents/specs/issue-*-spec.md

# Verify clean working directory (only expected changes)
git status

# Review what will be committed
git diff --staged
git diff
```

### Step 2: Stage Files
Stage only relevant files - NEVER stage sensitive files:

```bash
# Stage specific files (preferred)
git add src/features/newFeature.ts
git add src/__tests__/newFeature.test.ts

# Or stage by pattern
git add "src/**/*.ts"

# NEVER do this blindly
# git add -A  # Dangerous - may include secrets
# git add .   # Dangerous - may include secrets
```

**Files to NEVER commit:**
- `.env`, `.env.*`
- `*.pem`, `*.key`, `*.crt`
- `credentials.json`, `secrets.yaml`
- `node_modules/`, `__pycache__/`
- `.DS_Store`, `Thumbs.db`

### Step 3: Craft Commit Message

#### Conventional Commit Format
```
<type>(<scope>): <subject>

<body>

<footer>
```

#### Types
| Type | Description |
|------|-------------|
| `feat` | New feature |
| `fix` | Bug fix |
| `docs` | Documentation only |
| `style` | Formatting, no code change |
| `refactor` | Code change, no feature/fix |
| `perf` | Performance improvement |
| `test` | Adding/updating tests |
| `chore` | Maintenance, dependencies |
| `ci` | CI/CD changes |

#### Scope
Optional, indicates affected area:
- `auth`, `api`, `ui`, `db`, `config`

#### Subject Rules
- Imperative mood ("add" not "added")
- No period at end
- Max 50 characters
- Lowercase

#### Body Rules
- Explain WHAT and WHY (not how)
- Wrap at 72 characters
- Separate from subject with blank line

#### Footer
- Reference issues: `Closes #123`
- Breaking changes: `BREAKING CHANGE: description`
- Co-authors: `Co-authored-by: Name <email>`

### Step 4: Create Commit

```bash
git commit -m "feat(auth): add JWT token refresh endpoint

Implement automatic token refresh to improve user session handling.
Tokens are refreshed 5 minutes before expiry to prevent interruptions.

- Add /api/auth/refresh endpoint
- Implement token rotation for security
- Add refresh token to HTTP-only cookie

Closes #42

Co-authored-by: AI Assistant <ai@example.com>"
```

### Step 5: Verify Commit

```bash
# Check commit was created
git log -1 --oneline

# Verify commit content
git show HEAD --stat
```

### Commit Message Templates

#### Feature
```
feat(<scope>): <what was added>

<Why this feature is needed>
<How it works at high level>

- Bullet point of change 1
- Bullet point of change 2

Closes #<issue>
```

#### Bug Fix
```
fix(<scope>): <what was fixed>

<What was broken>
<Root cause>
<How it was fixed>

Closes #<issue>
```

#### Refactor
```
refactor(<scope>): <what was refactored>

<Why refactoring was needed>
<What approach was taken>

No functional changes.

Related to #<issue>
```

### Pre-commit Checks
Before committing, verify:
- [ ] All tests pass
- [ ] Linting passes
- [ ] No console.log/print statements
- [ ] No commented-out code
- [ ] No TODO without issue reference
- [ ] No sensitive data in diff

### Output Format
```
## Commit Created

**Hash:** a1b2c3d
**Type:** feat
**Scope:** auth
**Message:** add JWT token refresh endpoint

**Files committed:**
- src/features/auth/refresh.ts (new)
- src/features/auth/tokenService.ts (modified)
- src/__tests__/auth/refresh.test.ts (new)

**Issue:** Closes #42

Ready for: push-changes skill
```

### Error Handling

**If pre-commit hook fails:**
1. Read hook output
2. Fix the issue
3. Stage fixes
4. Try commit again
5. DO NOT use `--no-verify`

**If commit is too large:**
1. Consider splitting into logical commits
2. Each commit should be atomic (one logical change)
