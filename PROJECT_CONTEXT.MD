# Project Context: [Project Name]

> **Note to AI Agents:** This document is the single source of truth for architecture, technology stack, and code constraints for this project. Every specification, planning, and implementation MUST align with the rules defined here.

---

## 1. Project Overview

* **Description:** [Explain in 1-2 sentences what the system does. Ex: An ERP system focused on inventory management for small stores.]

* **Primary Objective:** [What main pain point does it solve? Ex: Eliminate manual spreadsheets and automate out-of-stock alerts.]

* **Target Audience:** [Who will use it? Ex: Store managers with limited technical knowledge.]

* **Key Features:**
  - [Feature 1]
  - [Feature 2]
  - [Feature 3]

---

## 2. Technology Stack

### Frontend
* **Framework:** [Ex: React 18, Next.js 14 with App Router]
* **State Management:** [Ex: Zustand, Redux, Context API, TanStack Query]
* **Styling:** [Ex: TailwindCSS, CSS Modules, Styled Components]
* **UI Components:** [Ex: Shadcn/ui, Material-UI, Ant Design]
* **Build Tool:** [Ex: Vite, Webpack, esbuild]
* **Testing:** [Ex: Jest, Vitest, Playwright for E2E]
* **Package Manager:** [Ex: npm, yarn, pnpm]

### Backend
* **Language & Runtime:** [Ex: Python 3.12, Node.js 20, Go 1.21]
* **Framework:** [Ex: FastAPI, Express, Django, Gin]
* **API Style:** [Ex: REST, GraphQL, gRPC]
* **Validation:** [Ex: Pydantic, Joi, zod]
* **ORM/Query:** [Ex: SQLAlchemy, Prisma, sqlc]
* **Testing:** [Ex: Pytest, Jest, Go testing]
* **Package Manager:** [Ex: pip, npm, go mod]

### Database
* **Primary DB:** [Ex: PostgreSQL 15, MongoDB 6, MySQL 8]
* **Caching:** [Ex: Redis 7, Memcached]
* **Search:** [Ex: Elasticsearch, Meilisearch]
* **Migrations:** [Ex: Alembic, Knex, Migrate]

### Infrastructure & DevOps
* **Hosting:** [Ex: AWS, Google Cloud, Azure, Vercel]
* **Container:** [Ex: Docker, Podman]
* **Orchestration:** [Ex: Kubernetes, Docker Compose]
* **CI/CD:** [Ex: GitHub Actions, GitLab CI, Jenkins]
* **Observability:** [Ex: Datadog, New Relic, CloudWatch]

### External Services
* **Authentication:** [Ex: Auth0, Firebase, Clerk, Cognito]
* **Payments:** [Ex: Stripe, PayPal]
* **Email:** [Ex: SendGrid, AWS SES]
* **File Storage:** [Ex: AWS S3, Google Cloud Storage]
* **Analytics:** [Ex: Segment, Mixpanel, Amplitude]

---

## 3. Architecture and Design Patterns

### Overall Architecture Pattern
[Ex: Clean Architecture / Hexagonal / CQRS / Microservices]

**Justification:**
[Why this pattern was chosen for this project]

### Layer Structure

**Frontend Architecture:**
* [Component structure - feature-sliced, atomic design, etc.]
* [State management boundaries - local vs global]
* [Data fetching patterns - React Query, SWR, etc.]

**Backend Architecture:**
* **Domain Layer:** [Business logic, entities, value objects]
* **Application Layer:** [Use cases, services, DTOs]
* **Infrastructure Layer:** [Database, external APIs, caching]
* **Presentation Layer:** [Controllers, routes, HTTP handlers]

### Data Flow
```
[Diagram or description of data flow from UI to database and back]
```

### Module Organization
```
src/
├── domain/           # Business logic (independent of framework)
├── application/      # Services, use cases
├── infrastructure/   # Database, API clients
├── presentation/     # Controllers, routes, HTTP handlers
└── shared/           # Common utilities, types
```

---

## 4. Coding Standards and Conventions

### Language-Specific Standards

**JavaScript/TypeScript:**
* **Linting:** [Ex: ESLint with strict config]
* **Formatting:** [Ex: Prettier, 2 spaces]
* **Type Checking:** [Ex: TypeScript strict mode]
* **Naming:** `camelCase` for variables/functions, `PascalCase` for classes/components
* **File Structure:** [Ex: index.ts exports, barrel pattern]

**Python:**
* **Linting:** [Ex: Ruff, Black formatting]
* **Type Hints:** [Ex: Mandatory type hints on all functions]
* **Naming:** `snake_case` for functions/variables, `PascalCase` for classes
* **Code Style:** [Ex: PEP 8 + custom rules]

**Go:**
* **Linting:** [Ex: golangci-lint, gofmt]
* **Error Handling:** [Ex: Explicit error returns, no panic in production code]
* **Naming:** [Ex: MixedCase for exported, lowercase for unexported]
* **Concurrency:** [Ex: Context propagation, goroutine management with sync or errgroup]

### Universal Rules

#### Error Handling
* Never silently swallow errors (no `try/catch pass` or `_ = err`)
* Create detailed error messages that indicate what failed and why
* Return appropriate HTTP status codes (400, 401, 403, 404, 500)
* Log errors with full context (user, operation, timestamp)

