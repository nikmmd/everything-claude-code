# Go Security

## Secret Management

```go
apiKey := os.Getenv("API_KEY")
if apiKey == "" {
    log.Fatal("API_KEY environment variable is required")
}
```

## Static Analysis

```bash
gosec ./...           # Security vulnerability scanning
golangci-lint run     # Includes security linters
```

## Concurrency Safety

```go
// Always use context with timeouts
ctx, cancel := context.WithTimeout(ctx, 5*time.Second)
defer cancel()

result, err := client.DoRequest(ctx)
```

## SQL Safety

```go
// GOOD: parameterized
row := db.QueryRowContext(ctx, "SELECT * FROM users WHERE id = $1", userID)

// BAD: string concatenation
row := db.QueryRowContext(ctx, "SELECT * FROM users WHERE id = " + userID)
```

Scope: `**/*.go`, `**/go.mod`, `**/go.sum`
