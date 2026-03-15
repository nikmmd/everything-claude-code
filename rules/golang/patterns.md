# Go Patterns

## Functional Options

```go
type Option func(*Server)

func WithPort(port int) Option {
    return func(s *Server) { s.port = port }
}

func NewServer(opts ...Option) *Server {
    s := &Server{port: 8080}
    for _, opt := range opts {
        opt(s)
    }
    return s
}
```

## Interface Design

Define interfaces at point of consumption:

```go
// In the consumer package, not the provider
type UserStore interface {
    GetUser(ctx context.Context, id string) (*User, error)
}
```

## Constructor-Based DI

```go
type Service struct {
    store  UserStore
    cache  Cache
    logger *slog.Logger
}

func NewService(store UserStore, cache Cache, logger *slog.Logger) *Service {
    return &Service{store: store, cache: cache, logger: logger}
}
```

## Concurrency

```go
// Use errgroup for parallel operations with error handling
g, ctx := errgroup.WithContext(ctx)

g.Go(func() error { return fetchUsers(ctx) })
g.Go(func() error { return fetchOrders(ctx) })

if err := g.Wait(); err != nil {
    return fmt.Errorf("parallel fetch failed: %w", err)
}
```

## Graceful Shutdown

```go
ctx, stop := signal.NotifyContext(context.Background(), os.Interrupt, syscall.SIGTERM)
defer stop()

go func() { srv.ListenAndServe() }()

<-ctx.Done()
shutdownCtx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
defer cancel()
srv.Shutdown(shutdownCtx)
```
