# TypeScript Patterns

## API Error Type

```typescript
interface ApiError {
  error: string    // Machine-readable: "validation_error", "not_found"
  message: string  // Human-readable description
}

// Success: return data directly, HTTP status conveys outcome
// Pagination: add meta alongside data
interface PaginatedResponse<T> {
  data: T[]
  meta: { total: number; page: number; limit: number }
}
```

## Custom Hooks

```typescript
export function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState<T>(value)
  useEffect(() => {
    const handler = setTimeout(() => setDebouncedValue(value), delay)
    return () => clearTimeout(handler)
  }, [value, delay])
  return debouncedValue
}
```

## Repository Pattern

```typescript
interface Repository<T> {
  findAll(filters?: Filters): Promise<T[]>
  findById(id: string): Promise<T | null>
  create(data: CreateDto): Promise<T>
  update(id: string, data: UpdateDto): Promise<T>
  delete(id: string): Promise<void>
}
```
