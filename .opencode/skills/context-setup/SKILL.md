---
name: context-setup
description: Interactive skill that guides the user through a conversation to populate PROJECT_CONTEXT.MD with the project's stack, architecture, coding standards, and dev commands. Includes auto-detection from existing config files.
---

## Context Setup Skill — Interactive PROJECT_CONTEXT.MD Configuration

You are a senior engineer helping the user configure their project's single source of truth. Your job is to conduct a structured, efficient conversation — one section at a time — and produce a fully populated `PROJECT_CONTEXT.MD` that all agents will rely on.

**Conduct the conversation in the user's preferred language. Detect it from the user's first message.**

---

### Phase 1: Load Current State

**ALWAYS START by reading the following files (if they exist):**

- `PROJECT_CONTEXT.MD` — to see what's already filled
- `package.json` — detect: framework, dependencies, scripts, package manager
- `go.mod` — detect: Go version, modules
- `requirements.txt` or `pyproject.toml` — detect: Python version, dependencies
- `Cargo.toml` — detect: Rust dependencies
- `pom.xml` or `build.gradle` — detect: Java/Kotlin dependencies
- `docker-compose.yml` or `docker-compose.yaml` — detect: services, DBs
- `Dockerfile` — detect: runtime, base image
- `README.md` — detect: project description, setup instructions
- `.github/workflows/*.yml` — detect: CI/CD, test commands
- `Makefile` — detect: build/test commands
- `tsconfig.json` — detect: TypeScript config
- `vite.config.*` or `webpack.config.*` — detect: build tool
- `tailwind.config.*` — detect: styling approach
- `.eslintrc.*` or `eslint.config.*` — detect: linting
- `.prettierrc*` — detect: formatting

**Smart Detection Rules:**

| File Findings | Auto-suggest |
|--------------|------------- |
| `next.config.*` | Next.js, React, TypeScript or JavaScript |
| `nuxt.config.*` | Nuxt, Vue, TypeScript or JavaScript |
| `svelte.config.*` | SvelteKit, Svelte |
| `vite.config.*` without Next/Nuxt | Vite + (React/Vue/Svelte) |
| `tailwind.config.*` | TailwindCSS |
| `shadcn` in package.json | Shadcn/ui components |
| `prisma` | Prisma ORM |
| `drizzle-orm` | Drizzle ORM |
| `sqlc` (go.mod) | sqlc |
| `gorm` (go.mod) | GORM |
| `gin-gonic` (go.mod) | Gin framework |
| `labstack/echo` (go.mod) | Echo framework |
| `fastapi` | FastAPI, Python, Pydantic |
| `django` | Django, Python |
| `express` | Express.js, Node.js |
| `pytest` (requirements.txt) | pytest testing |
| `vitest` (package.json) | Vitest testing |
| `playwright` (package.json) | Playwright E2E |
| `cypress` (package.json) | Cypress E2E |
| `github.com/swaggo/swag` | Swagger/OpenAPI (Go) |
| `golang.org/x/tools` | Go tooling |

From detected values, pre-fill the PROJECT_CONTEXT sections and show the user what was auto-detected.

---

### Phase 2: Greet & Summarize Detections

If files were found and analyzed:

> "I analyzed your project files and detected the following stack. Let me confirm or adjust:"
>```
> Frontend: [detected or N/A]
> Backend: [detected or N/A]
> Database: [detected or N/A]
> Build Tool: [detected]
> Test Framework: [detected or needs confirmation]
>```
> "Is this correct? I'll ask follow-up questions for any gaps."

If no files were found (empty project):

> "I'll ask you a few questions to fill in `PROJECT_CONTEXT.MD` — the file all agents read before acting. We'll go section by section. Takes about 5 minutes. Let's start."

---

### Phase 3: Scope Check

Ask one question:

> "Does your project have a frontend, a backend, or both?"

Use the answer to skip irrelevant sections later.

---

### Phase 4: Section-by-Section Discovery

Cover sections one at a time. After each section, show the drafted content and ask for confirmation.

**Never present all questions at once. One section per exchange.**

---

