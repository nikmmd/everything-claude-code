---
name: coding-standards
description: Universal coding standards, best practices, and patterns for Python, Go, TypeScript, Terraform, JavaScript, React, and Node.js development.
---

# Coding Standards & Best Practices

Universal coding standards applicable across all projects.

## Code Quality Principles

### 1. Readability First

- Code is read more than written
- Clear variable and function names
- Self-documenting code preferred over comments
- Consistent formatting

### 2. KISS (Keep It Simple, Stupid)

- Simplest solution that works
- Avoid over-engineering
- No premature optimization
- Easy to understand > clever code

### 3. DRY (Don't Repeat Yourself)

- Extract common logic into functions
- Create reusable components
- Share utilities across modules
- Avoid copy-paste programming

### 4. YAGNI (You Aren't Gonna Need It)

- Don't build features before they're needed
- Avoid speculative generality
- Add complexity only when required
- Start simple, refactor when needed

### 5. Avoid Deep Nesting & Over-Abstraction (Lasagna Code)

- Excessive function extraction forces readers to jump through 3-4 layers to understand simple logic.

```python
# ❌ BAD: Over-extracted - requires jumping through 4 functions
def process_user(user):
    return validate_and_transform(user)

def validate_and_transform(user):
    validated = validate_user(user)
    return transform_user(validated)

def validate_user(user):
    return check_user_fields(user)

def check_user_fields(user):
    if not user.get('email'):
        raise ValueError('Email required')
    return user

# ✅ GOOD: Single readable function
def process_user(user):
    if not user.get('email'):
        raise ValueError('Email required')
    return {
        'id': user['id'],
        'email': user['email'].lower(),
        'name': user.get('name', 'Anonymous')
    }
```

```typescript
// ❌ BAD: Tiny functions that should be inline
function getFirstName(user: User) {
  return user.name.split(" ")[0];
}

function formatGreeting(name: string) {
  return `Hello, ${name}`;
}

function greetUser(user: User) {
  return formatGreeting(getFirstName(user));
}

// ✅ GOOD: Simple inline logic
function greetUser(user: User) {
  const firstName = user.name.split(" ")[0];
  return `Hello, ${firstName}`;
}
```

**When TO extract a function:**

- Logic is reused 3+ times across different files
- Function is 50+ lines and has clear sub-responsibilities
- Logic needs independent unit testing
- Function represents a domain concept (e.g., `calculateTax`)

**When NOT to extract:**

- Logic is used once or twice
- Function would be < 5 lines
- Extraction just adds indirection without clarity
- You're extracting to "make it look clean" rather than for reuse

## TypeScript/JavaScript Standards

### Variable Naming

```typescript
// ✅ GOOD: Descriptive names
const marketSearchQuery = "election";
const isUserAuthenticated = true;
const totalRevenue = 1000;

// ❌ BAD: Unclear names
const q = "election";
const flag = true;
const x = 1000;
```

### Function Naming

```typescript
// ✅ GOOD: Verb-noun pattern
async function fetchMarketData(marketId: string) {}
function calculateSimilarity(a: number[], b: number[]) {}
function isValidEmail(email: string): boolean {}

// ❌ BAD: Unclear or noun-only
async function market(id: string) {}
function similarity(a, b) {}
function email(e) {}
```

### Immutability Pattern (CRITICAL)

```typescript
// ✅ ALWAYS use spread operator
const updatedUser = {
  ...user,
  name: "New Name",
};

const updatedArray = [...items, newItem];

// ❌ NEVER mutate directly
user.name = "New Name"; // BAD
items.push(newItem); // BAD
```

### Error Handling

```typescript
// ✅ GOOD: Comprehensive error handling
async function fetchData(url: string) {
  try {
    const response = await fetch(url);

    if (!response.ok) {
      throw new Error(`HTTP ${response.status}: ${response.statusText}`);
    }

    return await response.json();
  } catch (error) {
    console.error("Fetch failed:", error);
    throw new Error("Failed to fetch data");
  }
}

// ❌ BAD: No error handling
async function fetchData(url) {
  const response = await fetch(url);
  return response.json();
}
```

### Async/Await Best Practices

```typescript
// ✅ GOOD: Parallel execution when possible
const [users, markets, stats] = await Promise.all([
  fetchUsers(),
  fetchMarkets(),
  fetchStats(),
]);

// ❌ BAD: Sequential when unnecessary
const users = await fetchUsers();
const markets = await fetchMarkets();
const stats = await fetchStats();
```

