---
name: backend-patterns
description: Backend architecture patterns, API design, database optimization, and server-side best practices for Python, Go, Node.js, Express, and Next.js API routes.
---

# Backend Development Patterns

Backend architecture patterns and best practices for scalable server-side applications.

## API Design Patterns

### RESTful API Structure

```typescript
// ✅ Resource-based URLs
GET    /api/markets                 # List resources
GET    /api/markets/:id             # Get single resource
POST   /api/markets                 # Create resource
PUT    /api/markets/:id             # Replace resource
PATCH  /api/markets/:id             # Update resource
DELETE /api/markets/:id             # Delete resource

// ✅ Query parameters for filtering, sorting, pagination
GET /api/markets?status=active&sort=volume&limit=20&offset=0
```

### Repository Pattern

```typescript
// Abstract data access logic
interface MarketRepository {
  findAll(filters?: MarketFilters): Promise<Market[]>;
  findById(id: string): Promise<Market | null>;
  create(data: CreateMarketDto): Promise<Market>;
  update(id: string, data: UpdateMarketDto): Promise<Market>;
  delete(id: string): Promise<void>;
}

class PostgresMarketRepository implements MarketRepository {
  constructor(private db: Pool) {}

  async findAll(filters?: MarketFilters): Promise<Market[]> {
    // Use parameterized queries - NEVER concatenate user input
    const { rows } = await this.db.query<Market>(
      `SELECT id, name, status, created_at FROM markets
       WHERE ($1::text IS NULL OR status = $1)
       ORDER BY created_at DESC
       LIMIT $2`,
      [filters?.status ?? null, filters?.limit ?? 100],
    );
    return rows;
  }

  async findById(id: string): Promise<Market | null> {
    const { rows } = await this.db.query<Market>(
      "SELECT * FROM markets WHERE id = $1",
      [id],
    );
    return rows[0] ?? null;
  }

  // Other methods...
}
```

### Service Layer Pattern

```typescript
// Business logic separated from data access
class MarketService {
  constructor(private marketRepo: MarketRepository) {}

  async searchMarkets(query: string, limit: number = 10): Promise<Market[]> {
    // Business logic
    const embedding = await generateEmbedding(query);
    const results = await this.vectorSearch(embedding, limit);

    // Fetch full data
    const markets = await this.marketRepo.findByIds(results.map((r) => r.id));

    // Sort by similarity
    return markets.sort((a, b) => {
      const scoreA = results.find((r) => r.id === a.id)?.score || 0;
      const scoreB = results.find((r) => r.id === b.id)?.score || 0;
      return scoreA - scoreB;
    });
  }

  private async vectorSearch(embedding: number[], limit: number) {
    // Vector search implementation
  }
}
```

### Middleware Pattern

```typescript
// Request/response processing pipeline
export function withAuth(handler: NextApiHandler): NextApiHandler {
  return async (req, res) => {
    const token = req.headers.authorization?.replace("Bearer ", "");

    if (!token) {
      return res.status(401).json({ error: "Unauthorized" });
    }

    try {
      const user = await verifyToken(token);
      req.user = user;
      return handler(req, res);
    } catch (error) {
      return res.status(401).json({ error: "Invalid token" });
    }
  };
}

// Usage
export default withAuth(async (req, res) => {
  // Handler has access to req.user
});
```

## Database Patterns

### Query Optimization

```typescript
// ✅ GOOD: Select only needed columns (parameterized)
const { rows } = await db.query<Market>(
  `SELECT id, name, status, volume FROM markets
   WHERE status = $1
   ORDER BY volume DESC
   LIMIT $2`,
  ["active", 10],
);

// ❌ BAD: Select everything
const { rows } = await db.query("SELECT * FROM markets");
```

### N+1 Query Prevention

```typescript
// ❌ BAD: N+1 query problem
const markets = await getMarkets();
for (const market of markets) {
  market.creator = await getUser(market.creator_id); // N queries
}

// ✅ GOOD: Batch fetch
const markets = await getMarkets();
const creatorIds = markets.map((m) => m.creator_id);
const creators = await getUsers(creatorIds); // 1 query
const creatorMap = new Map(creators.map((c) => [c.id, c]));

markets.forEach((market) => {
  market.creator = creatorMap.get(market.creator_id);
});
```

