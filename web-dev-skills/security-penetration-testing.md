---
name: security-penetration-testing
description: >
  Activates elite application security and defensive security engineering capabilities. Use this
  skill for ANY task involving: security code reviews; OWASP Top 10 vulnerability assessment;
  authentication and authorization hardening; JWT security; SQL injection prevention; XSS
  prevention; CSRF protection; API security; secrets management review; dependency vulnerability
  scanning; rate limiting implementation; input validation hardening; security headers; CORS
  configuration; penetration testing checklists; threat modeling; secure coding patterns; or any
  task where the question is "is this code/system secure?" Trigger when the user says "security
  review", "is this secure", "pen test", "vulnerability", "OWASP", "auth hardening", "JWT",
  "injection", "XSS", "CSRF", "rate limiting", "security headers", or "harden this". Always
  think like an attacker, build like a defender.
---

# Elite Application Security Engineer

You think like an attacker and build like a defender. Security is not a feature added at
the end — it's an architectural property embedded from the first line of code.

---

## PART 1 — OWASP TOP 10 (2021) — FULL CHECKLIST

### A01: Broken Access Control

```typescript
// WRONG — trusting client-supplied IDs without ownership check
app.get('/api/documents/:id', async (c) => {
  const doc = await db.select().from(documents).where(eq(documents.id, c.req.param('id'))).get();
  return c.json(doc);  // Returns ANY document to ANY user!
});

// RIGHT — always verify ownership/permission server-side
app.get('/api/documents/:id', requireAuth(), async (c) => {
  const userId = c.get('userId');
  const workspaceId = c.get('workspaceId');

  const doc = await db.select()
    .from(documents)
    .where(and(
      eq(documents.id, c.req.param('id')),
      eq(documents.workspaceId, workspaceId),  // Tenant isolation
      isNull(documents.deletedAt),
    ))
    .get();

  if (!doc) return c.json({ error: 'NOT_FOUND' }, 404);  // Same response whether missing or forbidden

  // Check user has read access within workspace
  await requireDocumentPermission(userId, doc.id, 'read', c.env);

  return c.json(doc);
});
```

**Checklist:**
```
□ Every endpoint verifies the authenticated user owns/can access the resource
□ Admin endpoints require admin role check (not just auth)
□ Tenant isolation on every multi-tenant query
□ Directory traversal prevented: validate/sanitize path parameters
□ No client-supplied IDs used for privilege escalation
□ IDOR (Insecure Direct Object Reference) tested on every resource endpoint
```

### A02: Cryptographic Failures

```typescript
// Sensitive data encryption at rest
import { createCipheriv, createDecipheriv, randomBytes } from 'crypto';

export function encrypt(plaintext: string, key: Buffer): { iv: string; ciphertext: string } {
  const iv = randomBytes(16);
  const cipher = createCipheriv('aes-256-gcm', key, iv);
  const encrypted = Buffer.concat([cipher.update(plaintext, 'utf8'), cipher.final()]);
  const authTag = cipher.getAuthTag();
  return {
    iv: iv.toString('base64'),
    ciphertext: Buffer.concat([encrypted, authTag]).toString('base64'),
  };
}

export function decrypt(iv: string, ciphertext: string, key: Buffer): string {
  const ivBuffer = Buffer.from(iv, 'base64');
  const ciphertextBuffer = Buffer.from(ciphertext, 'base64');
  const authTag = ciphertextBuffer.slice(-16);
  const data = ciphertextBuffer.slice(0, -16);
  const decipher = createDecipheriv('aes-256-gcm', key, ivBuffer);
  decipher.setAuthTag(authTag);
  return decipher.update(data) + decipher.final('utf8');
}

// Passwords: use argon2id (preferred) or bcrypt cost 12
import { hash, verify } from 'argon2';

export const password = {
  hash: (plaintext: string) => hash(plaintext, {
    type: 2,         // argon2id
    memoryCost: 65536,  // 64MB
    timeCost: 3,
    parallelism: 4,
  }),
  verify: (hash: string, plaintext: string) => verify(hash, plaintext),
};
```

**Checklist:**
```
□ Passwords hashed with argon2id or bcrypt (cost >= 12), never MD5/SHA1
□ Sensitive fields (SSN, PAN, keys) encrypted at rest (AES-256-GCM)
□ HTTPS enforced (HSTS header, no HTTP fallback)
□ Secrets never in git, logs, or error messages
□ JWT signed with RS256 or HS256 with 256-bit+ secret
□ Tokens have appropriate expiry (access: 15min, refresh: 7-30d)
□ Secure + HttpOnly + SameSite=Strict on auth cookies
```

### A03: Injection

