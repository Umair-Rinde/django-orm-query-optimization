# Indexes and Query Plans

Load when one SQL statement is slow or evidence suggests poor access paths.

## Workflow

1. Identify slow SQL and its parameter pattern, redacting sensitive values.
2. Inspect `QuerySet.explain()` and, when suitable, database-native plans and slow-query telemetry.
3. Review predicates, joins, sort/group operations, estimated versus actual rows, repeated loops, and row estimates.
4. Propose an index only when it fits the real filter/order/join pattern.
5. Add via model `Meta.indexes` and a migration; verify on representative data after deployment.

## Index considerations

- Single-column indexes may help selective filters and joins.
- Composite index column order must match query predicates and ordering; do not index every filtered field mechanically.
- Partial/conditional, expression, and covering indexes depend on backend and version.
- Unique constraints may already create indexes. Avoid redundant indexes.
- Indexes consume storage and add write overhead. A sequential scan can be optimal for small tables or low-selectivity conditions.
- Confirm the database actually chooses the proposed index. Do not promise a speedup without measurement.

`EXPLAIN ANALYZE`-style options may execute the query and have meaningful cost; follow database safety guidance, especially in production. Use docs for the installed Django and database versions.