### Transaction Pattern

```typescript
async function createMarketWithPosition(
  db: Pool,
  marketData: CreateMarketDto,
  positionData: CreatePositionDto,
) {
  const client = await db.connect();

  try {
    await client.query("BEGIN");

    const {
      rows: [market],
    } = await client.query<Market>(
      `INSERT INTO markets (id, name, description, status)
       VALUES ($1, $2, $3, $4)
       RETURNING *`,
      [marketData.id, marketData.name, marketData.description, "active"],
    );

    await client.query(
      `INSERT INTO positions (market_id, user_id, amount)
       VALUES ($1, $2, $3)`,
      [market.id, positionData.userId, positionData.amount],
    );

    await client.query("COMMIT");
    return market;
  } catch (error) {
    await client.query("ROLLBACK");
    throw error;
  } finally {
    client.release();
  }
}
```

## Caching Strategies

### Redis Caching Layer

```typescript
class CachedMarketRepository implements MarketRepository {
  constructor(
    private baseRepo: MarketRepository,
    private redis: RedisClient,
  ) {}

  async findById(id: string): Promise<Market | null> {
    // Check cache first
    const cached = await this.redis.get(`market:${id}`);

    if (cached) {
      return JSON.parse(cached);
    }

    // Cache miss - fetch from database
    const market = await this.baseRepo.findById(id);

    if (market) {
      // Cache for 5 minutes
      await this.redis.setex(`market:${id}`, 300, JSON.stringify(market));
    }

    return market;
  }

  async invalidateCache(id: string): Promise<void> {
    await this.redis.del(`market:${id}`);
  }
}
```

### Cache-Aside Pattern

```typescript
async function getMarketWithCache(id: string): Promise<Market> {
  const cacheKey = `market:${id}`;

  // Try cache
  const cached = await redis.get(cacheKey);
  if (cached) return JSON.parse(cached);

  // Cache miss - fetch from DB
  const market = await db.markets.findUnique({ where: { id } });

  if (!market) throw new Error("Market not found");

  // Update cache
  await redis.setex(cacheKey, 300, JSON.stringify(market));

  return market;
}
```

## Error Handling Patterns

### Centralized Error Handler

```typescript
class ApiError extends Error {
  constructor(
    public statusCode: number,
    public message: string,
    public isOperational = true,
  ) {
    super(message);
    Object.setPrototypeOf(this, ApiError.prototype);
  }
}

export function errorHandler(error: unknown, req: Request): Response {
  if (error instanceof ApiError) {
    return NextResponse.json(
      {
        success: false,
        error: error.message,
      },
      { status: error.statusCode },
    );
  }

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

  // Log unexpected errors
  console.error("Unexpected error:", error);

  return NextResponse.json(
    {
      success: false,
      error: "Internal server error",
    },
    { status: 500 },
  );
}

// Usage
export async function GET(request: Request) {
  try {
    const data = await fetchData();
    return NextResponse.json({ success: true, data });
  } catch (error) {
    return errorHandler(error, request);
  }
}
```

### Retry with Exponential Backoff

```typescript
async function fetchWithRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3,
): Promise<T> {
  let lastError: Error;

  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      lastError = error as Error;

      if (i < maxRetries - 1) {
        // Exponential backoff: 1s, 2s, 4s
        const delay = Math.pow(2, i) * 1000;
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }
  }

  throw lastError!;
}

// Usage
const data = await fetchWithRetry(() => fetchFromAPI());
```

## Authentication & Authorization

### JWT Token Validation

