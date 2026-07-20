---
name: sql-optimization-patterns
description: Optimizes SQL queries and schemas. Use when a query is slow, when designing indexes, when reading EXPLAIN plans, or when database load is the suspected bottleneck. PostgreSQL-flavored; principles apply to any relational DB.
---

# SQL Optimization Patterns

## Overview

Database optimization is measurement-driven: read the execution plan, fix the actual bottleneck, verify the plan changed. Guessing at indexes is how tables end up with ten of them and slow writes. Examples are PostgreSQL; the concepts (plans, index selectivity, keyset pagination) transfer to MySQL and others — verify syntax per `source-driven-development`.

## When to Use

- A query or endpoint is measurably slow (profile first — see `performance-optimization`)
- Designing indexes for a new table or query pattern
- Reviewing schema/query changes for performance
- Reducing database load, cost, or lock contention

## Read the Plan First

```sql
EXPLAIN (ANALYZE, BUFFERS) SELECT ... ;   -- actual times, not just estimates
```

| Signal in plan | Meaning |
|---|---|
| `Seq Scan` on a large table with a selective filter | Missing or unusable index |
| `Index Scan` | Good — index used, heap visited |
| `Index Only Scan` | Best — answered from the index alone |
| `Rows` estimate wildly off actual | Stale statistics — run `ANALYZE <table>` |
| `Nested Loop` over many rows | Often fine for small sets; a red flag at scale |
| `Filter` removing most rows *after* a scan | Index doesn't cover the predicate |

Optimize the node that dominates actual time; re-run EXPLAIN after each change to confirm the plan improved.

## Index Strategy

```sql
CREATE INDEX idx_orders_user_status ON orders(user_id, status);       -- composite: order matters
CREATE INDEX idx_active_users ON users(email) WHERE status = 'active'; -- partial: hot subset only
CREATE INDEX idx_users_lower_email ON users(LOWER(email));             -- expression: match the query
CREATE INDEX idx_users_email_covering ON users(email) INCLUDE (name);  -- covering: index-only scans
```

**Rules:**
- Index what your WHERE/JOIN/ORDER BY actually use. Composite index column order: equality columns first, then range/sort columns; a `(a, b)` index serves `a` and `a,b` — not `b` alone.
- Every index taxes INSERT/UPDATE/DELETE. Index deliberately, and drop unused indexes (query below).
- B-tree is the default; reach for GIN for JSONB/arrays/full-text, BRIN for huge append-only tables.
- An index is useless if the query wraps the column in a function (`WHERE LOWER(email) = ...`) unless the index is on that expression — or if the predicate has a leading wildcard (`LIKE '%abc'`) or an implicit type cast.

## Query Anti-Patterns → Fixes

| Anti-pattern | Fix |
|---|---|
| N+1: one query per row in a loop | One JOIN, or one batched `WHERE id IN (...)` + group in app code (see `performance-optimization`) |
| `SELECT *` | Select needed columns — enables index-only scans, shrinks payloads |
| `LIMIT 20 OFFSET 100000` | Keyset pagination: `WHERE (created_at, id) < ($cursor_at, $cursor_id) ORDER BY created_at DESC, id DESC LIMIT 20` + matching index |
| Correlated subquery per row | Rewrite as JOIN + GROUP BY, or a window function |
| Row-by-row INSERT/UPDATE loops | Multi-row `VALUES`, `COPY` for bulk loads, or update via join to a temp table |
| `COUNT(*)` over a huge table for UI | Filtered count on an indexed column, or `pg_class.reltuples` estimate when approximate is fine |
| Filtering only in `HAVING` | Move row-level conditions to `WHERE` so they prune before aggregation |

Use CTEs for readability, but check the plan — they can act as optimization fences in some databases/versions.

## Bigger Hammers (Use When Measured Need Exists)

- **Materialized views** — precompute expensive aggregations read far more often than source data changes; you own the refresh strategy (`REFRESH MATERIALIZED VIEW CONCURRENTLY`), so staleness is a feature decision.
- **Partitioning** — for very large tables (100M+ rows) where queries filter on an obvious key (usually time ranges); enables partition pruning and cheap retention drops. Complexity tax — don't reach for it early.
- **Maintenance** — autovacuum usually suffices, but bulk-modified tables may need manual `VACUUM ANALYZE`; keep statistics fresh so the planner chooses well.
- **Application-side** — connection pooling, and caching for read-heavy/rarely-changing data, often beat further query heroics.

## Monitoring

```sql
-- Slowest queries by mean time (requires pg_stat_statements)
SELECT query, calls, mean_exec_time FROM pg_stat_statements ORDER BY mean_exec_time DESC LIMIT 10;

-- Tables doing heavy sequential scans (index candidates)
SELECT relname, seq_scan, seq_tup_read, idx_scan FROM pg_stat_user_tables ORDER BY seq_tup_read DESC LIMIT 10;

-- Unused indexes (drop candidates)
SELECT relname, indexrelname, idx_scan FROM pg_stat_user_indexes WHERE idx_scan = 0
ORDER BY pg_relation_size(indexrelid) DESC;
```

## Red Flags

- Adding indexes without reading the plan (or keeping ones nothing uses)
- ORM code generating N+1 loops or unbounded fetches
- OFFSET pagination on large tables
- Queries whose predicates defeat their own indexes (functions, casts, leading wildcards)
- `SELECT *` in hot paths
- No slow-query visibility in production
- Schema/index changes shipped without before/after timings

## Verification

After optimizing a query or schema:

- [ ] EXPLAIN ANALYZE before/after captured; the plan changed the way you intended
- [ ] Timing improved on production-scale data (dev-sized tables prove nothing)
- [ ] New indexes are exercised by the target queries (`idx_scan` increments)
- [ ] Write-path performance checked — the new index didn't blow up insert/update cost
- [ ] Results are identical to the unoptimized query (correctness first)
