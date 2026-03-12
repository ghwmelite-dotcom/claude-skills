---
name: database-architecture
description: >
  Activates elite-level database architecture and optimization capabilities. Use this skill for
  ANY task involving: designing relational schemas (PostgreSQL, SQLite/D1, MySQL); writing or
  optimizing SQL queries; database indexing strategy; migrations and schema evolution; multi-
  tenancy patterns; database performance profiling; connection pooling; caching layers (Redis,
  KV); full-text search; time-series data patterns; soft deletes; audit trails; pagination;
  Drizzle ORM or Prisma usage; vector storage design; or any data modeling problem. Trigger when
  the user says "schema", "table", "query is slow", "index", "migration", "database design",
  "ORM", "SQL", "relations", "normalization", or describes a data storage problem. Never design
  schemas that won't scale — always think 3 years ahead.
---

# Elite Database Architect

You design data systems that are correct, fast, and evolvable. You balance normalization with
query performance, think in access patterns first, and never ship a schema without indexes.

---

## PART 1 — SCHEMA DESIGN PRINCIPLES

### 1.1 Universal Table Template

```sql
-- Every table should have these columns:
CREATE TABLE entities (
    id          TEXT    PRIMARY KEY DEFAULT (lower(hex(randomblob(16)))),
    -- Use UUIDs (text hex) for distributed systems, not autoincrement integers
    -- Exception: high-write tables where integer PKs are needed for B-tree performance

    created_at  TEXT    NOT NULL DEFAULT (strftime('%Y-%m-%dT%H:%M:%SZ', 'now')),
    updated_at  TEXT    NOT NULL DEFAULT (strftime('%Y-%m-%dT%H:%M:%SZ', 'now')),
    deleted_at  TEXT    NULL,   -- NULL = active; timestamp = soft-deleted
    
    -- Your columns here
    tenant_id   TEXT    NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    -- Always include tenant_id for multi-tenant apps
    
    CONSTRAINT chk_dates CHECK (updated_at >= created_at)
);

-- Always index created_at for time-based queries
CREATE INDEX idx_entities_created_at ON entities(created_at);
-- Always index tenant_id for multi-tenant isolation
CREATE INDEX idx_entities_tenant_id ON entities(tenant_id);
-- Partial index for soft deletes — most queries only want active records
CREATE INDEX idx_entities_active ON entities(tenant_id, created_at) WHERE deleted_at IS NULL;
```

### 1.2 Naming Conventions

```
Tables:          plural snake_case    → users, order_items, api_keys
Columns:         singular snake_case  → user_id, created_at, is_active
Primary keys:    id                   → TEXT UUID
Foreign keys:    {table}_id           → user_id, order_id
Booleans:        is_ / has_ / can_    → is_active, has_verified_email
Timestamps:      _at suffix           → created_at, deleted_at, verified_at
Amounts:         store as integers    → price_cents (not price_dollars)
Enums:           TEXT with CHECK      → status TEXT CHECK(status IN ('active','inactive'))
Indexes:         idx_{table}_{col}    → idx_users_email
Unique:          uq_{table}_{col}     → uq_users_email
Foreign key:     fk_{table}_{ref}     → fk_orders_user_id
```

### 1.3 Data Type Decisions

```sql
-- IDs: TEXT hex UUIDs (portable, no autoincrement conflicts in distributed systems)
id TEXT PRIMARY KEY DEFAULT (lower(hex(randomblob(16))))

-- Timestamps: ISO 8601 TEXT (sortable, human-readable, timezone-safe)
created_at TEXT NOT NULL DEFAULT (strftime('%Y-%m-%dT%H:%M:%SZ', 'now'))

-- Money: INTEGER cents (never FLOAT — floating point imprecision kills fintech)
price_cents INTEGER NOT NULL  -- $19.99 stored as 1999

-- Enums: TEXT + CHECK constraint (flexible, readable, migratable)
status TEXT NOT NULL DEFAULT 'active' CHECK(status IN ('active','inactive','suspended'))

-- JSON/flexible data: TEXT JSON (index specific paths if queried)
metadata TEXT NOT NULL DEFAULT '{}'

-- Booleans: INTEGER 0/1 (SQLite has no native boolean)
is_verified INTEGER NOT NULL DEFAULT 0 CHECK(is_verified IN (0, 1))

-- Large text: TEXT (no size limit in SQLite)
content TEXT NOT NULL DEFAULT ''

-- Binary: BLOB (avatars, small files — large files → R2)
avatar_data BLOB
```

---

## PART 2 — INDEXING STRATEGY

### 2.1 Index Decision Framework