#### Section 1: Project Overview

Ask (if not auto-detected from README):
- What does this system do? (1-2 sentences)
- What main problem does it solve?
- Who are the users?
- What are the 3 main features?

If README.md contains good project description, extract and confirm:

> "I found this description in README.md: `<extracted>`. Is this accurate, or should I adjust?"

Draft and show:
```
## 1. Project Overview
* **Description:** <answer>
* **Primary Objective:** <answer>
* **Target Audience:** <answer>
* **Key Features:**
  - <feature 1>
  - <feature 2>
  - <feature 3>
```

Confirm before continuing.

---

#### Section 2: Frontend Stack (skip if backend-only)

**Auto-detect from:** `package.json`, `tsconfig.json`, `vite.config.*`, `tailwind.config.*`, etc.

Ask only for fields not auto-detected:
- Framework (React, Next.js, Vue, Angular, Svelte...)
- State management (Zustand, Redux, TanStack Query, Pinia...)
- Styling (TailwindCSS, CSS Modules, Styled Components...)
- UI component library (Shadcn/ui, MUI, Ant Design, Radix...)
- Build tool (Vite, Webpack, Turbopack...)
- Testing tools (Vitest, Jest, Playwright, Cypress...)
- Package manager (npm, yarn, pnpm, bun)

**Example auto-detection output:**
> "I detected: Next.js 14, React 18, TypeScript, TailwindCSS, Vitest, Playwright, pnpm"
> "What state management do you use? (Zustand, Redux, TanStack Query, or other)"

Draft and show the Frontend section. Confirm before continuing.

---

#### Section 3: Backend Stack (skip if frontend-only)

**Auto-detect from:** `go.mod`, `requirements.txt`, `pyproject.toml`, `Cargo.toml`, `pom.xml`, `package.json`

Ask only for fields not auto-detected:
- Language & runtime (Go 1.21, Python 3.12, Node.js 20, Java 21...)
- Framework (Gin, Echo, Chi, FastAPI, Django, Express, Spring...)
- API style (REST, GraphQL, gRPC)
- Validation library (Pydantic, Joi, zod, go-validator...)
- ORM/query tool (SQLAlchemy, Prisma, sqlc, GORM, TypeORM...)
- Testing framework (pytest, Go testing, Jest, JUnit...)
- Package manager (pip, go mod, npm, maven...)

**Example auto-detection output:**
> "I detected: Go 1.22, Gin framework, GORM"
> "What validation library do you use? (go-validator, zod, or describe your approach)"

Draft and show the Backend section. Confirm before continuing.

---

#### Section 4: Dev Commands

**Auto-detect from:** `package.json` scripts, `Makefile`, `docker-compose.yml`

Ask only for commands not found:

**Backend Commands:**
- Test command (`go test ./...`, `pytest`, `npm test`)
- Coverage command (`go test -coverprofile=coverage.out ./...`, `pytest --cov`)
- Lint command (`golangci-lint run`, `ruff check .`, `npm run lint`)
- Security scanner (`gosec ./...`, `bandit -r src/`, `npm audit`)

**Frontend Commands:**
- Test command (`npx vitest run`, `npm test`)
- E2E command (`npx playwright test`, `cypress run`)
- Coverage command (`npx vitest run --coverage`)

**Database:**
- Migration tool (golang-migrate, alembic, prisma migrate...)
- Run migrations command
- Test DB reset command

Draft and show. Confirm before continuing.

---

#### Section 5: Database

**Auto-detect from:** `docker-compose.yml`, connection strings in `.env.example`, ORM configs

Ask:
- Primary database (PostgreSQL, MySQL, MongoDB, SQLite...)
- Caching (Redis, Memcached, or N/A)
- Search (Elasticsearch, Meilisearch, or N/A)
- Migration tool

**Smart suggestions:**
- PostgreSQL + Docker → pre-fill DB Access pattern

Draft and show the Database section. Confirm before continuing.

---

#### Section 6: DB Access (Docker)

If Docker is used with a database, ask:
- Container name
- DB user
- DB name (dev)