```typescript
// SQL Injection — ALWAYS use prepared statements
// WRONG
const user = await db.execute(`SELECT * FROM users WHERE email = '${email}'`);  // INJECTION!

// RIGHT — Drizzle parameterization
const user = await db.select().from(users).where(eq(users.email, email)).get();

// Or raw prepared statement
const user = await env.DB.prepare('SELECT * FROM users WHERE email = ? AND deleted_at IS NULL')
  .bind(email)
  .first();

// Command injection prevention — never pass user input to shell commands
// WRONG
exec(`convert ${userInput} output.pdf`);

// RIGHT — use libraries instead of shell, or validate input strictly
import { execFile } from 'child_process';
const safeFilename = path.basename(userInput).replace(/[^a-zA-Z0-9._-]/g, '');
execFile('convert', [safeFilename, 'output.pdf']);

// LDAP injection prevention — encode special characters
const safeDn = ldapEscape.filter(userInput);

// NoSQL injection (MongoDB)
// WRONG
db.users.find({ email: { $regex: userInput } });  // Operator injection
// RIGHT
db.users.find({ email: String(userInput) });
```

**Checklist:**
```
□ Zero string concatenation in SQL queries
□ All queries use parameterized statements / ORM
□ User input never passed to eval(), exec(), execSync()
□ File paths validated and sandboxed
□ XML/JSON inputs validated against schema before processing
□ Template engines use auto-escaping
```

### A05: Security Misconfiguration

```typescript
// Security headers middleware (Hono)
app.use('*', async (c, next) => {
  await next();

  // Prevent clickjacking
  c.header('X-Frame-Options', 'DENY');
  // Prevent MIME sniffing
  c.header('X-Content-Type-Options', 'nosniff');
  // Enable HSTS
  c.header('Strict-Transport-Security', 'max-age=31536000; includeSubDomains; preload');
  // Content Security Policy
  c.header('Content-Security-Policy', [
    "default-src 'self'",
    "script-src 'self' 'nonce-{NONCE}'",  // Nonce-based inline scripts only
    "style-src 'self' 'unsafe-inline' https://fonts.googleapis.com",
    "img-src 'self' data: https:",
    "font-src 'self' https://fonts.gstatic.com",
    "connect-src 'self' wss://api.myapp.com",
    "frame-ancestors 'none'",
    "base-uri 'self'",
    "form-action 'self'",
  ].join('; '));
  // Referrer policy
  c.header('Referrer-Policy', 'strict-origin-when-cross-origin');
  // Permissions policy
  c.header('Permissions-Policy', 'camera=(), microphone=(), geolocation=()');
  // Remove fingerprinting headers
  c.header('X-Powered-By', '');
});

// CORS — explicit allowlist only
app.use('*', cors({
  origin: (origin) => {
    const allowed = ['https://myapp.com', 'https://www.myapp.com'];
    return allowed.includes(origin) ? origin : '';
  },
  allowMethods: ['GET', 'POST', 'PUT', 'DELETE', 'OPTIONS'],
  allowHeaders: ['Content-Type', 'Authorization', 'X-Request-Id'],
  maxAge: 86400,
  credentials: true,
}));
```

### A07: Identification & Authentication Failures

```typescript
// JWT validation — never skip any check
import * as jose from 'jose';

export async function verifyJWT(token: string, env: Env): Promise<JWTPayload> {
  const secret = new TextEncoder().encode(env.JWT_SECRET);

  const { payload } = await jose.jwtVerify(token, secret, {
    issuer: 'myapp.com',         // Must match
    audience: 'myapp-api',      // Must match
    clockTolerance: 30,          // 30s leeway for clock skew
  });

  // Check token hasn't been revoked (check against KV blocklist)
  const isRevoked = await env.KV.get(`revoked:${payload.jti}`);
  if (isRevoked) throw new Error('Token has been revoked');

  // Validate required claims
  if (!payload.sub || !payload.workspaceId) {
    throw new Error('Invalid token claims');
  }

  return payload;
}

// Session fixation prevention — always rotate session ID on login
export async function loginUser(userId: string, env: Env): Promise<string> {
  // Generate new session ID (never reuse pre-auth session ID)
  const sessionId = crypto.randomUUID();
  const token = await signJWT({ sub: userId, jti: sessionId }, env);

  await env.KV.put(
    `session:${sessionId}`,
    JSON.stringify({ userId, createdAt: new Date().toISOString() }),
    { expirationTtl: 86400 }  // 24h
  );

  return token;
}

// Account lockout after failed attempts
export async function checkLoginAttempts(identifier: string, env: Env): Promise<void> {
  const key = `login_attempts:${identifier}`;
  const attempts = parseInt(await env.KV.get(key) ?? '0');

  if (attempts >= 5) {
    throw new RateLimitError('Account temporarily locked. Try again in 15 minutes.');
  }

  await env.KV.put(key, String(attempts + 1), { expirationTtl: 900 });  // 15min window
}

export async function clearLoginAttempts(identifier: string, env: Env): Promise<void> {
  await env.KV.delete(`login_attempts:${identifier}`);
}
```

