---
name: db-migrator
description: Manages database migrations safely, including creation, validation, rollback planning, and execution guidance.
---
## Database Migrator Skill

Safely manage database schema changes with proper migration practices.

### When to Use
- When backend changes require schema modifications
- During planner-backend phase
- Before executor implements database changes
- For rollback planning

### Prerequisites
**CRITICAL**: Read `PROJECT_CONTEXT.md` for:
- Database type (PostgreSQL, MySQL, MongoDB, etc.)
- Migration tool (Prisma, Knex, Alembic, GORM, etc.)
- Migration naming conventions
- Environment handling (dev, staging, prod)

Use MCP `db-query` to inspect current schema.

### Migration Workflow

```
┌─────────────────────────────────────────────────────┐
│  1. Analyze Required Changes                         │
│     - Review spec for data model changes            │
│     - Compare with current schema                    │
└───────────────────────────┬─────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────┐
│  2. Generate Migration                               │
│     - Create up migration                            │
│     - Create down migration (rollback)              │
│     - Add data migration if needed                   │
└───────────────────────────┬─────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────┐
│  3. Validate Migration                               │
│     - Check syntax                                   │
│     - Verify rollback works                          │
│     - Test with sample data                          │
└───────────────────────────┬─────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────┐
│  4. Document Migration                               │
│     - Add to migration plan                          │
│     - Document rollback procedure                    │
│     - Note data backup requirements                  │
└─────────────────────────────────────────────────────┘
```

### Step 1: Analyze Schema Changes

Use MCP `db-query` to get current schema:
```sql
-- PostgreSQL
SELECT table_name, column_name, data_type, is_nullable
FROM information_schema.columns
WHERE table_schema = 'public'
ORDER BY table_name, ordinal_position;

-- Get indexes
SELECT indexname, indexdef
FROM pg_indexes
WHERE schemaname = 'public';

-- Get constraints
SELECT conname, contype, conrelid::regclass
FROM pg_constraint;
```

### Step 2: Generate Migration

#### Prisma Example
```bash
# Generate migration
npx prisma migrate dev --name add_user_refresh_token

# Migration file: prisma/migrations/20240315_add_user_refresh_token/
```

#### Knex Example
```bash
# Create migration
npx knex migrate:make add_user_refresh_token
```

```javascript
// migrations/20240315_add_user_refresh_token.js
exports.up = function(knex) {
  return knex.schema.alterTable('users', (table) => {
    table.string('refresh_token', 512).nullable();
    table.timestamp('token_expires_at').nullable();
    table.index('refresh_token');
  });
};

exports.down = function(knex) {
  return knex.schema.alterTable('users', (table) => {
    table.dropIndex('refresh_token');
    table.dropColumn('token_expires_at');
    table.dropColumn('refresh_token');
  });
};
```

#### Alembic Example (Python)
```bash
alembic revision --autogenerate -m "add_user_refresh_token"
```

```python
# alembic/versions/20240315_add_user_refresh_token.py
def upgrade():
    op.add_column('users', sa.Column('refresh_token', sa.String(512)))
    op.add_column('users', sa.Column('token_expires_at', sa.DateTime()))
    op.create_index('ix_users_refresh_token', 'users', ['refresh_token'])

def downgrade():
    op.drop_index('ix_users_refresh_token')
    op.drop_column('users', 'token_expires_at')
    op.drop_column('users', 'refresh_token')
```

### Step 3: Migration Safety Rules

#### MUST DO
- Always write rollback (down) migration
- Test migration on copy of production data
- Back up database before production migration
- Use transactions for multi-step migrations
- Add indexes for new foreign keys

#### MUST NOT DO
- Drop columns without data backup
- Rename columns without application coordination
- Add NOT NULL without default value
- Run migrations during peak traffic
- Skip staging environment testing

### Step 4: Data Migration Patterns

#### Adding Required Column
```sql
-- Step 1: Add nullable column
ALTER TABLE users ADD COLUMN status VARCHAR(20);

-- Step 2: Backfill data
UPDATE users SET status = 'active' WHERE status IS NULL;

-- Step 3: Make NOT NULL
ALTER TABLE users ALTER COLUMN status SET NOT NULL;
```

#### Renaming Column (Zero Downtime)
```sql
-- Step 1: Add new column
ALTER TABLE users ADD COLUMN full_name VARCHAR(255);

-- Step 2: Backfill
UPDATE users SET full_name = name;

-- Step 3: Update application to use both columns

-- Step 4: (Later migration) Drop old column
ALTER TABLE users DROP COLUMN name;
```

### Step 5: Migration Plan Document

Create in `agents/tasks/migration-plan-<issue-num>.md`:

```markdown
# Migration Plan: Issue #<num>

## Overview
<what schema changes are needed and why>

## Changes

### Table: users
| Action | Column | Type | Notes |
|--------|--------|------|-------|
| ADD | refresh_token | VARCHAR(512) | Nullable, indexed |
| ADD | token_expires_at | TIMESTAMP | Nullable |

### Indexes
| Action | Name | Columns |
|--------|------|---------|
| CREATE | ix_users_refresh_token | refresh_token |

## Migration Files
- `migrations/20240315_add_user_refresh_token.js`

## Execution Plan

### Pre-Migration
- [ ] Backup database
- [ ] Verify disk space
- [ ] Schedule maintenance window (if needed)

### Migration
```bash
# Development
npm run migrate:dev

# Staging
npm run migrate:staging

# Production
npm run migrate:prod
```

### Rollback Plan
```bash
# If issues occur
npm run migrate:rollback

# Verify rollback
npm run db:status
```

### Post-Migration
- [ ] Verify schema matches expected
- [ ] Run smoke tests
- [ ] Monitor for errors

## Estimated Duration
- Migration: ~2 minutes
- Rollback: ~1 minute

## Risk Assessment
- **Risk:** Low
- **Data Loss Potential:** None (additive only)
- **Downtime Required:** No
```

### Step 6: Validation

```bash
# Check migration syntax
npm run migrate:validate

# Dry run (if supported)
npm run migrate:dry-run

# Test rollback
npm run migrate:up && npm run migrate:down && npm run migrate:up
```

### Output Format

```
## Migration Plan Ready

**Issue:** #42
**Migration:** add_user_refresh_token

### Schema Changes
- users.refresh_token (ADD, VARCHAR(512))
- users.token_expires_at (ADD, TIMESTAMP)
- ix_users_refresh_token (CREATE INDEX)

### Files Created
- migrations/20240315_add_user_refresh_token.js

### Risk Level: Low
- Additive changes only
- No data modification
- Rollback tested

### Plan Document
- agents/tasks/migration-plan-42.md

Ready for: @executor to implement application changes
```

### Integration
- Used by: `planner-backend`
- Reports to: `todo-manager`
- Uses: MCP `db-query`