```typescript
import jwt from "jsonwebtoken";

interface JWTPayload {
  userId: string;
  email: string;
  role: "admin" | "user";
}

export function verifyToken(token: string): JWTPayload {
  try {
    const payload = jwt.verify(token, process.env.JWT_SECRET!) as JWTPayload;
    return payload;
  } catch (error) {
    throw new ApiError(401, "Invalid token");
  }
}

export async function requireAuth(request: Request) {
  const token = request.headers.get("authorization")?.replace("Bearer ", "");

  if (!token) {
    throw new ApiError(401, "Missing authorization token");
  }

  return verifyToken(token);
}

// Usage in API route
export async function GET(request: Request) {
  const user = await requireAuth(request);

  const data = await getDataForUser(user.userId);

  return NextResponse.json({ success: true, data });
}
```

### Role-Based Access Control

```typescript
type Permission = "read" | "write" | "delete" | "admin";

interface User {
  id: string;
  role: "admin" | "moderator" | "user";
}

const rolePermissions: Record<User["role"], Permission[]> = {
  admin: ["read", "write", "delete", "admin"],
  moderator: ["read", "write", "delete"],
  user: ["read", "write"],
};

export function hasPermission(user: User, permission: Permission): boolean {
  return rolePermissions[user.role].includes(permission);
}

export function requirePermission(permission: Permission) {
  return async (request: Request) => {
    const user = await requireAuth(request);

    if (!hasPermission(user, permission)) {
      throw new ApiError(403, "Insufficient permissions");
    }

    return user;
  };
}

// Usage
export const DELETE = requirePermission("delete")(async (request: Request) => {
  // Handler with permission check
});
```

## Rate Limiting

### Simple In-Memory Rate Limiter

```typescript
class RateLimiter {
  private requests = new Map<string, number[]>();

  async checkLimit(
    identifier: string,
    maxRequests: number,
    windowMs: number,
  ): Promise<boolean> {
    const now = Date.now();
    const requests = this.requests.get(identifier) || [];

    // Remove old requests outside window
    const recentRequests = requests.filter((time) => now - time < windowMs);

    if (recentRequests.length >= maxRequests) {
      return false; // Rate limit exceeded
    }

    // Add current request
    recentRequests.push(now);
    this.requests.set(identifier, recentRequests);

    return true;
  }
}

const limiter = new RateLimiter();

export async function GET(request: Request) {
  const ip = request.headers.get("x-forwarded-for") || "unknown";

  const allowed = await limiter.checkLimit(ip, 100, 60000); // 100 req/min

  if (!allowed) {
    return NextResponse.json(
      {
        error: "Rate limit exceeded",
      },
      { status: 429 },
    );
  }

  // Continue with request
}
```

## Background Jobs & Queues

### Simple Queue Pattern

```typescript
class JobQueue<T> {
  private queue: T[] = [];
  private processing = false;

  async add(job: T): Promise<void> {
    this.queue.push(job);

    if (!this.processing) {
      this.process();
    }
  }

  private async process(): Promise<void> {
    this.processing = true;

    while (this.queue.length > 0) {
      const job = this.queue.shift()!;

      try {
        await this.execute(job);
      } catch (error) {
        console.error("Job failed:", error);
      }
    }

    this.processing = false;
  }

  private async execute(job: T): Promise<void> {
    // Job execution logic
  }
}

// Usage for indexing markets
interface IndexJob {
  marketId: string;
}

const indexQueue = new JobQueue<IndexJob>();

export async function POST(request: Request) {
  const { marketId } = await request.json();

  // Add to queue instead of blocking
  await indexQueue.add({ marketId });

  return NextResponse.json({ success: true, message: "Job queued" });
}
```

## Logging & Monitoring

### Structured Logging