### A08: Software & Data Integrity Failures

```typescript
// Webhook signature verification (Stripe, GitHub, etc.)
export async function verifyWebhookSignature(
  body: string,
  signature: string,
  secret: string
): Promise<boolean> {
  const encoder = new TextEncoder();
  const key = await crypto.subtle.importKey(
    'raw',
    encoder.encode(secret),
    { name: 'HMAC', hash: 'SHA-256' },
    false,
    ['sign', 'verify']
  );

  const expectedSig = await crypto.subtle.sign(
    'HMAC',
    key,
    encoder.encode(body)
  );

  const expectedHex = Array.from(new Uint8Array(expectedSig))
    .map(b => b.toString(16).padStart(2, '0'))
    .join('');

  // Timing-safe comparison to prevent timing attacks
  return timingSafeEqual(signature, `sha256=${expectedHex}`);
}

function timingSafeEqual(a: string, b: string): boolean {
  if (a.length !== b.length) return false;
  let result = 0;
  for (let i = 0; i < a.length; i++) {
    result |= a.charCodeAt(i) ^ b.charCodeAt(i);
  }
  return result === 0;
}
```

---

## PART 2 — RATE LIMITING

```typescript
// Multi-tier rate limiting: global + per-user + per-endpoint
export class RateLimiter {
  constructor(private kv: KVNamespace) {}

  async check(
    identifier: string,
    window: number,   // seconds
    limit: number,
    prefix = 'rl'
  ): Promise<{ allowed: boolean; remaining: number; resetAt: number }> {
    const key = `${prefix}:${identifier}`;
    const now = Math.floor(Date.now() / 1000);
    const windowStart = now - window;

    // Simple fixed window counter
    const countStr = await this.kv.get(key);
    const count = countStr ? parseInt(countStr) : 0;

    if (count >= limit) {
      const ttl = await this.kv.getWithMetadata(key);
      return {
        allowed: false,
        remaining: 0,
        resetAt: now + (window - (now % window)),
      };
    }

    await this.kv.put(key, String(count + 1), { expirationTtl: window });

    return {
      allowed: true,
      remaining: limit - count - 1,
      resetAt: now + (window - (now % window)),
    };
  }
}

// Usage in middleware
app.use('/api/*', async (c, next) => {
  const limiter = new RateLimiter(c.env.KV);
  const ip = c.req.header('CF-Connecting-IP') ?? 'unknown';
  const userId = c.get('userId');

  // IP-level rate limit (unauthenticated)
  const ipResult = await limiter.check(`ip:${ip}`, 60, 60);  // 60 req/min per IP
  if (!ipResult.allowed) {
    return c.json({ error: 'RATE_LIMIT_EXCEEDED' }, 429, {
      'Retry-After': String(ipResult.resetAt - Math.floor(Date.now() / 1000)),
      'X-RateLimit-Limit': '60',
      'X-RateLimit-Remaining': '0',
    });
  }

  // User-level rate limit (authenticated)
  if (userId) {
    const userResult = await limiter.check(`user:${userId}`, 60, 300);  // 300 req/min per user
    c.header('X-RateLimit-Limit', '300');
    c.header('X-RateLimit-Remaining', String(userResult.remaining));
    if (!userResult.allowed) return c.json({ error: 'RATE_LIMIT_EXCEEDED' }, 429);
  }

  await next();
});
```

---

## PART 3 — INPUT VALIDATION

```typescript
// Zod validation on every external input
import { z } from 'zod';

// Strict schemas — reject anything unexpected
const CreateUserSchema = z.object({
  email: z.string().email().max(255).toLowerCase().trim(),
  displayName: z.string().min(1).max(100).trim(),
  password: z.string().min(8).max(128)
    .regex(/[A-Z]/, 'Must contain uppercase')
    .regex(/[0-9]/, 'Must contain number'),
}).strict();   // .strict() — rejects unexpected fields

// File upload validation
const UploadSchema = z.object({
  filename: z.string().max(255).regex(/^[a-zA-Z0-9._-]+$/, 'Invalid filename'),
  mimeType: z.enum(['image/jpeg', 'image/png', 'image/webp', 'application/pdf']),
  sizeBytes: z.number().max(10 * 1024 * 1024),  // 10MB max
});

// Path parameter validation — prevent traversal
const IdSchema = z.string().uuid();  // Validates UUID format

app.get('/api/files/:filename', async (c) => {
  // Validate path parameter
  const filename = path.basename(c.req.param('filename'));  // Strips ../ attempts
  const safeName = filename.replace(/[^a-zA-Z0-9._-]/g, '');  // Allowlist chars

  if (safeName !== filename) {
    return c.json({ error: 'INVALID_FILENAME' }, 400);
  }
  // ...
});
```

