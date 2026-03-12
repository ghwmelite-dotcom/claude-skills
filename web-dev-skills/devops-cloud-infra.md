---
name: devops-cloud-infra
description: >
  Activates elite DevOps and cloud infrastructure engineering capabilities. Use this skill for
  ANY task involving: CI/CD pipelines (GitHub Actions, Cloudflare Pages/Workers deployment);
  Docker containerization and orchestration; infrastructure as code; environment management
  (dev/staging/prod); monitoring and observability (Sentry, Grafana, Datadog, Cloudflare
  Analytics); secrets management; zero-downtime deployments; database migrations in production;
  performance monitoring; error tracking; log aggregation; health checks and alerting; or any
  task related to shipping, deploying, or operating software reliably in production. Trigger when
  the user mentions "deploy", "CI/CD", "pipeline", "Docker", "staging", "production", "monitoring",
  "alerts", "logs", "secrets", or "infrastructure". Never ship without observability.
---

# Elite DevOps & Cloud Infrastructure Engineer

You design and operate production systems that are reliable, observable, and deployable with
confidence. Every system ships with CI/CD, monitoring, error tracking, and runbooks.

---

## PART 1 — CI/CD WITH GITHUB ACTIONS

### 1.1 Full Pipeline (Test → Build → Deploy)

```yaml
# .github/workflows/deploy.yml
name: CI/CD Pipeline

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

env:
  NODE_VERSION: '22'

jobs:
  # ─── Test ────────────────────────────────────────────────
  test:
    name: Test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'

      - run: npm ci

      - name: Type check
        run: npm run type-check

      - name: Lint
        run: npm run lint

      - name: Unit tests
        run: npm run test:unit -- --coverage

      - name: Upload coverage
        uses: codecov/codecov-action@v4
        with:
          token: ${{ secrets.CODECOV_TOKEN }}

  # ─── Build ───────────────────────────────────────────────
  build:
    name: Build
    runs-on: ubuntu-latest
    needs: test
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '${{ env.NODE_VERSION }}', cache: 'npm' }
      - run: npm ci
      - run: npm run build
      - uses: actions/upload-artifact@v4
        with: { name: dist, path: dist/ }

  # ─── Deploy Staging ──────────────────────────────────────
  deploy-staging:
    name: Deploy → Staging
    runs-on: ubuntu-latest
    needs: build
    if: github.event_name == 'pull_request'
    environment: staging
    steps:
      - uses: actions/checkout@v4
      - uses: actions/download-artifact@v4
        with: { name: dist, path: dist/ }

      - name: Deploy to Cloudflare Pages (preview)
        uses: cloudflare/wrangler-action@v3
        with:
          apiToken: ${{ secrets.CF_API_TOKEN }}
          accountId: ${{ secrets.CF_ACCOUNT_ID }}
          command: pages deploy dist/ --project-name=my-app --branch=staging

  # ─── Deploy Production ───────────────────────────────────
  deploy-production:
    name: Deploy → Production
    runs-on: ubuntu-latest
    needs: build
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    environment: production
    steps:
      - uses: actions/checkout@v4
      - uses: actions/download-artifact@v4
        with: { name: dist, path: dist/ }

      - name: Run DB migrations
        run: npx wrangler d1 migrations apply DB --env production
        env:
          CLOUDFLARE_API_TOKEN: ${{ secrets.CF_API_TOKEN }}

      - name: Deploy Workers
        uses: cloudflare/wrangler-action@v3
        with:
          apiToken: ${{ secrets.CF_API_TOKEN }}
          accountId: ${{ secrets.CF_ACCOUNT_ID }}
          command: deploy --env production

      - name: Notify Sentry of release
        run: |
          npx @sentry/cli releases new ${{ github.sha }}
          npx @sentry/cli releases set-commits ${{ github.sha }} --auto
          npx @sentry/cli releases finalize ${{ github.sha }}
        env:
          SENTRY_AUTH_TOKEN: ${{ secrets.SENTRY_AUTH_TOKEN }}
          SENTRY_ORG: ${{ secrets.SENTRY_ORG }}
          SENTRY_PROJECT: ${{ secrets.SENTRY_PROJECT }}
```

### 1.2 PR Checks Pipeline

