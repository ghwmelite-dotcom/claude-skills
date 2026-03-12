---
name: technical-writing
description: >
  Activates elite technical writing and documentation capabilities. Use this skill for ANY task
  involving: writing or improving README files; API documentation (OpenAPI/Swagger specs); user
  guides and onboarding docs; developer documentation; changelogs and release notes; architecture
  decision records (ADRs); runbooks and incident response guides; code comments and docstrings;
  SDK documentation; tutorial and how-to guides; FAQ pages; technical blog posts; product
  requirement documents (PRDs); or any written artifact that communicates technical information
  to developers, users, or stakeholders. Trigger when the user says "write docs", "README",
  "document this", "OpenAPI", "changelog", "runbook", "user guide", "explain how", or "write a
  guide for". Good documentation is a product feature — ship it with the same quality as code.
---

# Elite Technical Writer

You write documentation that developers actually read, users actually follow, and teams
actually maintain. Clear, scannable, accurate, and always audience-aware.

---

## PART 1 — README TEMPLATE

### 1.1 Production README Structure

```markdown
# Project Name

> One-line tagline that explains what this does and who it's for.

[![Build Status](badge)](link)
[![npm version](badge)](link)
[![License](badge)](link)

## What is this?

2-3 sentences explaining the problem this solves and the solution.
Be concrete: "X lets you do Y without Z" not "X is a powerful tool."

## Quick Start

The fastest path to working — should take under 5 minutes.

```bash
# Install
npm install my-package

# Configure
cp .env.example .env
# Add your ANTHROPIC_API_KEY to .env

# Run
npm run dev
```

Visit http://localhost:3000 → you should see [describe what success looks like].

## Features

- **Feature 1** — short description of the benefit
- **Feature 2** — short description of the benefit
- **Feature 3** — short description of the benefit

## Installation

### Prerequisites

- Node.js 22+
- A [Cloudflare account](https://dash.cloudflare.com) (free)

### Step-by-step

1. Clone the repository:
   ```bash
   git clone https://github.com/user/repo.git
   cd repo
   ```

2. Install dependencies:
   ```bash
   npm install
   ```

3. Set up environment variables:
   ```bash
   cp .env.example .env
   ```
   Edit `.env` and add:
   - `ANTHROPIC_API_KEY` — get from [console.anthropic.com](https://console.anthropic.com)
   - `DATABASE_URL` — your PostgreSQL connection string

4. Run database migrations:
   ```bash
   npm run db:migrate
   ```

5. Start the development server:
   ```bash
   npm run dev
   ```

## Configuration

| Variable | Required | Default | Description |
|---|---|---|---|
| `ANTHROPIC_API_KEY` | ✓ | — | Anthropic API key for Claude |
| `DATABASE_URL` | ✓ | — | PostgreSQL connection string |
| `PORT` | | `3000` | Server port |
| `LOG_LEVEL` | | `info` | Logging verbosity (`debug`/`info`/`warn`/`error`) |

## Usage

### Basic example

```typescript
import { MyClient } from 'my-package';

const client = new MyClient({ apiKey: process.env.ANTHROPIC_API_KEY });

const result = await client.process({
  input: 'your data here',
  options: { format: 'json' },
});

console.log(result.output);
```

### Advanced example

[Show a more complex real-world usage]

## API Reference

See [API documentation](./docs/api.md) for full reference.

## Contributing

We welcome contributions! Please read [CONTRIBUTING.md](./CONTRIBUTING.md) first.

```bash
# Run tests
npm test

# Run linting
npm run lint

# Build
npm run build
```

## Changelog

See [CHANGELOG.md](./CHANGELOG.md) for release history.

## License

MIT — see [LICENSE](./LICENSE) for details.
```

---

## PART 2 — OPENAPI SPECIFICATION

### 2.1 Full OpenAPI 3.1 Template

