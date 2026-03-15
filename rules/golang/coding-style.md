# Go Coding Style

## Formatting

- Always use `gofmt` and `goimports` — non-negotiable
- Accept interfaces, return concrete types
- Keep interfaces minimal (1-3 methods)
- Define interfaces at point of consumption, not implementation

## Error Handling

```go
// Always wrap errors with context
if err != nil {
    return fmt.Errorf("failed to fetch user %s: %w", userID, err)
}

// Custom error types for domain errors
type NotFoundError struct {
    Resource string
    ID       string
}

func (e *NotFoundError) Error() string {
    return fmt.Sprintf("%s %s not found", e.Resource, e.ID)
}
```

## Naming

- Short, clear names: `srv` not `myServer`, `ctx` not `context`
- Exported names need good package context: `http.Client` not `http.HTTPClient`
- Acronyms all caps: `userID`, `httpURL`

## Struct Design

```go
// Functional options for flexible constructors
type Option func(*Server)

func WithPort(port int) Option {
    return func(s *Server) { s.port = port }
}

func NewServer(opts ...Option) *Server {
    s := &Server{port: 8080} // sensible defaults
    for _, opt := range opts {
        opt(s)
    }
    return s
}
```

## Tools

- **Formatting**: `gofmt`, `goimports`
- **Linting**: `golangci-lint run`
- **Vetting**: `go vet ./...`
- **Static analysis**: `staticcheck ./...`
