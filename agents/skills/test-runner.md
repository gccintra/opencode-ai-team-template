---
name: test-runner
description: Executes tests using project testing tools, captures results, and reports pass/fail status with details.
---
## Test Runner Skill

Execute all tests using the project's testing framework and capture results for reporting.

### When to Use
- After executor completes implementation
- After test-generator creates new tests
- Before code review
- As part of CI verification

### Prerequisites
**CRITICAL**: Read `PROJECT_CONTEXT.md` to understand:
- Testing frameworks and commands
- Required environment variables
- Test database setup (if applicable)
- Coverage thresholds

### Step 1: Environment Check
Verify test environment is ready:
```bash
# Check for test dependencies
npm list --depth=0 | grep -E "(jest|vitest|playwright)"
# or
pip list | grep pytest

# Check for test configuration
ls -la jest.config.* vitest.config.* pytest.ini setup.cfg pyproject.toml
```

### Step 2: Run Unit Tests
Use MCP `testing-tools` for execution:

**JavaScript/TypeScript:**
```bash
npm run test -- --coverage --reporter=verbose
# or
npx vitest run --coverage
```

**Python:**
```bash
pytest -v --cov=src --cov-report=term-missing
```

**Go:**
```bash
go test -v -race -cover ./...
```

### Step 3: Run Integration Tests
```bash
# If separate from unit tests
npm run test:integration
# or
pytest tests/integration/ -v
```

### Step 4: Run E2E Tests
```bash
# Playwright
npx playwright test --reporter=list

# Cypress
npx cypress run
```

### Step 5: Capture Results

#### Parse Test Output
Extract from test output:
- Total tests run
- Tests passed
- Tests failed
- Tests skipped
- Execution time
- Coverage percentage

#### Result Structure
```json
{
  "summary": {
    "total": 45,
    "passed": 43,
    "failed": 2,
    "skipped": 0,
    "duration": "12.5s"
  },
  "coverage": {
    "statements": 85.2,
    "branches": 78.4,
    "functions": 90.1,
    "lines": 84.8
  },
  "failures": [
    {
      "test": "UserService.login should reject invalid credentials",
      "file": "src/__tests__/userService.test.ts:45",
      "error": "Expected throw but got undefined",
      "stack": "..."
    }
  ]
}
```

### Step 6: Report Results

#### All Tests Pass
```markdown
## Test Results: PASS ✓

**Summary:**
- Total: 45 tests
- Passed: 45
- Failed: 0
- Duration: 12.5s

**Coverage:**
- Statements: 85.2%
- Branches: 78.4%
- Functions: 90.1%
- Lines: 84.8%

**Status:** Ready for code review
```

#### Tests Failed
```markdown
## Test Results: FAIL ✗

**Summary:**
- Total: 45 tests
- Passed: 43
- Failed: 2
- Duration: 12.5s

**Failed Tests:**

### 1. UserService.login should reject invalid credentials
**File:** `src/__tests__/userService.test.ts:45`
**Error:** Expected throw but got undefined
**Probable Cause:** Missing validation in login function

### 2. API.createUser should return 201 on success
**File:** `src/__tests__/api.test.ts:112`
**Error:** Expected 201 but received 500
**Probable Cause:** Database connection issue or missing migration

**Action Required:** Return to @executor for fixes
```

### Step 7: Handoff Decision

**If ALL tests pass AND coverage >= threshold:**
→ Handoff to `@reviewer`

**If ANY tests fail:**
→ Return to `@executor` with failure report

**If coverage below threshold:**
→ Return to `@executor` with coverage report

### Coverage Threshold
Default: 80% (or as specified in PROJECT_CONTEXT.md)

Check against threshold:
```bash
# Extract coverage from report
# If below threshold, fail the gate
```

### Test Isolation
Before running tests, ensure:
```bash
# Reset test database if applicable
npm run db:test:reset

# Clear test caches
npm run test:clear-cache
# or
pytest --cache-clear
```

### Output Format
```
## Test Execution Report

**Issue:** #<num>
**Timestamp:** <ISO timestamp>
**Duration:** <total time>

### Results
| Type | Passed | Failed | Skipped |
|------|--------|--------|---------|
| Unit | 40 | 0 | 2 |
| Integration | 5 | 0 | 0 |
| E2E | 3 | 0 | 0 |
| **Total** | **48** | **0** | **2** |

### Coverage
- New code: 87%
- Overall: 82%
- Threshold: 80% ✓

### Verdict: PASS
Ready for: @reviewer
```

### Integration with Other Skills
- Receives tests from: `test-generator`
- Reports to: `test-logger`
- On fail: Returns to `@executor`
- On pass: Handoffs to `@reviewer`