```yaml
# openapi.yaml
openapi: '3.1.0'

info:
  title: MyApp API
  version: '1.0.0'
  description: |
    The MyApp REST API enables you to programmatically manage your workspace,
    projects, and data.

    ## Authentication

    All API requests require a Bearer token in the Authorization header:
    ```
    Authorization: Bearer <your-api-key>
    ```

    Generate API keys in [Settings → API Keys](https://myapp.com/settings/api).

    ## Rate Limiting

    Requests are limited to 1,000/hour per API key. Rate limit info is returned
    in response headers:
    - `X-RateLimit-Limit`: Total requests allowed per hour
    - `X-RateLimit-Remaining`: Requests remaining in current window
    - `X-RateLimit-Reset`: Unix timestamp when the window resets

    ## Errors

    All errors return a consistent shape:
    ```json
    {
      "error": {
        "code": "VALIDATION_ERROR",
        "message": "Email address is invalid",
        "details": { "field": "email" },
        "requestId": "req_abc123"
      }
    }
    ```

  contact:
    name: API Support
    email: api@myapp.com
    url: https://docs.myapp.com
  license:
    name: MIT
    url: https://opensource.org/licenses/MIT

servers:
  - url: https://api.myapp.com/v1
    description: Production
  - url: https://api.staging.myapp.com/v1
    description: Staging

security:
  - BearerAuth: []

tags:
  - name: Users
    description: User account management
  - name: Projects
    description: Project creation and management
  - name: Webhooks
    description: Webhook configuration

paths:
  /users/{userId}:
    get:
      operationId: getUser
      summary: Get a user
      description: Retrieves a user by their ID.
      tags: [Users]
      parameters:
        - name: userId
          in: path
          required: true
          description: The unique user identifier
          schema:
            type: string
            format: uuid
            example: '01234567-89ab-cdef-0123-456789abcdef'
      responses:
        '200':
          description: User found
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
        '404':
          $ref: '#/components/responses/NotFound'
        '401':
          $ref: '#/components/responses/Unauthorized'

  /projects:
    post:
      operationId: createProject
      summary: Create a project
      description: |
        Creates a new project in the authenticated user's workspace.

        Projects are subject to plan limits. Free plans allow up to 3 projects.
        Attempting to create more will return a `402 Payment Required` error.
      tags: [Projects]
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateProjectRequest'
            examples:
              basic:
                summary: Basic project
                value:
                  name: "My API Project"
                  description: "A project for testing the API"
      responses:
        '201':
          description: Project created
          headers:
            Location:
              description: URL of the created project
              schema:
                type: string
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Project'
        '402':
          $ref: '#/components/responses/PlanLimitExceeded'

components:
  securitySchemes:
    BearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
      description: API key from Settings → API Keys

  schemas:
    User:
      type: object
      required: [id, email, displayName, createdAt]
      properties:
        id:
          type: string
          format: uuid
          description: Unique user identifier
          readOnly: true
          example: '01234567-89ab-cdef-0123-456789abcdef'
        email:
          type: string
          format: email
          description: User's email address
          example: 'user@example.com'
        displayName:
          type: string
          description: User's display name
          minLength: 1
          maxLength: 100
          example: 'Ozzy Mensah'
        avatarUrl:
          type: string
          format: uri
          nullable: true
          description: URL of user's avatar image
        createdAt:
          type: string
          format: date-time
          readOnly: true

    CreateProjectRequest:
      type: object
      required: [name]
      properties:
        name:
          type: string
          minLength: 1
          maxLength: 100
          description: Project display name
        description:
          type: string
          maxLength: 500
          description: Optional project description

  responses:
    Unauthorized:
      description: Missing or invalid authentication
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Error'
    NotFound:
      description: Resource not found
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Error'
    PlanLimitExceeded:
      description: Plan limit reached — upgrade required
      content:
        application/json:
          schema:
            allOf:
              - $ref: '#/components/schemas/Error'
              - type: object
                properties:
                  upgradeUrl:
                    type: string
                    format: uri
```

---

## PART 3 — CHANGELOG FORMAT (KEEP A CHANGELOG)

```markdown
# Changelog

All notable changes to this project will be documented here.
Format: [Keep a Changelog](https://keepachangelog.com/en/1.1.0/)
Versioning: [Semantic Versioning](https://semver.org/spec/v2.0.0.html)

## [Unreleased]

### Added
- New feature in development

---

## [2.1.0] — 2025-03-10

### Added
- **AI-powered search**: Semantic search across all projects using embeddings (#234)
- **Webhook signatures**: All webhook payloads now include HMAC-SHA256 signature for verification
- `DELETE /api/projects/{id}` endpoint for project deletion

### Changed
- **Breaking**: `user.name` field renamed to `user.displayName` in all API responses
  — update your integration before upgrading
- Improved error messages for validation failures — now include field-level detail
- Rate limits increased to 1,000 req/hour for Pro plan (was 500)

### Fixed
- Fixed race condition in concurrent project creation that could create duplicate slugs (#289)
- Fixed timezone handling in date filters — all dates now consistently use UTC
- Fixed memory leak in WebSocket connection handler

### Deprecated
- `POST /api/search/legacy` — will be removed in v3.0.0. Use `POST /api/search` instead.

### Security
- Updated `jose` dependency to patch CVE-2024-XXXXX (token validation bypass)

---

## [2.0.0] — 2025-01-15

### Breaking Changes

**API versioning**: The API base URL has changed from `/api/` to `/api/v2/`.
All v1 endpoints continue to work until 2025-07-01.

**Authentication**: API keys now use `Bearer` scheme instead of `X-API-Key` header.

**Migration guide**: See [2.0 Migration Guide](./docs/migrations/v2.md)
```

---

## PART 4 — ARCHITECTURE DECISION RECORDS (ADRs)

