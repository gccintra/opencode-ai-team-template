# Project Context — [Project Name]

> **Last Updated:** [date] | **Maintained By:** [team/person]
> **Architecture:** [Clean Architecture / Hexagonal / Layered / Microservices / Modular Monolith]

---

## 1. Project Overview

[2-3 sentences: what it does, who it's for, core value proposition]

**Primary Objective:** [What main pain point does it solve?]

**Target Audience:** [Who will use it?]

**Key Features:**
- [Feature 1]
- [Feature 2]
- [Feature 3]

---

## 2. Technology Stack — Dev Commands

### Stack

| Layer | Technology |
|-------|-----------|
| Frontend | [React 19 / Vue 3 / Next.js 14 / Not applicable] |
| Build Tool | [Vite / Webpack / esbuild] |
| CSS | [TailwindCSS / CSS Modules / Styled Components] |
| State Management | [Zustand / Redux / React Query / Pinia] |
| Backend | [Go 1.24 / Python 3.13 / Node.js 22 / Not applicable] |
| API Style | [REST / GraphQL / gRPC] |
| Database | [PostgreSQL 16 / MySQL 8 / MongoDB 7] |
| ORM | [Prisma / GORM / SQLAlchemy / sqlc] |
| Cache | [Redis / None] |
| Auth | [JWT / OAuth2 / Session / API Keys / None] |
| Package Manager | [npm / yarn / pnpm / pip / go modules] |
| Linter | [ESLint / Ruff / golangci-lint] |
| Formatter | [Prettier / Black / gofmt] |

### Dev Commands

**Backend:**

| Command | Description |
|---------|------------|
| `[dev-server-cmd]` | Start dev server |
| `[test-cmd]` | Run unit + integration tests |
| `[coverage-cmd]` | Run tests with coverage |
| `[lint-cmd]` | Lint code |
| `[typecheck-cmd]` | Type-check (or N/A) |
| `[build-cmd]` | Build for production |
| `[security-cmd]` | Security scan (or N/A) |

**Frontend (if applicable):**

| Command | Description |
|---------|------------|
| `[dev-server-cmd]` | Start dev server |
| `[test-cmd]` | Run unit tests |
| `[e2e-cmd]` | Run E2E tests (or N/A) |
| `[coverage-cmd]` | Run tests with coverage |
| `[lint-cmd]` | Lint code |
| `[typecheck-cmd]` | Type-check |
| `[build-cmd]` | Build for production |

**Test DB Management:**

| Command | Description |
|---------|------------|
| `[db-reset-cmd]` | Reset test database (or N/A) |
| `[migrate-cmd]` | Run database migrations (or N/A) |

### DB Access (Docker)

- **DB Container:** [container-name]
- **DB User:** [user]
- **DB Name:** [db-name]
- **Access Command:** `docker exec -i <container> psql -U <user> -d <db> -c "<query>"`

---

## 3. Architecture

**Pattern:** [Clean Architecture / Hexagonal / Layered (MVC) / Microservices / Modular Monolith]

**Justification:** [Why this pattern was chosen]

### Layer Structure

**Backend:**
- **Domain Layer:** [Business logic, entities, value objects]
- **Application Layer:** [Use cases, services, DTOs]
- **Infrastructure Layer:** [Database, external APIs, caching]
- **Presentation Layer:** [Controllers, routes, HTTP handlers]

**Frontend (if applicable):**
- [Component structure — feature-sliced, atomic design, etc.]
- [State management boundaries — local vs global]
- [Data fetching patterns]

### Module Organization

```
src/
├── domain/           # Business logic (independent of framework)
├── application/      # Services, use cases
├── infrastructure/   # Database, API clients
├── presentation/     # Controllers, routes, HTTP handlers
└── shared/           # Common utilities, types
```

### API Conventions

**Success Response:**
```json
{
  "data": { ... },
  "error": null
}
```

**Error Response:**
```json
{
  "data": null,
  "error": "error_code",
  "message": "Human readable description"
}
```

**Conventions:**
- Rate Limiting: [100 req/min per IP or N/A]
- Pagination: [cursor-based / offset, 50 per page]
- Versioning: [URL /api/v1/ or Header]

---

## 4. Data Model

**Core Entities:**

| Entity | Key Fields | Relationships |
|--------|-----------|---------------|
| [Entity 1] | [field1, field2, field3] | [belongs to X, has many Y] |
| [Entity 2] | [field1, field2, field3] | [belongs to X] |
| [Entity 3] | [field1, field2, field3] | [has many Y] |

**Naming Conventions:**
- Tables: `snake_case` plural
- Columns: `snake_case`
- Foreign Keys: `<table>_id`
- Timestamps: `created_at`, `updated_at`

**Database Rules:**
- All tables have `id` as primary key
- Index foreign keys and frequently queried columns
- Every schema change requires a migration
- Use transactions for multi-step operations

---

## 5. Coding Standards & Conventions

### Language-Specific

**JavaScript/TypeScript:**
- Linting: [ESLint + strict config]
- Formatting: [Prettier, 2 spaces]
- Type Checking: [TypeScript strict mode]
- Naming: `camelCase` variables/functions, `PascalCase` classes/components
- Imports: [standard / alphabetical / grouped by type]

**Python:**
- Linting: [Ruff]
- Formatting: [Black]
- Type Hints: [mandatory on all functions]
- Naming: `snake_case` functions/variables, `PascalCase` classes

**Go:**
- Linting: [golangci-lint]
- Formatting: [gofmt]
- Error Handling: [explicit returns, no panic in production]
- Naming: MixedCase exported, lowercase unexported

### Universal Rules

**Error Handling:**
- Never silently swallow errors
- Return appropriate HTTP status codes (400, 401, 403, 404, 500)
- Log errors with context (user, operation, timestamp)

**Code Organization:**
- KISS — avoid over-engineering
- DRY — extract patterns, don't create premature abstractions
- No magic numbers — use named constants
- Clear, self-documenting names

**Comments:**
- Explain WHY, not WHAT
- No commented-out code in commits

**Security:**
- Input validation on all user inputs
- Parameterized queries (no SQL injection)
- Output encoding (XSS prevention)
- Never commit secrets — use environment variables
- CORS configured explicitly

**Performance:**
- Avoid N+1 queries — use indexes
- Lazy load images, code split frontend
- Cache appropriately

### Commit Convention

```
<type>(<scope>): <subject>

<body>

<footer>
```

Types: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `chore`

**Branch Naming:** `<type>/<id>-<short-desc>`

---

## 6. Testing Strategy

| Aspect | Value |
|--------|-------|
| Framework | [Jest / Vitest / PyTest / Go Test] |
| E2E Tool | [Playwright / Cypress / N/A] |
| Coverage Threshold | [80%] |
| Test File Convention | `*.test.ts` / `test_*.py` / `*_test.go` |
| Test Location | [next to source / `__tests__/` / separate `tests/` dir] |
| Mock Strategy | [mock at service boundaries / use test doubles] |
| Pre-commit/PR | [tests must pass before merge? yes/no] |

### Test Types

**Unit Tests:**
- Test business logic in isolation
- Mock all external dependencies
- Focus on edge cases and error conditions

**Integration Tests:**
- Test API endpoints with database
- Use test database or containers
- Verify data flow end-to-end

**E2E Tests (if applicable):**
- Critical user flows only
- Happy path and error scenarios

---

## 7. Authentication & Security

| Aspect | Value |
|--------|-------|
| Auth Method | [JWT / OAuth2 / Session / API Keys / None] |
| Token Storage | [HTTP-only cookie / localStorage / Authorization header] |
| Token Expiry | [15 min access / 7 day refresh] |
| Security Scanner | [gosec / bandit / npm audit / N/A] |
| Secrets Management | [.env file / HashiCorp Vault / cloud secrets manager] |
| CORS | [configured / not applicable] |
| Rate Limiting | [100 req/min per IP / N/A] |

---

## 8. Styling & Design (UI Projects)

| Aspect | Value |
|--------|-------|
| Figma File Key | [key — extracted from `figma.com/design/:key/...`] |
| Figma File URL | [full URL or N/A] |
| Primary Font | [Inter / Geist / DM Sans / System default] |
| Color Palette | [e.g., `--primary: #111`, `--accent: #6366f1`] |
| Icon Library | [Lucide / Heroicons / Phosphor / from Figma] |
| Component Library | [Shadcn/ui / Radix / custom only] |
| Design Tokens | [CSS custom properties / Tailwind config / separate file] |

**Accessibility:** WCAG 2.1 AA minimum — color contrast 4.5:1, keyboard navigation, screen reader support, focus management.

**Browser Support:** [Chrome, Firefox, Safari — last 2 versions]

---

## 9. External Dependencies & Integrations

| Service | Purpose | Auth / Config |
|---------|---------|---------------|
| [Service 1] | [What it does] | [API key / OAuth / env var] |
| [Service 2] | [What it does] | [API key / OAuth / env var] |

Use `N/A` if no external integrations.

---

## 10. Common Pitfalls & Lessons Learned

> _This section is filled automatically by the `lessons-writer` skill as agents work on the project._
> _Patterns, bugs, performance insights, and security considerations are appended here during development._

### Architecture Patterns

<!-- Add patterns discovered during development -->

### Common Pitfalls

<!-- Add common mistakes to avoid -->

### Testing Insights

<!-- Add testing discoveries and strategies -->

### Performance Notes

<!-- Add performance optimizations learned -->

### Security Considerations

<!-- Add security patterns and vulnerabilities to watch for -->

### Tool-Specific Tips

<!-- Add tips for specific tools/libraries -->

---

*Created by @project-setup on [date]*
