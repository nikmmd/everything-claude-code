---
name: security-review
description: Auto-triggered security checklist for authentication, user input handling, secrets, API endpoints, and sensitive features. Delegates deep analysis to the security-reviewer agent.
compatibility: Designed for Claude Code
---

# Security Review

Auto-activates when working on authentication, user input, secrets, API endpoints, or sensitive features. For deep vulnerability analysis, delegate to the **security-reviewer** agent.

## When This Activates

- Implementing authentication or authorization
- Handling user input or file uploads
- Creating or modifying API endpoints
- Working with secrets or credentials
- Implementing payment or sensitive features
- Integrating third-party APIs

## Quick Checklist

Before proceeding with security-sensitive code, verify:

- [ ] No hardcoded secrets — all in environment variables
- [ ] All user inputs validated with schemas (e.g., Zod)
- [ ] Database queries parameterized (no string concatenation)
- [ ] User-provided HTML sanitized (DOMPurify)
- [ ] CSRF protection on state-changing operations
- [ ] Authorization checks before sensitive operations
- [ ] Rate limiting on all endpoints
- [ ] Error messages don't leak internal details
- [ ] No sensitive data in logs
- [ ] Dependencies up to date (`npm audit` clean)

## Key Patterns

### Secrets

```typescript
// NEVER hardcode
const apiKey = process.env.API_KEY
if (!apiKey) throw new Error('API_KEY not configured')
```

### Input Validation

```typescript
import { z } from 'zod'
const schema = z.object({
  email: z.string().email(),
  name: z.string().min(1).max(100),
})
const validated = schema.parse(input)
```

### Parameterized Queries

```typescript
// ALWAYS parameterize
const { rows } = await db.query('SELECT * FROM users WHERE email = $1', [email])
```

### Auth Tokens

```typescript
// httpOnly cookies, NOT localStorage
res.setHeader('Set-Cookie',
  `token=${token}; HttpOnly; Secure; SameSite=Strict; Max-Age=3600`)
```

## Deep Analysis

For comprehensive security review (OWASP Top 10, vulnerability scanning, dependency audit), use the **security-reviewer** agent:

```
Launch Agent: security-reviewer
```

See [references/CHECKLIST.md](references/CHECKLIST.md) for the full pre-deployment security checklist with detailed examples.