#### Code Organization
* **KISS (Keep It Simple, Stupid)** - Avoid over-engineering
* **DRY (Don't Repeat Yourself)** - Extract common patterns, but don't create premature abstractions
* **SOLID Principles** - Single responsibility, Open/closed, etc.
* **No magic numbers** - Use named constants
* **Clear naming** - Variable/function names should be self-documenting

#### Comments
* Comments explain WHY, not WHAT (code shows what)
* Add comments for non-obvious logic or business rules
* Keep comments in sync with code
* No commented-out code in final commits

#### Testing Standards
* **Coverage:** Minimum 80% for new code
* **Types:** Unit, integration, and E2E tests
* **Naming:** Test names describe what they test (e.g., `should_return_404_when_user_not_found`)
* **Isolation:** Tests should not depend on each other
* **Mocking:** Mock external dependencies in unit tests

#### Security
* **Input Validation:** All user inputs must be validated
* **SQL Injection:** Always use parameterized queries
* **XSS Prevention:** Escape/encode output
* **CORS:** Configure explicitly
* **Secrets:** Never commit to git, use environment variables
* **Auth:** Implement proper authentication and authorization

#### Performance
* **Database:** Avoid N+1 queries, use indexes
* **Frontend:** Lazy load images, code split, minimize bundle
* **Caching:** Cache appropriately (avoid over-caching)
* **Monitoring:** Track key metrics (response time, error rate)

### Commit Message Convention

**Format:**
```
<type>(<scope>): <subject>

<body>

<footer>
```

**Types:** `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `chore`

**Example:**
```
feat(auth): add JWT token refresh endpoint

Implement automatic token refresh to improve user session handling.
Tokens are refreshed 5 minutes before expiry.

- Add /api/auth/refresh endpoint
- Implement token rotation
- Add refresh token to HTTP-only cookie

Closes #42
```

---

## 5. Feature-Specific Guidelines

### Frontend Features
* **Responsive Design:** Mobile-first approach, support [list breakpoints]
* **Accessibility:** WCAG 2.1 AA standards minimum
  - Color contrast 4.5:1 for text
  - Keyboard navigation
  - Screen reader support
  - Focus management
* **Performance Budget:** [Ex: LCP < 2.5s, CLS < 0.1]
* **Browser Support:** [Ex: Chrome, Firefox, Safari (last 2 versions)]

### Backend APIs
* **Response Format:**
  ```json
  {
    "data": {...},
    "error": null,
    "message": "Optional success message"
  }
  ```
* **Error Format:**
  ```json
  {
    "data": null,
    "error": "error_code",
    "message": "Human readable message"
  }
  ```
* **Rate Limiting:** [Ex: 100 requests per minute per IP]
* **Pagination:** [Ex: Cursor-based, 50 items per page]
* **Versioning:** [Ex: URL versioning /api/v1/, Header versioning]

### Database
* **Naming:** [Ex: snake_case for tables and columns]
* **Indexes:** Index foreign keys and frequently queried columns
* **Constraints:** Use NOT NULL, UNIQUE, FK constraints
* **Migrations:** Every schema change requires a migration
* **Transactions:** Use for multi-step operations affecting consistency

### Testing Approach

**Unit Tests:**
* Test business logic in isolation
* Mock all external dependencies
* Focus on edge cases and error conditions
* Table-driven tests for multiple scenarios

**Integration Tests:**
* Test API endpoints with database
* Use test database or containers
* Verify data flow end-to-end
* Test error scenarios

**E2E Tests:**
* Critical user flows only
* Happy path and error scenarios
* Test in staging environment before production

---

## 6. Development Workflow Rules

### Before Implementation
1. Read `PROJECT_CONTEXT.md` completely
2. Understand the spec and design
3. Verify no contradictions with architecture

### During Implementation
1. Follow coding standards defined above
2. Write tests alongside code (not after)
3. Run linters and formatters
4. Commit frequently with clear messages
5. Document non-obvious decisions

### Quality Gates
1. **Code Quality:** Must pass linting and formatting
2. **Tests:** Must pass all tests, 80%+ coverage
3. **Security:** Must pass security scan, no hardcoded secrets
4. **Architecture:** Must follow patterns defined above
5. **Documentation:** Updated if needed (README, comments)

---

## 7. Common Patterns and Examples

### API Endpoint Pattern
[Show example of a typical GET/POST/PUT/DELETE endpoint following your conventions]

### Service/Use Case Pattern
[Show example of a service class or use case implementation]

### Component Pattern
[Show example of a frontend component following your conventions]

### Database Query Pattern
[Show example of a query following your conventions]

### Error Handling Pattern
[Show how errors should be handled]

### Testing Pattern
[Show example test structure]

---

## 8. Project-Specific Rules

[Add any project-specific rules or constraints that don't fit above categories]

* Rule 1: [Ex: All dates must be UTC]
* Rule 2: [Ex: Feature flags required for new features]
* Rule 3: [Ex: Database backups required before deployments]

---

## 9. When in Doubt

1. **Code is read more often than written** - Optimize for clarity
2. **Simple is better than clever** - Don't over-engineer
3. **Ask the team** - Check PROJECT_CONTEXT.md, issues, or previous implementations
4. **Write tests** - Tests clarify intent and catch bugs early
5. **Follow existing patterns** - Stay consistent with codebase

---

**Last Updated:** [date]
**Maintained By:** [team/person]
