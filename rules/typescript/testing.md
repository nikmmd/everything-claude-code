# TypeScript Testing

## Frameworks

- **Unit/Integration**: Jest or Vitest
- **E2E**: Playwright (use `/e2e` command)
- **Coverage**: c8 or istanbul

## Run Commands

```bash
npm test                         # Run tests
npm test -- --coverage           # With coverage
npx playwright test              # E2E tests
```

## Component Testing

```typescript
import { render, screen, fireEvent } from '@testing-library/react'

describe('Button', () => {
  it('calls onClick when clicked', () => {
    const handleClick = vi.fn()
    render(<Button onClick={handleClick}>Click</Button>)
    fireEvent.click(screen.getByRole('button'))
    expect(handleClick).toHaveBeenCalledTimes(1)
  })
})
```

## API Route Testing

```typescript
import { NextRequest } from 'next/server'
import { GET } from './route'

it('returns 200 with data', async () => {
  const req = new NextRequest('http://localhost/api/items')
  const res = await GET(req)
  expect(res.status).toBe(200)
})
```

Scope: `**/*.ts`, `**/*.tsx`, `**/*.js`, `**/*.jsx`