### Type Safety

```typescript
// ✅ GOOD: Proper types
interface Market {
  id: string;
  name: string;
  status: "active" | "resolved" | "closed";
  created_at: Date;
}

function getMarket(id: string): Promise<Market> {
  // Implementation
}

// ❌ BAD: Using 'any'
function getMarket(id: any): Promise<any> {
  // Implementation
}
```

## React Best Practices

### Component Structure

```typescript
// ✅ GOOD: Functional component with types
interface ButtonProps {
  children: React.ReactNode
  onClick: () => void
  disabled?: boolean
  variant?: 'primary' | 'secondary'
}

export function Button({
  children,
  onClick,
  disabled = false,
  variant = 'primary'
}: ButtonProps) {
  return (
    <button
      onClick={onClick}
      disabled={disabled}
      className={`btn btn-${variant}`}
    >
      {children}
    </button>
  )
}

// ❌ BAD: No types, unclear structure
export function Button(props) {
  return <button onClick={props.onClick}>{props.children}</button>
}
```

### Custom Hooks

```typescript
// ✅ GOOD: Reusable custom hook
export function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState<T>(value);

  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    return () => clearTimeout(handler);
  }, [value, delay]);

  return debouncedValue;
}

// Usage
const debouncedQuery = useDebounce(searchQuery, 500);
```

### State Management

```typescript
// ✅ GOOD: Proper state updates
const [count, setCount] = useState(0);

// Functional update for state based on previous state
setCount((prev) => prev + 1);

// ❌ BAD: Direct state reference
setCount(count + 1); // Can be stale in async scenarios
```

### Conditional Rendering

```typescript
// ✅ GOOD: Clear conditional rendering
{isLoading && <Spinner />}
{error && <ErrorMessage error={error} />}
{data && <DataDisplay data={data} />}

// ❌ BAD: Ternary hell
{isLoading ? <Spinner /> : error ? <ErrorMessage error={error} /> : data ? <DataDisplay data={data} /> : null}
```

## API Design Standards

### REST API Conventions

```
GET    /api/markets              # List all markets
GET    /api/markets/:id          # Get specific market
POST   /api/markets              # Create new market
PUT    /api/markets/:id          # Update market (full)
PATCH  /api/markets/:id          # Update market (partial)
DELETE /api/markets/:id          # Delete market

# Query parameters for filtering
GET /api/markets?status=active&limit=10&offset=0
```

### Response Format

```typescript
// ✅ GOOD: Consistent response structure
interface ApiResponse<T> {
  success: boolean;
  data?: T;
  error?: string;
  meta?: {
    total: number;
    page: number;
    limit: number;
  };
}

// Success response
return NextResponse.json({
  success: true,
  data: markets,
  meta: { total: 100, page: 1, limit: 10 },
});

// Error response
return NextResponse.json(
  {
    success: false,
    error: "Invalid request",
  },
  { status: 400 },
);
```

### Input Validation

```typescript
import { z } from "zod";

// ✅ GOOD: Schema validation
const CreateMarketSchema = z.object({
  name: z.string().min(1).max(200),
  description: z.string().min(1).max(2000),
  endDate: z.string().datetime(),
  categories: z.array(z.string()).min(1),
});

export async function POST(request: Request) {
  const body = await request.json();

  try {
    const validated = CreateMarketSchema.parse(body);
    // Proceed with validated data
  } catch (error) {
    if (error instanceof z.ZodError) {
      return NextResponse.json(
        {
          success: false,
          error: "Validation failed",
          details: error.errors,
        },
        { status: 400 },
      );
    }
  }
}
```

## File Organization

### Project Structure

```
src/
├── app/                    # Next.js App Router
│   ├── api/               # API routes
│   ├── markets/           # Market pages
│   └── (auth)/           # Auth pages (route groups)
├── components/            # React components
│   ├── ui/               # Generic UI components
│   ├── forms/            # Form components
│   └── layouts/          # Layout components
├── hooks/                # Custom React hooks
├── lib/                  # Utilities and configs
│   ├── api/             # API clients
│   ├── utils/           # Helper functions
│   └── constants/       # Constants
├── types/                # TypeScript types
└── styles/              # Global styles
```