```yaml
# .github/workflows/pr-checks.yml
name: PR Checks

on:
  pull_request:
    types: [opened, synchronize, reopened]

jobs:
  security-audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm audit --audit-level=high

  dependency-review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/dependency-review-action@v4

  bundle-size:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci && npm run build
      - uses: andresz1/size-limit-action@v1
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}

  e2e:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '22', cache: 'npm' }
      - run: npm ci
      - run: npx playwright install --with-deps chromium
      - run: npm run test:e2e
      - uses: actions/upload-artifact@v4
        if: failure()
        with:
          name: playwright-report
          path: playwright-report/
```

---

## PART 2 — CLOUDFLARE DEPLOYMENT

### 2.1 Wrangler Configuration

```toml
# wrangler.toml
name = "my-api"
main = "src/index.ts"
compatibility_date = "2024-12-01"
compatibility_flags = ["nodejs_compat"]

# ─── Environments ────────────────────────────────────────
[env.staging]
name = "my-api-staging"
vars = { ENVIRONMENT = "staging", LOG_LEVEL = "debug" }

[env.production]
name = "my-api-production"
vars = { ENVIRONMENT = "production", LOG_LEVEL = "info" }

# ─── Bindings ────────────────────────────────────────────
[[d1_databases]]
binding = "DB"
database_name = "my-app-db"
database_id = "xxxx-xxxx-xxxx-xxxx"

[[kv_namespaces]]
binding = "KV"
id = "xxxx-xxxx-xxxx-xxxx"
preview_id = "xxxx-xxxx-xxxx-xxxx"

[[r2_buckets]]
binding = "STORAGE"
bucket_name = "my-app-storage"

[ai]
binding = "AI"

# ─── Observability ───────────────────────────────────────
[observability]
enabled = true
head_sampling_rate = 1

# ─── Cron Triggers ───────────────────────────────────────
[triggers]
crons = ["0 2 * * *"]   # 2am UTC daily

# ─── Limits ──────────────────────────────────────────────
[limits]
cpu_ms = 50             # Default Worker CPU limit
```

### 2.2 Zero-Downtime Database Migrations

```typescript
// migrations/0001_add_users.sql
-- Migration: add users table
-- Direction: forward
-- Created: 2024-01-15

CREATE TABLE IF NOT EXISTS users (
    id          TEXT PRIMARY KEY DEFAULT (lower(hex(randomblob(16)))),
    email       TEXT NOT NULL UNIQUE,
    display_name TEXT NOT NULL,
    created_at  TEXT NOT NULL DEFAULT (strftime('%Y-%m-%dT%H:%M:%SZ', 'now')),
    updated_at  TEXT NOT NULL DEFAULT (strftime('%Y-%m-%dT%H:%M:%SZ', 'now'))
);

CREATE INDEX IF NOT EXISTS idx_users_email ON users(email);
CREATE INDEX IF NOT EXISTS idx_users_created_at ON users(created_at);
```

```bash
# Apply migrations via wrangler
npx wrangler d1 migrations apply DB --env production

# Rollback: D1 doesn't support automatic rollback
# Always write backward-compatible migrations:
# - Add columns as nullable first
# - Never rename/delete columns in the same deploy as code that uses old name
# - Use feature flags to gate new columns until migration is complete
```

### 2.3 Deployment Checklist

```
Pre-deploy:
□ All tests green (unit + e2e)
□ Type check passes
□ No high/critical security vulnerabilities (npm audit)
□ Bundle size within budget
□ DB migrations tested on staging
□ Feature flags configured
□ Rollback plan documented

Post-deploy:
□ Smoke test critical paths in production
□ Check error rate in Sentry (should not spike)
□ Check p95 latency in analytics
□ Verify DB migrations applied cleanly
□ Check Worker CPU usage
□ Sentry release tagged
```

---

## PART 3 — DOCKER

### 3.1 Production Dockerfile (Node.js)

```dockerfile
# syntax=docker/dockerfile:1
FROM node:22-alpine AS base

# ─── Dependencies ────────────────────────────────────────
FROM base AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production && cp -R node_modules prod_modules
RUN npm ci

# ─── Builder ─────────────────────────────────────────────
FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npm run build

# ─── Production ──────────────────────────────────────────
FROM base AS production
WORKDIR /app

# Security: non-root user
RUN addgroup --system --gid 1001 nodejs \
 && adduser --system --uid 1001 app
USER app

COPY --from=deps --chown=app:nodejs /app/prod_modules ./node_modules
COPY --from=builder --chown=app:nodejs /app/dist ./dist
COPY --from=builder --chown=app:nodejs /app/package.json ./

# Health check
HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
  CMD wget -qO- http://localhost:3000/health || exit 1

EXPOSE 3000
ENV NODE_ENV=production PORT=3000

CMD ["node", "dist/index.js"]
```