```typescript
interface LogContext {
  userId?: string;
  requestId?: string;
  method?: string;
  path?: string;
  [key: string]: unknown;
}

class Logger {
  log(level: "info" | "warn" | "error", message: string, context?: LogContext) {
    const entry = {
      timestamp: new Date().toISOString(),
      level,
      message,
      ...context,
    };

    console.log(JSON.stringify(entry));
  }

  info(message: string, context?: LogContext) {
    this.log("info", message, context);
  }

  warn(message: string, context?: LogContext) {
    this.log("warn", message, context);
  }

  error(message: string, error: Error, context?: LogContext) {
    this.log("error", message, {
      ...context,
      error: error.message,
      stack: error.stack,
    });
  }
}

const logger = new Logger();

// Usage
export async function GET(request: Request) {
  const requestId = crypto.randomUUID();

  logger.info("Fetching markets", {
    requestId,
    method: "GET",
    path: "/api/markets",
  });

  try {
    const markets = await fetchMarkets();
    return NextResponse.json({ success: true, data: markets });
  } catch (error) {
    logger.error("Failed to fetch markets", error as Error, { requestId });
    return NextResponse.json({ error: "Internal error" }, { status: 500 });
  }
}
```

**Remember**: Backend patterns enable scalable, maintainable server-side applications. Choose patterns that fit your complexity level.

---

## Python/FastAPI Backend Patterns

### Router Organization

```python
# src/routers/__init__.py
from fastapi import APIRouter
from .markets import router as markets_router
from .users import router as users_router

api_router = APIRouter(prefix="/api")
api_router.include_router(markets_router)
api_router.include_router(users_router)

# src/routers/markets.py
from fastapi import APIRouter, Depends, HTTPException, status, Query
from typing import Annotated

router = APIRouter(prefix="/markets", tags=["markets"])

@router.get("/")
async def list_markets(
    status: str | None = Query(None, description="Filter by status"),
    limit: int = Query(10, ge=1, le=100),
    offset: int = Query(0, ge=0),
    db: DbDep = Depends(get_db)
) -> ApiResponse[list[Market]]:
    markets = await db.list_markets(status=status, limit=limit, offset=offset)
    total = await db.count_markets(status=status)
    return ApiResponse(
        success=True,
        data=markets,
        meta={"total": total, "limit": limit, "offset": offset}
    )

@router.get("/{market_id}")
async def get_market(market_id: str, db: DbDep) -> ApiResponse[Market]:
    market = await db.get_market(market_id)
    if not market:
        raise HTTPException(status_code=404, detail="Market not found")
    return ApiResponse(success=True, data=market)

@router.post("/", status_code=status.HTTP_201_CREATED)
async def create_market(
    request: CreateMarketRequest,
    db: DbDep,
    current_user: UserDep
) -> ApiResponse[Market]:
    market = await db.create_market(request, creator_id=current_user.id)
    return ApiResponse(success=True, data=market)
```

### Dependency Injection Pattern

```python
from fastapi import Depends
from typing import Annotated, AsyncGenerator
from contextlib import asynccontextmanager

# Database dependency
async def get_db() -> AsyncGenerator[Database, None]:
    db = Database(settings.database_url)
    try:
        await db.connect()
        yield db
    finally:
        await db.disconnect()

DbDep = Annotated[Database, Depends(get_db)]

# Authentication dependency
async def get_current_user(
    token: str = Depends(oauth2_scheme),
    db: DbDep = Depends(get_db)
) -> User:
    try:
        payload = jwt.decode(token, settings.secret_key, algorithms=["HS256"])
        user_id = payload.get("sub")
        if not user_id:
            raise HTTPException(status_code=401, detail="Invalid token")
    except JWTError:
        raise HTTPException(status_code=401, detail="Invalid token")

    user = await db.get_user(user_id)
    if not user:
        raise HTTPException(status_code=401, detail="User not found")
    return user

UserDep = Annotated[User, Depends(get_current_user)]

# Optional auth (for public endpoints with optional features)
async def get_optional_user(
    token: str | None = Depends(oauth2_scheme_optional),
    db: DbDep = Depends(get_db)
) -> User | None:
    if not token:
        return None
    try:
        return await get_current_user(token, db)
    except HTTPException:
        return None

OptionalUserDep = Annotated[User | None, Depends(get_optional_user)]
```

### Background Tasks