### File Naming

```
components/Button.tsx          # PascalCase for components
hooks/useAuth.ts              # camelCase with 'use' prefix
lib/formatDate.ts             # camelCase for utilities
types/market.types.ts         # camelCase with .types suffix
```

## Comments & Documentation

### When to Comment

```typescript
// ✅ GOOD: Explain WHY, not WHAT
// Use exponential backoff to avoid overwhelming the API during outages
const delay = Math.min(1000 * Math.pow(2, retryCount), 30000);

// Deliberately using mutation here for performance with large arrays
items.push(newItem);

// ❌ BAD: Stating the obvious
// Increment counter by 1
count++;

// Set name to user's name
name = user.name;
```

### JSDoc for Public APIs

````typescript
/**
 * Searches markets using semantic similarity.
 *
 * @param query - Natural language search query
 * @param limit - Maximum number of results (default: 10)
 * @returns Array of markets sorted by similarity score
 * @throws {Error} If OpenAI API fails or Redis unavailable
 *
 * @example
 * ```typescript
 * const results = await searchMarkets('election', 5)
 * console.log(results[0].name) // "Trump vs Biden"
 * ```
 */
export async function searchMarkets(
  query: string,
  limit: number = 10,
): Promise<Market[]> {
  // Implementation
}
````

## Performance Best Practices

### Memoization

```typescript
import { useMemo, useCallback } from "react";

// ✅ GOOD: Memoize expensive computations
const sortedMarkets = useMemo(() => {
  return markets.sort((a, b) => b.volume - a.volume);
}, [markets]);

// ✅ GOOD: Memoize callbacks
const handleSearch = useCallback((query: string) => {
  setSearchQuery(query);
}, []);
```

### Lazy Loading

```typescript
import { lazy, Suspense } from 'react'

// ✅ GOOD: Lazy load heavy components
const HeavyChart = lazy(() => import('./HeavyChart'))

export function Dashboard() {
  return (
    <Suspense fallback={<Spinner />}>
      <HeavyChart />
    </Suspense>
  )
}
```

### Database Queries

```typescript
// ✅ GOOD: Select only needed columns (parameterized)
const { rows } = await db.query<Market>(
  "SELECT id, name, status FROM markets LIMIT $1",
  [10],
);

// ❌ BAD: Select everything
const { rows } = await db.query("SELECT * FROM markets");

// ❌ BAD: String concatenation (SQL injection risk!)
const { rows } = await db.query(`SELECT * FROM markets WHERE id = '${id}'`);
```

## Testing Standards

### Test Structure (AAA Pattern)

```typescript
test("calculates similarity correctly", () => {
  // Arrange
  const vector1 = [1, 0, 0];
  const vector2 = [0, 1, 0];

  // Act
  const similarity = calculateCosineSimilarity(vector1, vector2);

  // Assert
  expect(similarity).toBe(0);
});
```

### Test Naming

```typescript
// ✅ GOOD: Descriptive test names
test("returns empty array when no markets match query", () => {});
test("throws error when OpenAI API key is missing", () => {});
test("falls back to substring search when Redis unavailable", () => {});

// ❌ BAD: Vague test names
test("works", () => {});
test("test search", () => {});
```

## Code Smell Detection

Watch for these anti-patterns:

### 1. Long Functions

```typescript
// ❌ BAD: Function > 50 lines
function processMarketData() {
  // 100 lines of code
}

// ✅ GOOD: Split into smaller functions
function processMarketData() {
  const validated = validateData();
  const transformed = transformData(validated);
  return saveData(transformed);
}
```

### 2. Deep Nesting

```typescript
// ❌ BAD: 5+ levels of nesting
if (user) {
  if (user.isAdmin) {
    if (market) {
      if (market.isActive) {
        if (hasPermission) {
          // Do something
        }
      }
    }
  }
}

// ✅ GOOD: Early returns
if (!user) return;
if (!user.isAdmin) return;
if (!market) return;
if (!market.isActive) return;
if (!hasPermission) return;

// Do something
```

### 3. Magic Numbers

```typescript
// ❌ BAD: Unexplained numbers
if (retryCount > 3) {
}
setTimeout(callback, 500);

// ✅ GOOD: Named constants
const MAX_RETRIES = 3;
const DEBOUNCE_DELAY_MS = 500;

if (retryCount > MAX_RETRIES) {
}
setTimeout(callback, DEBOUNCE_DELAY_MS);
```