### 3.2 Docker Compose (Local Dev)

```yaml
# docker-compose.yml
version: '3.9'

services:
  app:
    build:
      context: .
      target: builder
    ports: ['3000:3000']
    environment:
      - NODE_ENV=development
      - DATABASE_URL=postgresql://app:password@postgres:5432/app
      - REDIS_URL=redis://redis:6379
    volumes:
      - .:/app
      - /app/node_modules     # Prevent host overwriting container modules
    depends_on:
      postgres: { condition: service_healthy }
      redis: { condition: service_started }
    command: npm run dev

  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: app
      POSTGRES_USER: app
      POSTGRES_PASSWORD: password
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ['CMD-SHELL', 'pg_isready -U app']
      interval: 5s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    volumes:
      - redis_data:/data

volumes:
  postgres_data:
  redis_data:
```

---

## PART 4 — MONITORING & OBSERVABILITY

### 4.1 Sentry Setup (Error Tracking)

```typescript
// src/lib/sentry.ts
import * as Sentry from '@sentry/cloudflare';

export function initSentry(env: Env): void {
  Sentry.init({
    dsn: env.SENTRY_DSN,
    environment: env.ENVIRONMENT,
    tracesSampleRate: env.ENVIRONMENT === 'production' ? 0.1 : 1.0,
    profilesSampleRate: 0.1,
    integrations: [Sentry.rewriteFramesIntegration()],
  });
}

// Wrap Worker handler
export const withSentry = (handler: ExportedHandler): ExportedHandler => ({
  async fetch(request, env, ctx) {
    return Sentry.withIsolationScope(async () => {
      Sentry.setTag('worker', 'my-api');
      Sentry.setUser({ id: getUserId(request) });
      try {
        return await handler.fetch!(request, env, ctx);
      } catch (error) {
        Sentry.captureException(error);
        throw error;
      }
    });
  },
});
```

### 4.2 Structured Logging

```typescript
// src/lib/logger.ts
export type LogLevel = 'debug' | 'info' | 'warn' | 'error';

interface LogEntry {
  level: LogLevel;
  message: string;
  requestId?: string;
  userId?: string;
  durationMs?: number;
  error?: { message: string; stack?: string; code?: string };
  [key: string]: unknown;
}

export function createLogger(requestId: string, userId?: string) {
  function log(level: LogLevel, message: string, data?: Record<string, unknown>) {
    const entry: LogEntry = {
      level,
      message,
      requestId,
      userId,
      timestamp: new Date().toISOString(),
      ...data,
    };
    
    const output = JSON.stringify(entry);
    
    if (level === 'error' || level === 'warn') {
      console.error(output);
    } else {
      console.log(output);
    }
  }
  
  return {
    debug: (msg: string, data?: Record<string, unknown>) => log('debug', msg, data),
    info:  (msg: string, data?: Record<string, unknown>) => log('info', msg, data),
    warn:  (msg: string, data?: Record<string, unknown>) => log('warn', msg, data),
    error: (msg: string, error?: unknown, data?: Record<string, unknown>) => {
      const errorData = error instanceof Error
        ? { error: { message: error.message, stack: error.stack } }
        : { error };
      log('error', msg, { ...errorData, ...data });
    },
  };
}
```

### 4.3 Health Check Endpoint

```typescript
// GET /health — Used by load balancers, uptime monitors, Docker HEALTHCHECK
app.get('/health', async (c) => {
  const checks = await Promise.allSettled([
    checkDatabase(c.env.DB),
    checkKV(c.env.KV),
  ]);

  const status = checks.every(r => r.status === 'fulfilled') ? 'healthy' : 'degraded';
  const code = status === 'healthy' ? 200 : 503;

  return c.json({
    status,
    version: c.env.VERSION ?? 'unknown',
    timestamp: new Date().toISOString(),
    checks: {
      database: checks[0].status === 'fulfilled' ? 'ok' : 'fail',
      kv: checks[1].status === 'fulfilled' ? 'ok' : 'fail',
    },
  }, code);
});
```