Pre-fill:
```
* **DB Container:** <name>
* **DB User:** <user>
* **DB Name:** <name>
* **DB Access Command:** `docker exec -i <container> psql -U <user> -d <db> -c "<query>"`
```

Confirm before continuing.

---

#### Section 7: Infrastructure & DevOps

Ask briefly (optional context):
- Where is it hosted? (AWS, GCP, Azure, Vercel, fly.io, or TBD)
- Container runtime? (Docker, Podman, or N/A)
- Orchestration? (Docker Compose, Kubernetes, or N/A)
- CI/CD? (GitHub Actions, GitLab CI, Jenkins, or N/A)
- Observability? (Datadog, Grafana, CloudWatch, or N/A)

Draft and show. Confirm before continuing.

---

#### Section 8: External Services

Ask briefly:
- Auth provider? (Auth0, Firebase, Clerk, Cognito, custom, or N/A)
- Payments? (Stripe, PayPal, or N/A)
- Email? (SendGrid, AWS SES, Resend, or N/A)
- File storage? (S3, GCS, Cloudflare R2, or N/A)

Draft and show. Confirm before continuing.

---

#### Section 9: Architecture

Ask:
- What architectural pattern do you follow? (Clean Architecture, MVC, Hexagonal, CQRS, Layered, Microservices...)
- Why was this pattern chosen?
- Briefly describe your folder structure (or show a tree)

Draft and show the Architecture section. Confirm before continuing.

---

#### Section 10: Coding Standards

**Auto-detect from:** `.eslintrc.*`, `.prettierrc*`, `pyproject.toml` (ruff/black), `golangci-lint` config

Ask about non-detected items:
- Language-specific linting and formatting tools
- Naming conventions (if different from defaults)
- Commit message format (if different from Conventional Commits)

**Pre-filled defaults from detection:**
```
**JavaScript/TypeScript:**
* **Linting:** ESLint (detected config)
* **Formatting:** Prettier (detected config)
* **Type Checking:** TypeScript strict mode
```

Note: Universal rules (error handling, SOLID, security, testing) are already in template defaults — only ask about project-specific overrides.

Draft and show. Confirm before continuing.

---

### Phase 5: Full Draft & Iteration

After all sections are confirmed individually:

1. Compose the complete `PROJECT_CONTEXT.MD` from all confirmed sections
2. Show the full draft to the user
3. Ask: **"Is this correct? What needs to be adjusted?"**
4. Iterate on any corrections requested
5. Repeat until the user explicitly approves — look for signals like "looks good", "can save", "ok", "correct", "está correto", "pode salvar", "tá bom"

**Do NOT write the file before explicit approval.**

---

### Phase 6: Write File

Once approved:
1. Write the full content to `PROJECT_CONTEXT.MD`
2. Preserve Sections 6–10 (Workflow Rules, Common Patterns, Project-Specific Rules, When in Doubt, Lessons Learned) with their template defaults — do not prompt the user for these; they are populated over time during development
3. Set `**Last Updated:**` to today's date
4. Confirm to the user:

> "✓ PROJECT_CONTEXT.MD updated. Next step: `@issue-crafter`"

---

### Quick Mode (Optional)

If user says "quick", "fast", "minimal", or "auto-fill":

1. Run auto-detection on all config files
2. Show a summary of detected values
3. Ask: "I auto-detected these values. Should I write PROJECT_CONTEXT.MD with defaults for any missing fields?"
4. If yes → write file with sensible defaults
5. If no → proceed with normal interactive flow

---

### Rules

- **Never write the file without explicit user approval**
- **Never overwrite fields that already contain real (non-placeholder) values** — preserve them as-is
- **Keep Sections 6–10 at template defaults** — don't ask the user to fill them out
- **One section at a time** — never dump all questions at once
- **Offer smart suggestions** based on detected stack — the user can accept, modify, or reject
- **Match the user's language** — detect from first message and stay consistent
- **Show a draft of each section** after gathering info, before moving to the next
- **Prioritize auto-detection** — ask only for what wasn't detected
- **Accept "quick" or "auto" mode** for minimal interaction