```
Always index:
  □ Every foreign key column
  □ Columns in WHERE clauses of frequent queries
  □ Columns in ORDER BY of paginated queries
  □ Email, phone, username (unique lookups)
  □ created_at on large tables (time-range queries)
  □ tenant_id on every table in multi-tenant systems

Composite index column order rule:
  (equality columns first, range column, sort column)

Example — query: WHERE tenant_id = ? AND status = ? AND created_at > ? ORDER BY created_at
→ CREATE INDEX idx ON table(tenant_id, status, created_at)
   ✓ tenant_id = equality
   ✓ status = equality
   ✓ created_at = range + sort

Never index:
  □ Columns with very low cardinality (boolean, is_deleted) — unless partial index
  □ Columns never queried
  □ More than 5-6 indexes per table (slows writes)
```

### 2.2 Partial Indexes (Most Underused Feature)

```sql
-- Only index active records — dramatically smaller, faster
CREATE INDEX idx_users_active_email
    ON users(email)
    WHERE deleted_at IS NULL;

-- Only index pending jobs — queue is typically tiny vs history
CREATE INDEX idx_jobs_pending
    ON background_jobs(scheduled_at, priority)
    WHERE status = 'pending';

-- Only index high-value customers
CREATE INDEX idx_orders_premium
    ON orders(user_id, created_at)
    WHERE total_cents > 100000;
```

### 2.3 Query Analysis

```sql
-- ALWAYS run EXPLAIN QUERY PLAN before shipping any new query
EXPLAIN QUERY PLAN
SELECT u.id, u.email, COUNT(o.id) as order_count
FROM users u
LEFT JOIN orders o ON o.user_id = u.id AND o.deleted_at IS NULL
WHERE u.tenant_id = 'abc' AND u.deleted_at IS NULL
GROUP BY u.id
ORDER BY order_count DESC
LIMIT 20;

-- Look for:
-- ✓ "USING INDEX" = good, using an index
-- ✗ "SCAN TABLE" = bad, full table scan
-- ✗ "USING TEMPORARY" = temp table created, may need composite index
-- ✗ "USING FILESORT" = sort without index
```

---

## PART 3 — COMMON SCHEMA PATTERNS

### 3.1 Multi-Tenancy

```sql
-- Row-level tenancy (recommended for SaaS — single DB, tenant_id on every table)
CREATE TABLE workspaces (
    id          TEXT PRIMARY KEY DEFAULT (lower(hex(randomblob(16)))),
    slug        TEXT NOT NULL UNIQUE,       -- URL-friendly identifier
    name        TEXT NOT NULL,
    plan        TEXT NOT NULL DEFAULT 'free' CHECK(plan IN ('free','pro','enterprise')),
    created_at  TEXT NOT NULL DEFAULT (strftime('%Y-%m-%dT%H:%M:%SZ', 'now'))
);

CREATE TABLE workspace_members (
    workspace_id TEXT NOT NULL REFERENCES workspaces(id) ON DELETE CASCADE,
    user_id      TEXT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    role         TEXT NOT NULL DEFAULT 'member' CHECK(role IN ('owner','admin','member','viewer')),
    joined_at    TEXT NOT NULL DEFAULT (strftime('%Y-%m-%dT%H:%M:%SZ', 'now')),
    PRIMARY KEY (workspace_id, user_id)
);

-- All tenant data tables include workspace_id
CREATE TABLE projects (
    id           TEXT PRIMARY KEY DEFAULT (lower(hex(randomblob(16)))),
    workspace_id TEXT NOT NULL REFERENCES workspaces(id) ON DELETE CASCADE,
    name         TEXT NOT NULL,
    created_at   TEXT NOT NULL DEFAULT (strftime('%Y-%m-%dT%H:%M:%SZ', 'now'))
);
CREATE INDEX idx_projects_workspace ON projects(workspace_id, created_at);
```

### 3.2 Audit Trail / Event Log

```sql
-- Immutable audit log — never update or delete rows
CREATE TABLE audit_events (
    id           TEXT PRIMARY KEY DEFAULT (lower(hex(randomblob(16)))),
    workspace_id TEXT NOT NULL REFERENCES workspaces(id),
    actor_id     TEXT NOT NULL,             -- user who performed the action
    actor_type   TEXT NOT NULL DEFAULT 'user',  -- user | system | api_key
    resource_type TEXT NOT NULL,            -- 'project', 'user', 'order'
    resource_id  TEXT NOT NULL,
    action       TEXT NOT NULL,             -- 'created', 'updated', 'deleted'
    before_state TEXT,                      -- JSON snapshot of record before
    after_state  TEXT,                      -- JSON snapshot of record after
    ip_address   TEXT,
    user_agent   TEXT,
    created_at   TEXT NOT NULL DEFAULT (strftime('%Y-%m-%dT%H:%M:%SZ', 'now'))
);

CREATE INDEX idx_audit_workspace_created ON audit_events(workspace_id, created_at DESC);
CREATE INDEX idx_audit_resource ON audit_events(resource_type, resource_id, created_at DESC);
CREATE INDEX idx_audit_actor ON audit_events(actor_id, created_at DESC);
```

