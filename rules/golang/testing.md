# Go Testing

## Run Commands

```bash
go test ./...                          # Run all tests
go test ./... -race                    # With race detector (always use)
go test ./... -cover                   # With coverage
go test ./... -coverprofile=coverage.out  # Coverage report
go tool cover -html=coverage.out       # HTML report
go test ./handlers -run TestGetUser -v # Specific test
go test ./... -bench=. -benchmem       # Benchmarks
```

## Table-Driven Tests

```go
func TestCalculateScore(t *testing.T) {
    tests := []struct {
        name     string
        input    int
        expected int
        wantErr  bool
    }{
        {"positive", 10, 100, false},
        {"zero", 0, 0, false},
        {"negative", -1, 0, true},
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            result, err := CalculateScore(tt.input)
            if tt.wantErr {
                require.Error(t, err)
                return
            }
            require.NoError(t, err)
            assert.Equal(t, tt.expected, result)
        })
    }
}
```

## HTTP Handler Testing

```go
func TestGetUser(t *testing.T) {
    req := httptest.NewRequest(http.MethodGet, "/users/123", nil)
    w := httptest.NewRecorder()

    handler := NewUserHandler(mockStore)
    handler.ServeHTTP(w, req)

    assert.Equal(t, http.StatusOK, w.Code)
}
```

## Key Packages

`testing`, `testify` (assert/require/mock), `httptest`, `testcontainers-go`

Scope: `**/*.go`