```python
from fastapi import BackgroundTasks

@router.post("/markets/{market_id}/index")
async def trigger_indexing(
    market_id: str,
    background_tasks: BackgroundTasks,
    db: DbDep
) -> ApiResponse[dict]:
    market = await db.get_market(market_id)
    if not market:
        raise HTTPException(status_code=404, detail="Market not found")

    # Queue background task
    background_tasks.add_task(index_market, market_id)

    return ApiResponse(success=True, data={"message": "Indexing queued"})

async def index_market(market_id: str) -> None:
    """Background task to index a market for search."""
    async with Database(settings.database_url) as db:
        market = await db.get_market(market_id)
        embedding = await generate_embedding(market.description)
        await db.save_embedding(market_id, embedding)
```

### Error Handling

```python
from fastapi import Request
from fastapi.responses import JSONResponse
from fastapi.exceptions import RequestValidationError
from pydantic import ValidationError
import logging

logger = logging.getLogger(__name__)

class AppException(Exception):
    def __init__(self, status_code: int, detail: str, error_code: str | None = None):
        self.status_code = status_code
        self.detail = detail
        self.error_code = error_code

@app.exception_handler(AppException)
async def app_exception_handler(request: Request, exc: AppException):
    return JSONResponse(
        status_code=exc.status_code,
        content={
            "success": False,
            "error": exc.detail,
            "error_code": exc.error_code
        }
    )

@app.exception_handler(RequestValidationError)
async def validation_exception_handler(request: Request, exc: RequestValidationError):
    return JSONResponse(
        status_code=422,
        content={
            "success": False,
            "error": "Validation error",
            "details": exc.errors()
        }
    )

@app.exception_handler(Exception)
async def global_exception_handler(request: Request, exc: Exception):
    logger.exception(f"Unhandled exception: {exc}")
    return JSONResponse(
        status_code=500,
        content={
            "success": False,
            "error": "Internal server error"
        }
    )
```

### Repository Pattern

```python
from abc import ABC, abstractmethod
from typing import Generic, TypeVar

T = TypeVar("T")

class Repository(ABC, Generic[T]):
    @abstractmethod
    async def find_by_id(self, id: str) -> T | None:
        pass

    @abstractmethod
    async def save(self, entity: T) -> T:
        pass

    @abstractmethod
    async def delete(self, id: str) -> None:
        pass

class PostgresMarketRepository(Repository[Market]):
    def __init__(self, pool: asyncpg.Pool):
        self.pool = pool

    async def find_by_id(self, id: str) -> Market | None:
        async with self.pool.acquire() as conn:
            row = await conn.fetchrow(
                "SELECT * FROM markets WHERE id = $1", id
            )
            return Market(**row) if row else None

    async def save(self, market: Market) -> Market:
        async with self.pool.acquire() as conn:
            await conn.execute(
                """
                INSERT INTO markets (id, name, description, status, created_at)
                VALUES ($1, $2, $3, $4, $5)
                ON CONFLICT (id) DO UPDATE SET
                    name = EXCLUDED.name,
                    description = EXCLUDED.description,
                    status = EXCLUDED.status
                """,
                market.id, market.name, market.description,
                market.status, market.created_at
            )
            return market

    async def delete(self, id: str) -> None:
        async with self.pool.acquire() as conn:
            await conn.execute("DELETE FROM markets WHERE id = $1", id)
```

---

## Go HTTP Backend Patterns

### Router Setup (Chi)

```go
package main

import (
    "net/http"
    "time"

    "github.com/go-chi/chi/v5"
    "github.com/go-chi/chi/v5/middleware"
)

func NewRouter(svc *service.MarketService, auth *auth.Service) http.Handler {
    r := chi.NewRouter()

    // Global middleware
    r.Use(middleware.RequestID)
    r.Use(middleware.RealIP)
    r.Use(middleware.Logger)
    r.Use(middleware.Recoverer)
    r.Use(middleware.Timeout(30 * time.Second))

    // Health check (no auth)
    r.Get("/health", healthHandler)

    // API routes
    r.Route("/api", func(r chi.Router) {
        // Public routes
        r.Get("/markets", listMarketsHandler(svc))
        r.Get("/markets/{marketID}", getMarketHandler(svc))

        // Protected routes
        r.Group(func(r chi.Router) {
            r.Use(auth.Middleware)
            r.Post("/markets", createMarketHandler(svc))
            r.Put("/markets/{marketID}", updateMarketHandler(svc))
            r.Delete("/markets/{marketID}", deleteMarketHandler(svc))
        })
    })

    return r
}
```

