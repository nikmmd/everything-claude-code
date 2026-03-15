---
name: postgres-patterns
description: PostgreSQL query optimization, indexing strategies, migration patterns, and ORM usage. Auto-triggers when working with database queries, schema changes, or migrations.
compatibility: Designed for Claude Code
---

# PostgreSQL Patterns

Auto-activates when working with database queries, schema changes, or migrations. For deep analysis, delegate to the **database-designer** agent.

## Indexing Strategy

```sql
-- B-tree (default): equality and range queries
CREATE INDEX idx_student_email ON student(email);

-- Partial index: only index what you query
CREATE INDEX idx_active_courses ON course(department)
  WHERE is_active = true;

-- Composite: multi-column lookups (column order matters — most selective first)
CREATE INDEX idx_enrollment_lookup ON enrollment(student_id, semester, course_id);

-- GIN: array/jsonb containment
CREATE INDEX idx_course_tags ON course USING GIN(tags);

-- Expression: computed lookups
CREATE INDEX idx_lower_name ON student(LOWER(name));
```

**When to add indexes:**
- Columns in WHERE, JOIN ON, ORDER BY
- Foreign keys (PostgreSQL doesn't auto-index these)
- Columns with high selectivity (many distinct values)

**When NOT to index:**
- Small tables (< 1000 rows)
- Columns with low selectivity (boolean flags with 50/50 split)
- Tables with heavy write load and rare reads

## N+1 Prevention

```typescript
// BAD: N+1 queries
const students = await db.select().from(student)
for (const s of students) {
  const enrollments = await db.select().from(enrollment)
    .where(eq(enrollment.studentId, s.id))
}

// GOOD: Single query with join
const results = await db
  .select()
  .from(student)
  .leftJoin(enrollment, eq(enrollment.studentId, student.id))
  .leftJoin(course, eq(course.id, enrollment.courseId))
  .where(eq(enrollment.semester, 'fall-2025'))
```

## Pagination

```typescript
// Offset-based (simple, fine for small datasets)
const items = await db.select().from(course)
  .limit(20)
  .offset((page - 1) * 20)

// Cursor-based (better for large/real-time datasets)
const items = await db.select().from(course)
  .where(gt(course.id, cursor))
  .orderBy(course.id)
  .limit(20)
```

## Migrations

**Safety rules:**
- Never drop columns in production without a deprecation period
- Add columns as nullable first, backfill, then add NOT NULL
- Create indexes `CONCURRENTLY` to avoid table locks
- Test migrations against a copy of production data

```sql
-- Safe: add nullable column
ALTER TABLE student ADD COLUMN phone VARCHAR(20);

-- Safe: concurrent index (no lock)
CREATE INDEX CONCURRENTLY idx_phone ON student(phone);

-- Dangerous: adding NOT NULL without default blocks writes
-- Instead: add nullable → backfill → alter to NOT NULL
```

## Query Diagnosis

```sql
-- Explain query plan
EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT)
  SELECT * FROM student WHERE email = 'test@example.com';

-- Find slow queries
SELECT query, mean_exec_time, calls
FROM pg_stat_statements
ORDER BY mean_exec_time DESC
LIMIT 10;

-- Unused indexes (candidates for removal)
SELECT indexrelname, idx_scan, pg_size_pretty(pg_relation_size(indexrelid))
FROM pg_stat_user_indexes
WHERE idx_scan = 0
ORDER BY pg_relation_size(indexrelid) DESC;
```

## Connection Pooling

For serverless databases (Neon, Supabase): always use the pooled connection URL, not direct.

```
# Pooled — for application queries
DATABASE_URL=postgres://...neon.tech/db?sslmode=require

# Direct — for migrations only
DATABASE_URL_DIRECT=postgres://...neon.tech/db
```

## Common Drizzle Patterns

```typescript
// Transactions
await db.transaction(async (tx) => {
  const student = await tx.insert(studentTable).values(data).returning()
  await tx.insert(enrollmentTable).values({ studentId: student[0].id, courseId })
})

// Conditional where clauses
const query = db.select().from(course)
if (department) query.where(eq(course.department, department))
if (active) query.where(eq(course.isActive, true))
```
