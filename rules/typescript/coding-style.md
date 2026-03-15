# TypeScript Coding Style

## Types and Interfaces

- Explicit typing for public APIs; infer for internal/local
- Use `interface` for extensible shapes, `type` for unions/intersections
- Named interfaces for component props
- Explicit callback typing in component APIs

## Avoiding `any`

- Use `unknown` for external input, then narrow safely
- Use generics for reusable functions
- Prefer `Record<string, unknown>` over `{ [key: string]: any }`

## Immutability

```typescript
// Spread for object updates
const updated = { ...user, name: newName }

// Array methods that return new arrays
const filtered = items.filter(item => item.active)
const mapped = items.map(item => ({ ...item, processed: true }))
```

## Error Handling

```typescript
try {
  const result = await riskyOperation()
  return result
} catch (error) {
  if (error instanceof AppError) {
    throw error
  }
  throw new AppError('Operation failed', { cause: error })
}
```

## Validation

Use Zod for schema-based validation with type inference:

```typescript
import { z } from 'zod'

const UserSchema = z.object({
  email: z.string().email(),
  name: z.string().min(1).max(100),
})

type User = z.infer<typeof UserSchema>
```

## React Components

- Server Components by default, `"use client"` only for interactivity
- Named interfaces for props
- Small, focused components (<200 lines)
- Custom hooks for reusable logic