### 4.4 Performance Monitoring

```typescript
// Request timing middleware
app.use('*', async (c, next) => {
  const start = Date.now();
  const requestId = c.req.header('cf-ray') ?? crypto.randomUUID();

  c.set('requestId', requestId);
  c.set('logger', createLogger(requestId, getUserId(c)));

  await next();

  const durationMs = Date.now() - start;
  c.header('X-Request-Id', requestId);
  c.header('X-Response-Time', `${durationMs}ms`);

  c.get('logger').info('Request completed', {
    method: c.req.method,
    path: c.req.path,
    status: c.res.status,
    durationMs,
  });
});
```

---

## PART 5 — SECRETS MANAGEMENT

### 5.1 Cloudflare Secrets

```bash
# Set secrets (never in wrangler.toml!)
npx wrangler secret put ANTHROPIC_API_KEY --env production
npx wrangler secret put DATABASE_PASSWORD --env production
npx wrangler secret put JWT_SECRET --env production
npx wrangler secret put STRIPE_SECRET_KEY --env production

# List configured secrets
npx wrangler secret list --env production

# Rotate a secret (zero downtime)
# 1. Add new secret with temporary name
# 2. Deploy code that reads both old + new
# 3. Update to new name
# 4. Deploy code reading only new
# 5. Delete old secret
```

### 5.2 Environment Variable Typing

```typescript
// src/types/env.d.ts
export interface Env {
  // KV Namespaces
  KV: KVNamespace;
  SESSIONS: KVNamespace;

  // D1 Databases
  DB: D1Database;

  // R2 Buckets
  STORAGE: R2Bucket;

  // Secrets (string)
  ANTHROPIC_API_KEY: string;
  JWT_SECRET: string;
  STRIPE_SECRET_KEY: string;

  // Variables
  ENVIRONMENT: 'development' | 'staging' | 'production';
  VERSION: string;

  // AI
  AI: Ai;
  VECTORIZE: VectorizeIndex;
}
```

---

## PART 6 — INFRASTRUCTURE AS CODE

### 6.1 Terraform for Multi-Cloud (when needed)

```hcl
# Only use Terraform for complex multi-service infrastructure
# For Cloudflare-only stacks, prefer wrangler.toml

terraform {
  required_providers {
    cloudflare = { source = "cloudflare/cloudflare", version = "~> 4" }
  }
  backend "s3" {
    bucket = "my-terraform-state"
    key    = "production/terraform.tfstate"
    region = "us-east-1"
  }
}

resource "cloudflare_d1_database" "app_db" {
  account_id = var.cloudflare_account_id
  name       = "my-app-production"
}

resource "cloudflare_r2_bucket" "storage" {
  account_id = var.cloudflare_account_id
  name       = "my-app-storage"
  location   = "WEUR"
}

resource "cloudflare_worker_domain" "api" {
  account_id = var.cloudflare_account_id
  hostname   = "api.myapp.com"
  service    = "my-api-production"
  zone_id    = var.cloudflare_zone_id
}
```

---

## PART 7 — RELIABILITY PATTERNS

```
SLO Targets (define before building):
  Availability:  99.9% (44min downtime/month)
  Latency p95:   < 500ms (API), < 2s (page load)
  Error rate:    < 0.1%

Error Budget:
  1 - SLO = error budget
  99.9% SLO = 0.1% error budget = 43.8min/month
  Burn rate alerts: 1hr window, 2% burn rate → page immediately

Deployment Safety:
  - Canary deploys: 5% → 20% → 100% traffic over 30min
  - Feature flags (Cloudflare KV): decouple deploy from release
  - Rollback SLA: < 5 minutes (Wrangler rollback command)

Incident Response:
  1. Alert fires (Sentry / Uptime monitor)
  2. Acknowledge within 5min (PagerDuty / phone)
  3. Assess impact (users affected, revenue, data loss)
  4. Mitigate (rollback, feature flag off, scale)
  5. Resolve
  6. Post-mortem within 48hrs (no blame, 5 whys)
```

---

> **Core Principle**: Deploy with confidence, observe everything, fix forward fast.
> A system without monitoring is a system you're flying blind. Instrument first,
> optimize later. The best on-call is the one who gets paged least.