### Handler Pattern

```go
package handlers

import (
    "encoding/json"
    "errors"
    "net/http"

    "github.com/go-chi/chi/v5"
)

type Response struct {
    Success bool        `json:"success"`
    Data    interface{} `json:"data,omitempty"`
    Error   string      `json:"error,omitempty"`
    Meta    *Meta       `json:"meta,omitempty"`
}

type Meta struct {
    Total  int `json:"total"`
    Limit  int `json:"limit"`
    Offset int `json:"offset"`
}

func getMarketHandler(svc *service.MarketService) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        ctx := r.Context()
        marketID := chi.URLParam(r, "marketID")

        market, err := svc.GetMarket(ctx, marketID)
        if errors.Is(err, service.ErrNotFound) {
            writeError(w, http.StatusNotFound, "Market not found")
            return
        }
        if err != nil {
            writeError(w, http.StatusInternalServerError, "Failed to get market")
            return
        }

        writeJSON(w, http.StatusOK, Response{Success: true, Data: market})
    }
}

func createMarketHandler(svc *service.MarketService) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        ctx := r.Context()

        // Get user from context (set by auth middleware)
        user, ok := auth.UserFromContext(ctx)
        if !ok {
            writeError(w, http.StatusUnauthorized, "Unauthorized")
            return
        }

        var req CreateMarketRequest
        if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
            writeError(w, http.StatusBadRequest, "Invalid request body")
            return
        }

        if err := req.Validate(); err != nil {
            writeError(w, http.StatusBadRequest, err.Error())
            return
        }

        market, err := svc.CreateMarket(ctx, req, user.ID)
        if err != nil {
            writeError(w, http.StatusInternalServerError, "Failed to create market")
            return
        }

        writeJSON(w, http.StatusCreated, Response{Success: true, Data: market})
    }
}

func writeJSON(w http.ResponseWriter, status int, data interface{}) {
    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(status)
    json.NewEncoder(w).Encode(data)
}

func writeError(w http.ResponseWriter, status int, message string) {
    writeJSON(w, status, Response{Success: false, Error: message})
}
```

### Middleware Pattern

```go
package middleware

import (
    "context"
    "net/http"
    "strings"
)

type contextKey string

const userContextKey contextKey = "user"

func (a *AuthService) Middleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        authHeader := r.Header.Get("Authorization")
        if authHeader == "" {
            http.Error(w, "Unauthorized", http.StatusUnauthorized)
            return
        }

        token := strings.TrimPrefix(authHeader, "Bearer ")
        if token == authHeader {
            http.Error(w, "Invalid authorization header", http.StatusUnauthorized)
            return
        }

        user, err := a.ValidateToken(r.Context(), token)
        if err != nil {
            http.Error(w, "Invalid token", http.StatusUnauthorized)
            return
        }

        ctx := context.WithValue(r.Context(), userContextKey, user)
        next.ServeHTTP(w, r.WithContext(ctx))
    })
}

func UserFromContext(ctx context.Context) (*User, bool) {
    user, ok := ctx.Value(userContextKey).(*User)
    return user, ok
}
```

### Service Layer Pattern