**Remember**: Code quality is not negotiable. Clear, maintainable code enables rapid development and confident refactoring.

---

## Python Standards (FastAPI/Pydantic)

### Naming Conventions

```python
# ✅ GOOD: PEP 8 style (snake_case)
market_search_query = "election"
is_user_authenticated = True
total_revenue = 1000

# ✅ GOOD: Classes use PascalCase
class MarketRepository:
    pass

class UserNotFoundError(Exception):
    pass

# ❌ BAD: camelCase or unclear names
marketSearchQuery = "election"  # BAD
q = "election"  # BAD
```

### Type Hints (Mandatory)

```python
from typing import TypeVar, Generic
from pydantic import BaseModel

# ✅ GOOD: Full type annotations
def fetch_market_data(market_id: str) -> Market | None:
    pass

async def search_markets(query: str, limit: int = 10) -> list[Market]:
    pass

def is_valid_email(email: str) -> bool:
    pass

# ✅ GOOD: Generic types
T = TypeVar("T")

class ApiResponse(BaseModel, Generic[T]):
    success: bool
    data: T | None = None
    error: str | None = None

# ❌ BAD: No type hints
def get_market(id):  # BAD - no types
    pass
```

### Pydantic Models

```python
from pydantic import BaseModel, Field, field_validator
from datetime import datetime

# ✅ GOOD: Pydantic model with validation
class CreateMarketRequest(BaseModel):
    name: str = Field(..., min_length=1, max_length=200)
    description: str = Field(..., min_length=1, max_length=2000)
    end_date: datetime
    categories: list[str] = Field(..., min_length=1)

    @field_validator("categories")
    @classmethod
    def validate_categories(cls, v: list[str]) -> list[str]:
        if len(v) > 5:
            raise ValueError("Maximum 5 categories allowed")
        return v

# ✅ GOOD: Settings with environment variables
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    database_url: str
    redis_url: str
    openai_api_key: str
    debug: bool = False

    class Config:
        env_file = ".env"
```

### FastAPI Patterns

```python
from fastapi import APIRouter, Depends, HTTPException, status
from typing import Annotated

router = APIRouter(prefix="/api/markets", tags=["markets"])

# ✅ GOOD: Dependency injection
async def get_db() -> AsyncGenerator[Database, None]:
    async with Database() as db:
        yield db

DbDep = Annotated[Database, Depends(get_db)]

@router.get("/{market_id}")
async def get_market(
    market_id: str,
    db: DbDep
) -> ApiResponse[Market]:
    market = await db.get_market(market_id)
    if not market:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail=f"Market {market_id} not found"
        )
    return ApiResponse(success=True, data=market)

# ✅ GOOD: Response model validation
@router.post("/", response_model=ApiResponse[Market])
async def create_market(
    request: CreateMarketRequest,
    db: DbDep
) -> ApiResponse[Market]:
    market = await db.create_market(request)
    return ApiResponse(success=True, data=market)
```

### Error Handling

```python
import logging

logger = logging.getLogger(__name__)

# ✅ GOOD: Proper exception handling with logging
async def fetch_data(url: str) -> dict:
    try:
        async with httpx.AsyncClient() as client:
            response = await client.get(url)
            response.raise_for_status()
            return response.json()
    except httpx.HTTPStatusError as e:
        logger.error(f"HTTP error fetching {url}: {e.response.status_code}")
        raise
    except httpx.RequestError as e:
        logger.error(f"Request error fetching {url}: {e}")
        raise

# ❌ BAD: Bare except clause
try:
    data = await fetch_data(url)
except:  # BAD - catches everything including KeyboardInterrupt
    pass

# ❌ BAD: Using print instead of logging
print(f"Error: {e}")  # BAD - use logger.error()
```

### Async/Await Patterns

```python
import asyncio

# ✅ GOOD: Parallel execution with gather
async def fetch_all_data() -> tuple[list[User], list[Market], Stats]:
    users, markets, stats = await asyncio.gather(
        fetch_users(),
        fetch_markets(),
        fetch_stats()
    )
    return users, markets, stats

# ❌ BAD: Sequential when parallel is possible
async def fetch_all_data_slow():
    users = await fetch_users()
    markets = await fetch_markets()  # Waits unnecessarily
    stats = await fetch_stats()  # Waits unnecessarily
    return users, markets, stats
```