### 3.3 Hierarchical Data (Adjacency List + Closure Table)

```sql
-- Adjacency list: simple, works for shallow hierarchies
CREATE TABLE categories (
    id        TEXT PRIMARY KEY DEFAULT (lower(hex(randomblob(16)))),
    parent_id TEXT REFERENCES categories(id) ON DELETE SET NULL,
    name      TEXT NOT NULL,
    path      TEXT NOT NULL,   -- materialized path: "root/parent/child"
    depth     INTEGER NOT NULL DEFAULT 0
);

-- Closure table: efficient for deep hierarchies with frequent ancestor queries
CREATE TABLE category_ancestors (
    ancestor_id   TEXT NOT NULL REFERENCES categories(id) ON DELETE CASCADE,
    descendant_id TEXT NOT NULL REFERENCES categories(id) ON DELETE CASCADE,
    depth         INTEGER NOT NULL,
    PRIMARY KEY (ancestor_id, descendant_id)
);

-- Get all descendants of a category:
SELECT c.* FROM categories c
JOIN category_ancestors ca ON ca.descendant_id = c.id
WHERE ca.ancestor_id = :category_id AND ca.depth > 0;
```

### 3.4 Subscriptions / Billing

```sql
CREATE TABLE subscriptions (
    id                TEXT PRIMARY KEY DEFAULT (lower(hex(randomblob(16)))),
    workspace_id      TEXT NOT NULL UNIQUE REFERENCES workspaces(id),
    stripe_customer_id     TEXT UNIQUE,
    stripe_subscription_id TEXT UNIQUE,
    plan              TEXT NOT NULL DEFAULT 'free',
    status            TEXT NOT NULL DEFAULT 'active'
                      CHECK(status IN ('trialing','active','past_due','canceled','paused')),
    current_period_start TEXT,
    current_period_end   TEXT,
    cancel_at_period_end INTEGER NOT NULL DEFAULT 0,
    trial_end            TEXT,
    created_at        TEXT NOT NULL DEFAULT (strftime('%Y-%m-%dT%H:%M:%SZ', 'now')),
    updated_at        TEXT NOT NULL DEFAULT (strftime('%Y-%m-%dT%H:%M:%SZ', 'now'))
);

CREATE TABLE subscription_events (
    id              TEXT PRIMARY KEY DEFAULT (lower(hex(randomblob(16)))),
    subscription_id TEXT NOT NULL REFERENCES subscriptions(id),
    event_type      TEXT NOT NULL,   -- stripe event type
    stripe_event_id TEXT NOT NULL UNIQUE,
    payload         TEXT NOT NULL,   -- full Stripe event JSON
    processed_at    TEXT NOT NULL DEFAULT (strftime('%Y-%m-%dT%H:%M:%SZ', 'now'))
);
```

### 3.5 Cursor-Based Pagination

```sql
-- NEVER use OFFSET pagination for large tables (reads N rows every time)
-- ALWAYS use cursor-based (keyset) pagination

-- First page
SELECT * FROM orders
WHERE workspace_id = :workspace_id
  AND deleted_at IS NULL
ORDER BY created_at DESC, id DESC
LIMIT 20;

-- Next page (cursor = last row's created_at + id)
SELECT * FROM orders
WHERE workspace_id = :workspace_id
  AND deleted_at IS NULL
  AND (created_at < :cursor_created_at
       OR (created_at = :cursor_created_at AND id < :cursor_id))
ORDER BY created_at DESC, id DESC
LIMIT 20;
```

---

## PART 4 — DRIZZLE ORM (TypeScript)

### 4.1 Schema Definition

```typescript
import { sqliteTable, text, integer, index, uniqueIndex } from 'drizzle-orm/sqlite-core';
import { sql } from 'drizzle-orm';

const timestamps = {
  createdAt: text('created_at').notNull().default(sql`(strftime('%Y-%m-%dT%H:%M:%SZ', 'now'))`),
  updatedAt: text('updated_at').notNull().default(sql`(strftime('%Y-%m-%dT%H:%M:%SZ', 'now'))`),
  deletedAt: text('deleted_at'),
};

export const users = sqliteTable('users', {
  id: text('id').primaryKey().$defaultFn(() => crypto.randomUUID()),
  email: text('email').notNull(),
  displayName: text('display_name').notNull(),
  passwordHash: text('password_hash'),
  ...timestamps,
}, (table) => ({
  emailIdx: uniqueIndex('uq_users_email').on(table.email),
  createdAtIdx: index('idx_users_created_at').on(table.createdAt),
}));

export const workspaces = sqliteTable('workspaces', {
  id: text('id').primaryKey().$defaultFn(() => crypto.randomUUID()),
  slug: text('slug').notNull(),
  name: text('name').notNull(),
  plan: text('plan', { enum: ['free', 'pro', 'enterprise'] }).notNull().default('free'),
  ...timestamps,
}, (table) => ({
  slugIdx: uniqueIndex('uq_workspaces_slug').on(table.slug),
}));
```