```go
package service

import (
    "context"
    "errors"
    "fmt"
)

var (
    ErrNotFound     = errors.New("not found")
    ErrUnauthorized = errors.New("unauthorized")
    ErrValidation   = errors.New("validation error")
)

type MarketService struct {
    repo   MarketRepository
    cache  Cache
    search SearchService
}

func NewMarketService(repo MarketRepository, cache Cache, search SearchService) *MarketService {
    return &MarketService{
        repo:   repo,
        cache:  cache,
        search: search,
    }
}

func (s *MarketService) GetMarket(ctx context.Context, id string) (*Market, error) {
    // Try cache first
    if market, err := s.cache.Get(ctx, "market:"+id); err == nil {
        return market.(*Market), nil
    }

    // Cache miss - fetch from database
    market, err := s.repo.FindByID(ctx, id)
    if err != nil {
        return nil, fmt.Errorf("get market %s: %w", id, err)
    }
    if market == nil {
        return nil, ErrNotFound
    }

    // Update cache
    _ = s.cache.Set(ctx, "market:"+id, market, 5*time.Minute)

    return market, nil
}

func (s *MarketService) CreateMarket(ctx context.Context, req CreateMarketRequest, userID string) (*Market, error) {
    market := &Market{
        ID:          uuid.New().String(),
        Name:        req.Name,
        Description: req.Description,
        CreatorID:   userID,
        Status:      "active",
        CreatedAt:   time.Now(),
    }

    if err := s.repo.Save(ctx, market); err != nil {
        return nil, fmt.Errorf("save market: %w", err)
    }

    // Index for search (async)
    go func() {
        if err := s.search.Index(context.Background(), market); err != nil {
            log.Printf("failed to index market %s: %v", market.ID, err)
        }
    }()

    return market, nil
}
```

### Repository Interface

```go
package service

import "context"

type MarketRepository interface {
    FindByID(ctx context.Context, id string) (*Market, error)
    FindAll(ctx context.Context, opts FindOptions) ([]*Market, error)
    Save(ctx context.Context, market *Market) error
    Delete(ctx context.Context, id string) error
    Count(ctx context.Context, opts FindOptions) (int, error)
}

type FindOptions struct {
    Status *string
    Limit  int
    Offset int
}

// PostgresMarketRepository implements MarketRepository
type PostgresMarketRepository struct {
    db *sql.DB
}

func NewPostgresMarketRepository(db *sql.DB) *PostgresMarketRepository {
    return &PostgresMarketRepository{db: db}
}

func (r *PostgresMarketRepository) FindByID(ctx context.Context, id string) (*Market, error) {
    row := r.db.QueryRowContext(ctx,
        `SELECT id, name, description, status, creator_id, created_at
         FROM markets WHERE id = $1`, id)

    var m Market
    err := row.Scan(&m.ID, &m.Name, &m.Description, &m.Status, &m.CreatorID, &m.CreatedAt)
    if errors.Is(err, sql.ErrNoRows) {
        return nil, nil
    }
    if err != nil {
        return nil, fmt.Errorf("scan market: %w", err)
    }
    return &m, nil
}
```

### Graceful Shutdown

```go
package main

import (
    "context"
    "log"
    "net/http"
    "os"
    "os/signal"
    "syscall"
    "time"
)

func main() {
    // Initialize dependencies
    db, err := sql.Open("postgres", os.Getenv("DATABASE_URL"))
    if err != nil {
        log.Fatalf("failed to connect to database: %v", err)
    }
    defer db.Close()

    svc := service.NewMarketService(
        repository.NewPostgresMarketRepository(db),
        cache.NewRedisCache(redisClient),
        search.NewElasticSearch(esClient),
    )

    router := NewRouter(svc, authService)

    srv := &http.Server{
        Addr:         ":8080",
        Handler:      router,
        ReadTimeout:  15 * time.Second,
        WriteTimeout: 15 * time.Second,
        IdleTimeout:  60 * time.Second,
    }

    // Start server
    go func() {
        log.Printf("Starting server on %s", srv.Addr)
        if err := srv.ListenAndServe(); err != http.ErrServerClosed {
            log.Fatalf("server error: %v", err)
        }
    }()

    // Wait for shutdown signal
    quit := make(chan os.Signal, 1)
    signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
    <-quit
    log.Println("Shutting down server...")

    // Graceful shutdown with timeout
    ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
    defer cancel()

    if err := srv.Shutdown(ctx); err != nil {
        log.Fatalf("server shutdown error: %v", err)
    }

    // Close database connections
    if err := db.Close(); err != nil {
        log.Printf("database close error: %v", err)
    }

    log.Println("Server stopped gracefully")
}
```
