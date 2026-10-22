---
name: django-orm-query-optimization
description: "Diagnose and safely optimize Django ORM/database access in views, APIs, serializers, templates, admin, and jobs. Use for suspected N+1 queries, slow SQL, excessive data loading, inefficient aggregation, pagination, indexing, caching, bulk writes, and ORM performance tests."
category: backend-and-data
source_repo: hashgraph-online/awesome-codex-plugins
source_path: "plugins/LVTD-LLC/skills/skills/django-orm-query-optimization/SKILL.md"
---

# Django ORM Query Optimization

Optimize a demonstrated bottleneck without changing application behavior. Prefer the smallest justified fix. If evidence does not justify a change, explain why and stop.

## Operating modes

Choose the strongest mode available; do not assume the application can be run.

1. **Runtime available:** reproduce the exact path; capture query count, duplicate SQL, latency, and relevant query plans before and after.
2. **Partial runtime:** use available tests, logs, or database access. Separate observed evidence from assumptions.
3. **Static-only:** inspect code and identify *likely* issues, never claim an N+1 or speedup is confirmed. Give file/line references, explain the reasoning, propose the minimal fix, and provide a test or command the user can run.

## Workflow

1. **Identify the path and constraints.** Find the view/viewset, service, task, template, admin method, or serializer. Inspect models, managers, relationships, permissions, tenant scoping, filters, ordering, pagination, and tests.
2. **Build a query map.** Trace relationship access in loops, serializer fields, template rendering, `__str__`, properties, admin display methods, signals, and permission checks. Search for repeated queryset evaluation (`count`, `exists`, `list`, `len`, `bool`, iteration).
3. **Classify the bottleneck.** Distinguish too many queries (N+1), one expensive SQL statement, excessive rows/columns, Python processing, memory pressure, lock waits, and repeated cacheable work.
4. **Choose a targeted strategy.** Use the decision table below and load only the relevant reference file for advanced topics.
5. **Validate semantics.** Preserve response shape, ordering, permissions, tenant isolation, business logic, transaction behavior, signal/save semantics, and cache correctness.
6. **Verify or provide a verification plan.** Repeat the baseline measurement where possible. In static-only mode, label findings as unconfirmed and include a focused test or command.
7. **Stop when justified.** Do not optimize a small, already-fast path without a user-visible or operational reason. Do not add indexes, caches, raw SQL, broad prefetches, or complexity speculatively.

## Quick decision table

| Symptom | First strategy to inspect |
|---|---|
| Query count grows with result rows | `select_related`, `prefetch_related`, serializer/template/model-property access |
| One SQL query is slow | SQL plan, predicates, ordering, indexes, join/subquery shape |
| Wrong or inflated aggregate | Join multiplication, grouping, `distinct`, conditional aggregates, subqueries |
| Too many rows or wide payload | Filtering, pagination, `values()` / `values_list()`, selected fields |
| Large Python loop over related rows | `annotate`, `aggregate`, `Exists`, `Subquery`, or bulk database operation |
| Deep pages are slow | Stable indexed keyset/cursor pagination, if API semantics permit |
| Large batch writes are slow | Bulk operations and batching; check signals and `save()` behavior |
| Same expensive result is repeatedly read | Cache only if invalidation and tenant/user scope are clear |
| Latency under load, not locally | Locks, transaction duration, connections, pool/worker/database capacity |
| Async endpoint blocks | Sync ORM calls and supported async boundaries; async does not fix poor query shape |

## Core ORM patterns

- Use `select_related()` for single-valued `ForeignKey` and `OneToOneField` paths.
- Use `prefetch_related()` for reverse foreign keys and many-to-many relationships. Use `Prefetch()` for filtered/ordered/nested querysets; `to_attr` clearly stores a filtered subset.
- Use `values()` / `values_list()` when model instances and their behavior are unnecessary.
- Use `only()` / `defer()` only when avoided field loading is meaningful; deferred-field access can cause extra queries.
- Use `exists()` / `count()` only when their result is needed on its own; avoid an extra query before later evaluating the same queryset.
- Use database-side aggregation and expressions when they avoid loading rows or querying per row. Validate joins so aggregates are not inflated.
- Avoid broad eager loading, unbounded result sets, unnecessary ordering, redundant `distinct()`, and raw SQL without evidence.
- Inspect `__str__`, properties, serializer method fields, template includes, admin methods, and signals for hidden database access.
- Check installed Django/backend versions before using version- or backend-specific features.

## Static-only inspection checklist

When the app or database cannot be run:

- Trace the likely request path from URL/view to queryset, serializer/template, and model.
- Search for relationship access inside loops and methods called once per serialized/rendered row.
- Inspect `SerializerMethodField`, nested serializers, model properties, `__str__`, template relationship traversal, and admin `list_display`.
- Look for `.filter()` or `.order_by()` on a relation after prefetching; this may bypass the prefetched cache.
- Look for queryset evaluation inside loops and `count()` / `exists()` followed by full iteration.
- Check for Python-side grouping/summing, deep offset pagination, unbounded list endpoints, and likely index/query-plan concerns.
- Report findings as **confirmed from code** (pattern exists) or **runtime-unconfirmed** (performance impact not measured). Do not invent query counts or timings.
- Provide a targeted proposed diff and a reproducible verification command/test.

## Evidence and output scale

For a small, local fix, respond with: **finding + change + test/verification**. Include the diff or changed snippet.

For a broader investigation, report: **baseline/evidence → root cause → change → correctness risks → before/after verification → remaining trade-offs**.

Never invent benchmark results. If measurements are unavailable, say so and give the exact next steps needed to verify.

## References: load only what applies

- Relationship loading, hidden N+1s, serializers, templates, static review, and tests: `references/relationship-loading-and-tests.md`
- Aggregation, expressions, and join multiplication: `references/aggregation-and-expressions.md`
- Indexes and query plans: `references/indexes-and-query-plans.md`
- Pagination and large result sets: `references/pagination-and-large-data.md`
- Bulk writes: `references/bulk-writes.md`
- Caching: `references/caching.md`
- Transactions, locks, connections, and async ORM: `references/transactions-connections-async.md`
- Skill self-check scenarios: `references/skill-validation-cases.md`
- DRF relation behavior: https://www.django-rest-framework.org/api-guide/relations/

Before consulting version-specific docs, determine the installed version (for example, `python -m pip show Django` or inspect `requirements*.txt`, `pyproject.toml`, `Pipfile.lock`, or the lockfile). Then use matching official Django documentation. If the version cannot be determined, state that and avoid version-specific APIs. For older versions, use supported explicit `select_related()` / `prefetch_related()` patterns as the conservative fallback.