### Project Structure

```
project/
├── src/
│   ├── __init__.py
│   ├── main.py              # FastAPI app entry
│   ├── config.py            # Pydantic Settings
│   ├── models/              # Pydantic models
│   │   ├── __init__.py
│   │   ├── market.py
│   │   └── user.py
│   ├── routers/             # API route handlers
│   │   ├── __init__.py
│   │   ├── markets.py
│   │   └── users.py
│   ├── services/            # Business logic
│   ├── repositories/        # Data access layer
│   └── utils/               # Helper functions
├── tests/
│   ├── conftest.py          # pytest fixtures
│   ├── test_markets.py
│   └── e2e/
├── pyproject.toml
└── poetry.lock
```

### Python Tools

| Tool   | Purpose              | Command                         |
| ------ | -------------------- | ------------------------------- |
| ruff   | Linting + Formatting | `ruff check . && ruff format .` |
| mypy   | Type checking        | `mypy . --strict`               |
| pytest | Testing              | `pytest tests/ -v`              |
| bandit | Security scanning    | `bandit -r src/`                |

---

## Go Standards

### Naming Conventions

```go
// ✅ GOOD: mixedCaps (not snake_case)
var marketSearchQuery = "election"
var isAuthenticated = true

// ✅ GOOD: Exported (public) starts with uppercase
type MarketRepository interface {
    FindByID(id string) (*Market, error)
}

// ✅ GOOD: Unexported (private) starts with lowercase
type marketCache struct {
    data map[string]*Market
}

// ✅ GOOD: Acronyms are all caps
var httpClient *http.Client
var userID string
var apiURL string

// ❌ BAD: snake_case or inconsistent caps
var market_search_query = "election"  // BAD
var UserId string  // BAD - should be UserID
```

### Error Handling (CRITICAL)

```go
import (
    "errors"
    "fmt"
)

// Define sentinel errors
var (
    ErrNotFound     = errors.New("not found")
    ErrUnauthorized = errors.New("unauthorized")
)

// ✅ GOOD: Error wrapping with %w
func GetMarket(id string) (*Market, error) {
    market, err := repo.FindByID(id)
    if err != nil {
        return nil, fmt.Errorf("get market %s: %w", id, err)
    }
    return market, nil
}

// ✅ GOOD: Check errors with errors.Is() (preferred over ==)
func HandleMarket(id string) error {
    market, err := GetMarket(id)
    if errors.Is(err, ErrNotFound) {
        return fmt.Errorf("market %s does not exist", id)
    }
    if err != nil {
        return err
    }
    // use market
    return nil
}

// ✅ GOOD: Unwrap typed errors with errors.As()
type ValidationError struct {
    Field   string
    Message string
}

func (e *ValidationError) Error() string {
    return fmt.Sprintf("%s: %s", e.Field, e.Message)
}

func HandleRequest(r *http.Request) error {
    err := validateRequest(r)

    var validationErr *ValidationError
    if errors.As(err, &validationErr) {
        return fmt.Errorf("validation failed on %s: %s",
            validationErr.Field, validationErr.Message)
    }
    return err
}

// ❌ BAD: Ignoring errors
result, _ := SomeFunction()  // BAD - error ignored

// ❌ BAD: Direct comparison (doesn't work with wrapped errors)
if err == ErrNotFound {  // BAD - use errors.Is()
    // ...
}
```

### Interface Patterns

```go
// ✅ GOOD: Small, focused interfaces
type Reader interface {
    Read(p []byte) (n int, err error)
}

type Writer interface {
    Write(p []byte) (n int, err error)
}

// ✅ GOOD: Interface composition
type ReadWriter interface {
    Reader
    Writer
}

// ✅ GOOD: Accept interfaces, return structs
type MarketRepository interface {
    FindByID(ctx context.Context, id string) (*Market, error)
    Save(ctx context.Context, market *Market) error
}

func NewMarketService(repo MarketRepository) *MarketService {
    return &MarketService{repo: repo}
}

// ❌ BAD: Large interfaces
type DoEverything interface {
    Read() error
    Write() error
    Delete() error
    Update() error
    List() error
    Search() error
    // Too many methods - split into smaller interfaces
}
```

### HTTP API Patterns (Chi Router)