### 4.2 Query Patterns

```typescript
import { drizzle } from 'drizzle-orm/d1';
import { eq, and, isNull, lt, desc, count } from 'drizzle-orm';

const db = drizzle(env.DB);

// Simple find with soft-delete filter
const user = await db
  .select()
  .from(users)
  .where(and(eq(users.email, email), isNull(users.deletedAt)))
  .get();

// Join with aggregation
const workspaceStats = await db
  .select({
    workspaceId: workspaces.id,
    name: workspaces.name,
    memberCount: count(workspaceMembers.userId),
  })
  .from(workspaces)
  .leftJoin(workspaceMembers, eq(workspaceMembers.workspaceId, workspaces.id))
  .where(isNull(workspaces.deletedAt))
  .groupBy(workspaces.id)
  .all();

// Cursor pagination
const page = await db
  .select()
  .from(orders)
  .where(
    and(
      eq(orders.workspaceId, workspaceId),
      isNull(orders.deletedAt),
      cursor ? lt(orders.createdAt, cursor) : undefined
    )
  )
  .orderBy(desc(orders.createdAt))
  .limit(pageSize + 1)   // fetch +1 to know if there's a next page
  .all();

const hasNextPage = page.length > pageSize;
const items = hasNextPage ? page.slice(0, -1) : page;
const nextCursor = hasNextPage ? items[items.length - 1].createdAt : null;
```

### 4.3 Migrations

```bash
# Generate migration from schema changes
npx drizzle-kit generate

# Apply migrations locally
npx drizzle-kit migrate

# Apply to D1 (staging)
npx wrangler d1 migrations apply DB --env staging

# Apply to D1 (production — after staging verified)
npx wrangler d1 migrations apply DB --env production
```

---

## PART 5 — CACHING STRATEGY

### 5.1 Cache Hierarchy

```
L1: In-memory (Worker execution context)   → microseconds, ephemeral
L2: Cloudflare KV                          → ~10ms, global, eventually consistent
L3: Database (D1 / PostgreSQL)             → ~20-100ms, authoritative

Cache-aside pattern (most common):
  1. Check KV for cached value
  2. On miss: query DB, write to KV with TTL, return
  3. On write: invalidate or update KV

Cache TTL guidelines:
  User sessions:        24h (rotate on activity)
  Public content:       1h–24h
  User-specific data:   5min–1h
  Real-time data:       30s–5min
  Never cache:          payment data, passwords, tokens
```

```typescript
async function getCachedUser(userId: string, env: Env): Promise<User | null> {
  const cacheKey = `user:${userId}`;
  
  // L2: Check KV
  const cached = await env.KV.get(cacheKey, 'json');
  if (cached) return cached as User;
  
  // L3: Query DB
  const user = await db.select().from(users).where(eq(users.id, userId)).get();
  if (!user) return null;
  
  // Populate L2
  await env.KV.put(cacheKey, JSON.stringify(user), { expirationTtl: 300 });  // 5min
  return user;
}

async function invalidateUserCache(userId: string, env: Env): Promise<void> {
  await env.KV.delete(`user:${userId}`);
}
```

---

## PART 6 — PERFORMANCE CHECKLIST

```
Schema:
□ Every foreign key indexed
□ Composite indexes match query WHERE + ORDER BY patterns
□ Partial indexes for soft-delete and status filters
□ No N+1 queries (use JOINs or batch fetching)
□ Money stored as integers (cents)
□ Timestamps stored as ISO text (sortable)

Queries:
□ EXPLAIN QUERY PLAN run on all queries touching >10K rows
□ Cursor-based pagination (not OFFSET) for lists
□ Batch inserts with db.batch() not N individual inserts
□ Projections (SELECT col1, col2 not SELECT *)

Operations:
□ Migrations backward-compatible (additive first, cleanup in next deploy)
□ Long-running migrations run as background jobs
□ Index creation in migrations uses IF NOT EXISTS
□ Rollback plan for every migration
```

---

> **Core Principle**: Your database outlives your application code. Design schemas
> for the data that needs to exist, not just the features you're building today.
> Index what you query, migrate without downtime, and measure before optimizing.
