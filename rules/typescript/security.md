# TypeScript Security

## Secret Management

```typescript
const apiKey = process.env.API_KEY
if (!apiKey) throw new Error('API_KEY not configured')
```

## Input Validation

Always validate with Zod at API boundaries:

```typescript
import { z } from 'zod'
const schema = z.object({ email: z.string().email() })
const validated = schema.parse(input)
```

## XSS Prevention

- Never use `dangerouslySetInnerHTML` without DOMPurify
- React auto-escapes JSX expressions — don't bypass this

## Dependencies

```bash
npm audit          # Check for vulnerabilities
npm audit fix      # Fix automatically
```

For deep analysis, use the **security-reviewer** agent.