```go
import (
    "encoding/json"
    "net/http"

    "github.com/go-chi/chi/v5"
    "github.com/go-chi/chi/v5/middleware"
)

func NewRouter(svc *MarketService) http.Handler {
    r := chi.NewRouter()

    // Middleware
    r.Use(middleware.Logger)
    r.Use(middleware.Recoverer)
    r.Use(middleware.RequestID)

    // Routes
    r.Route("/api/markets", func(r chi.Router) {
        r.Get("/", listMarkets(svc))
        r.Post("/", createMarket(svc))
        r.Get("/{marketID}", getMarket(svc))
    })

    return r
}

func getMarket(svc *MarketService) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        ctx := r.Context()
        marketID := chi.URLParam(r, "marketID")

        market, err := svc.GetMarket(ctx, marketID)
        if errors.Is(err, ErrNotFound) {
            http.Error(w, "market not found", http.StatusNotFound)
            return
        }
        if err != nil {
            http.Error(w, "internal error", http.StatusInternalServerError)
            return
        }

        w.Header().Set("Content-Type", "application/json")
        json.NewEncoder(w).Encode(market)
    }
}
```

### Graceful Shutdown

```go
func main() {
    srv := &http.Server{
        Addr:    ":8080",
        Handler: NewRouter(svc),
    }

    // Start server in goroutine
    go func() {
        if err := srv.ListenAndServe(); err != http.ErrServerClosed {
            log.Fatalf("server error: %v", err)
        }
    }()

    // Wait for interrupt signal
    quit := make(chan os.Signal, 1)
    signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
    <-quit

    // Graceful shutdown with timeout
    ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
    defer cancel()

    if err := srv.Shutdown(ctx); err != nil {
        log.Fatalf("shutdown error: %v", err)
    }
}
```

### Concurrency Patterns

```go
// ✅ GOOD: Context propagation
func ProcessRequest(ctx context.Context, id string) error {
    // Pass context to downstream calls
    data, err := fetchData(ctx, id)
    if err != nil {
        return err
    }
    return saveData(ctx, data)
}

// ✅ GOOD: Worker pool
func processItems(ctx context.Context, items []Item, workers int) error {
    g, ctx := errgroup.WithContext(ctx)
    itemsCh := make(chan Item)

    // Start workers
    for i := 0; i < workers; i++ {
        g.Go(func() error {
            for item := range itemsCh {
                if err := processItem(ctx, item); err != nil {
                    return err
                }
            }
            return nil
        })
    }

    // Send items
    g.Go(func() error {
        defer close(itemsCh)
        for _, item := range items {
            select {
            case itemsCh <- item:
            case <-ctx.Done():
                return ctx.Err()
            }
        }
        return nil
    })

    return g.Wait()
}
```

### Project Structure

```
project/
├── cmd/
│   ├── api/                 # HTTP server entry point
│   │   └── main.go
│   └── cli/                 # CLI tool entry point
│       └── main.go
├── internal/                # Private packages
│   ├── handlers/            # HTTP handlers
│   ├── services/            # Business logic
│   ├── repositories/        # Data access
│   ├── models/              # Domain models
│   └── config/              # Configuration
├── pkg/                     # Public packages (if any)
├── go.mod
└── go.sum
```

### Go Tools

| Tool          | Purpose               | Command             |
| ------------- | --------------------- | ------------------- |
| gofmt         | Formatting            | `gofmt -w .`        |
| go vet        | Static analysis       | `go vet ./...`      |
| golangci-lint | Comprehensive linting | `golangci-lint run` |
| gosec         | Security scanning     | `gosec ./...`       |

---

## Terraform Standards

### Naming Conventions

```hcl
# ✅ GOOD: snake_case for resources and variables
resource "aws_instance" "web_server" {
  ami           = var.ami_id
  instance_type = var.instance_type
}

variable "environment_name" {
  type        = string
  description = "Environment name (dev, staging, prod)"
}

# ✅ GOOD: Descriptive resource names
resource "aws_security_group" "api_server_sg" {
  name        = "${var.project_name}-api-server-sg"
  description = "Security group for API server instances"
}

# ❌ BAD: Generic or unclear names
resource "aws_instance" "server" {  # BAD - too generic
  # ...
}

variable "x" {  # BAD - unclear
  type = string
}
```

### Variable Definitions