---

## PART 4 — SECURITY REVIEW CHECKLIST

### Pre-Launch Security Audit

```
Authentication:
□ Passwords hashed with argon2id or bcrypt ≥12
□ JWT properly validated (issuer, audience, expiry, revocation)
□ Session IDs rotated on login (prevent session fixation)
□ Account lockout after 5 failed attempts
□ Email verification before full access
□ Password reset tokens single-use and expire in 1 hour

Authorization:
□ Every endpoint checks resource ownership
□ Tenant isolation on all multi-tenant queries
□ Admin routes protected with role check
□ Service-to-service auth uses separate credentials

Input Validation:
□ All inputs validated with Zod (not just trusted)
□ File uploads validated: type, size, filename
□ Path parameters safe against directory traversal
□ SQL uses prepared statements throughout (no concatenation)
□ HTML output escaped (no dangerouslySetInnerHTML with user content)

Transport & Headers:
□ HTTPS enforced (HSTS header set)
□ Security headers set (CSP, X-Frame-Options, XCTO, Referrer)
□ CORS allowlist is explicit (no wildcards)
□ Sensitive cookies: Secure + HttpOnly + SameSite=Strict

Secrets & Data:
□ No secrets in code, git history, or logs
□ Secrets via environment variables / Cloudflare Secrets
□ PII minimized — only collect what you need
□ Sensitive data encrypted at rest (credit cards, SSNs)
□ Logs sanitized of passwords, tokens, card numbers

Dependencies:
□ npm audit run in CI — no high/critical unpatched vulns
□ Dependencies pinned to exact versions
□ Automatic dependency update PRs (Renovate / Dependabot)

API Security:
□ Rate limiting on all public endpoints
□ Webhook signatures verified before processing
□ Pagination limits enforced (no unbounded queries)
□ Error messages don't leak internal details (stack traces, DB errors)

Infrastructure:
□ Principle of least privilege on all service accounts
□ No unnecessary ports or services exposed
□ Database not publicly accessible (Workers only via binding)
□ Cloudflare WAF enabled on production domains
```

---

## PART 5 — COMMON VULNERABILITY PATTERNS

### XSS Prevention

```tsx
// WRONG — renders unescaped HTML
<div dangerouslySetInnerHTML={{ __html: userContent }} />

// RIGHT — sanitize before rendering
import DOMPurify from 'dompurify';
<div dangerouslySetInnerHTML={{ __html: DOMPurify.sanitize(userContent, {
  ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'a', 'p', 'br'],
  ALLOWED_ATTR: ['href'],
}) }} />

// BETTER — render as plain text
<p>{userContent}</p>  // React auto-escapes
```

### CSRF Protection

```typescript
// Double-submit cookie pattern
app.use('/api/*', async (c, next) => {
  if (['GET', 'HEAD', 'OPTIONS'].includes(c.req.method)) {
    return next();
  }

  const csrfCookie = getCookie(c, 'csrf_token');
  const csrfHeader = c.req.header('X-CSRF-Token');

  if (!csrfCookie || csrfCookie !== csrfHeader) {
    return c.json({ error: 'CSRF_TOKEN_MISMATCH' }, 403);
  }

  await next();
});
```

### Timing Attack Prevention

```typescript
// WRONG — reveals whether user exists by response time difference
async function login(email: string, password: string) {
  const user = await findUser(email);
  if (!user) throw new Error('User not found');  // Fast path — reveals user doesn't exist
  const valid = await verifyPassword(password, user.passwordHash);  // Slow path
  if (!valid) throw new Error('Invalid password');
}

// RIGHT — constant time path
async function login(email: string, password: string) {
  const user = await findUser(email);
  const dummyHash = '$argon2id$...$dummy$hash$for$timing';
  const hashToVerify = user?.passwordHash ?? dummyHash;
  const valid = await argon2.verify(hashToVerify, password);
  if (!user || !valid) throw new Error('Invalid credentials');  // Same error always
}
```

---

> **Core Principle**: Security is not a feature you add — it's a property you maintain.
> Assume breach: design systems that limit blast radius when (not if) something gets
> compromised. Defense in depth, principle of least privilege, fail secure.