```markdown
# ADR-0003: Use Cloudflare D1 Instead of PlanetScale

**Date**: 2025-01-20
**Status**: Accepted
**Deciders**: Ozzy Mensah

## Context

We need a database for our Cloudflare Workers-based API. We evaluated:
1. PlanetScale (MySQL, serverless, globally distributed)
2. Neon (PostgreSQL, serverless, branching)
3. Cloudflare D1 (SQLite, edge-native, integrated with Workers)

## Decision

We will use **Cloudflare D1**.

## Rationale

**For D1:**
- Zero latency to Workers (same infrastructure, no network hop)
- No connection pooling complexity (Workers are stateless)
- Free tier is generous for MVP scale
- Single vendor stack reduces operational complexity
- Strong consistency within a region

**Against alternatives:**
- PlanetScale: MySQL dialect, separate vendor, latency to Workers
- Neon: PostgreSQL (more features), but same separate-vendor problem

## Consequences

**Positive:**
- Simpler deployment (wrangler manages everything)
- Lower cost at current scale
- SQLite dialect well-supported by Drizzle ORM

**Negative:**
- SQLite has fewer features than PostgreSQL (no arrays, limited JSON ops)
- Global replication is read-only (writes go to primary)
- May need to migrate if we hit SQLite scaling limits

**Revisit if:** monthly active users exceed 100K or query complexity requires PostgreSQL features.
```

---

## PART 5 — RUNBOOK TEMPLATE

```markdown
# Runbook: API High Error Rate

**Alert**: `api_error_rate > 1% for 5 minutes`
**Severity**: P1 (page on-call)
**Last Updated**: 2025-03-01

## Impact

Users cannot complete API requests. All features depending on the API are degraded or down.

## Immediate Triage (< 5 minutes)

1. **Check Sentry** → [Production Errors](https://sentry.io/organizations/myapp/issues/)
   - Look for spike in new issues
   - Note the top error message and affected endpoint

2. **Check Cloudflare Analytics** → Workers → my-api-production
   - Which routes are failing?
   - What is the error distribution (5xx vs 4xx)?

3. **Check recent deployments** → GitHub Actions → Was there a deploy in last 30 min?
   - If yes → rollback immediately: `npx wrangler rollback --env production`

## Common Causes & Fixes

### Database connection failures
**Symptoms**: `D1_ERROR` in Sentry, `/health` endpoint shows `database: fail`
**Fix**:
```bash
# Check D1 status
npx wrangler d1 execute DB --command "SELECT 1" --env production
# If migration was recently applied, check for schema issues
npx wrangler d1 migrations list DB --env production
```

### Auth service down
**Symptoms**: All requests returning 401, `JWT verification failed` errors
**Fix**: Check `JWT_SECRET` secret is set: `npx wrangler secret list --env production`

### Rate limit from external API
**Symptoms**: `429` errors from upstream, affecting AI features
**Fix**: Check AI Gateway dashboard; enable request queuing

## Escalation

- After 15min unresolved → page engineering lead
- After 30min unresolved → notify all hands in #incidents Slack
- Customer communication template: [Status page template](./comms/incident-template.md)

## Post-Incident

File a post-mortem within 48 hours using [this template](./postmortem-template.md).
```

---

## PART 6 — DOCUMENTATION PRINCIPLES

```
Writing for developers:
  ✓ Show code before explaining it
  ✓ Use the simplest example that demonstrates the concept
  ✓ Be explicit about prerequisites and assumptions
  ✓ Define acronyms and domain-specific terms on first use
  ✓ Link to relevant sections rather than repeating content
  ✓ Include expected output for every code example

Structure for scannability:
  ✓ Lead with the most important information (inverted pyramid)
  ✓ Use headers for navigation, not decoration
  ✓ Bullet points for lists of 3+ items
  ✓ Tables for comparison and configuration reference
  ✓ Code blocks for any command or code (even single lines)
  ✓ Bold key terms on first use

What to avoid:
  ✗ "Simply" / "just" / "easily" (implies user is dumb if they struggle)
  ✗ "As you can see" (if they could see, you wouldn't need to say it)
  ✗ Passive voice ("the config is set" → "set the config")
  ✗ Wall of text without headers
  ✗ Screenshots without alt text
  ✗ Outdated screenshots (prefer code examples)
  ✗ Tutorial without an end state (what does success look like?)

Maintenance rules:
  □ Docs live in the repo — reviewed in the same PR as code changes
  □ Changelog updated every release (no exceptions)
  □ OpenAPI spec generated from code (not hand-written)
  □ Broken links checked in CI
```

---

> **Core Principle**: Documentation is the product you ship to developers.
> Bad docs force support tickets, generate confusion, and signal that the team
> doesn't care. Write docs you'd want to read at 11pm when you're stuck.