```hcl
# ✅ GOOD: Variables with descriptions and validation
variable "instance_type" {
  type        = string
  description = "EC2 instance type for the web server"
  default     = "t3.micro"

  validation {
    condition     = can(regex("^t3\\.", var.instance_type))
    error_message = "Instance type must be from t3 family."
  }
}

variable "environment" {
  type        = string
  description = "Deployment environment"

  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod."
  }
}

# ✅ GOOD: Sensitive variables marked
variable "database_password" {
  type        = string
  description = "Password for the RDS database"
  sensitive   = true
}

# ❌ BAD: No description or validation
variable "instance_type" {
  type = string
}
```

### Module Structure

```hcl
# modules/vpc/main.tf
resource "aws_vpc" "main" {
  cidr_block           = var.cidr_block
  enable_dns_hostnames = true
  enable_dns_support   = true

  tags = merge(var.tags, {
    Name = "${var.project_name}-vpc"
  })
}

# modules/vpc/variables.tf
variable "cidr_block" {
  type        = string
  description = "CIDR block for the VPC"
  default     = "10.0.0.0/16"
}

variable "project_name" {
  type        = string
  description = "Project name for resource naming"
}

variable "tags" {
  type        = map(string)
  description = "Tags to apply to all resources"
  default     = {}
}

# modules/vpc/outputs.tf
output "vpc_id" {
  value       = aws_vpc.main.id
  description = "The ID of the VPC"
}

output "vpc_cidr" {
  value       = aws_vpc.main.cidr_block
  description = "The CIDR block of the VPC"
}
```

### Security Patterns (CRITICAL)

```hcl
# ✅ GOOD: Least privilege IAM
resource "aws_iam_role" "lambda_role" {
  name = "${var.project_name}-lambda-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Action = "sts:AssumeRole"
      Effect = "Allow"
      Principal = {
        Service = "lambda.amazonaws.com"
      }
    }]
  })
}

resource "aws_iam_role_policy" "lambda_policy" {
  name = "${var.project_name}-lambda-policy"
  role = aws_iam_role.lambda_role.id

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect   = "Allow"
      Action   = ["s3:GetObject", "s3:PutObject"]
      Resource = "${aws_s3_bucket.data.arn}/*"  # Specific bucket
    }]
  })
}

# ❌ BAD: Overly permissive
resource "aws_iam_role_policy" "bad_policy" {
  policy = jsonencode({
    Statement = [{
      Effect   = "Allow"
      Action   = "*"          # BAD - too permissive
      Resource = "*"          # BAD - too permissive
    }]
  })
}

# ✅ GOOD: Restricted security groups
resource "aws_security_group" "api" {
  name = "${var.project_name}-api-sg"

  ingress {
    from_port       = 443
    to_port         = 443
    protocol        = "tcp"
    security_groups = [aws_security_group.alb.id]  # Specific source
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]  # Outbound is usually fine
  }
}

# ❌ BAD: Open to the world
resource "aws_security_group" "bad_sg" {
  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]  # BAD - SSH open to world
  }
}
```

### State Management

```hcl
# backend.tf - Remote state with locking
terraform {
  backend "s3" {
    bucket         = "myproject-terraform-state"
    key            = "environments/prod/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-state-lock"
  }
}

# State locking table
resource "aws_dynamodb_table" "terraform_lock" {
  name         = "terraform-state-lock"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "LockID"

  attribute {
    name = "LockID"
    type = "S"
  }
}
```

### Project Structure

```
infrastructure/
├── modules/
│   ├── vpc/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── outputs.tf
│   │   └── README.md
│   ├── ecs-service/
│   ├── rds/
│   └── lambda/
├── environments/
│   ├── dev/
│   │   ├── main.tf
│   │   ├── backend.tf
│   │   ├── variables.tf
│   │   └── terraform.tfvars
│   ├── staging/
│   └── prod/
└── README.md
```

### Terraform Tools

| Tool               | Purpose           | Command                    |
| ------------------ | ----------------- | -------------------------- |
| terraform fmt      | Formatting        | `terraform fmt -recursive` |
| terraform validate | Syntax validation | `terraform validate`       |
| tflint             | Linting           | `tflint --recursive`       |
| tfsec              | Security scanning | `tfsec .`                  |
| checkov            | Policy compliance | `checkov -d .`             |
