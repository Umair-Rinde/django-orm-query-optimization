# Django ORM Query Optimization

A reusable development skill for identifying, analyzing, and improving Django ORM query performance in existing applications.

The goal is to help developers find meaningful database bottlenecks, understand their root causes, and apply evidence-based optimizations without introducing unnecessary complexity.

## Overview

Django's ORM makes database access convenient, but inefficient query patterns can lead to slow APIs, excessive database calls, unnecessary memory usage, and poor performance as data grows.

This skill provides a structured approach to reviewing Django APIs, models, querysets, serializers, and related application code for optimization opportunities.

## What It Covers

- **N+1 queries:** Identify repeated database queries and improve relationship loading with `select_related()` and `prefetch_related()`.
- **Query efficiency:** Detect duplicate queries, unnecessary database hits, and over-fetching.
- **Aggregations and expressions:** Review `annotate()`, `aggregate()`, `F()`, `Subquery()`, and related ORM patterns.
- **Pagination:** Evaluate offset-based pagination and consider alternatives for large datasets.
- **Indexes and execution plans:** Identify potential indexing improvements and inspect database query plans.
- **Bulk operations:** Evaluate opportunities to replace repeated writes with `bulk_create()` or `bulk_update()`.
- **Caching:** Identify suitable caching opportunities while considering invalidation and data consistency.
- **Transactions and concurrency:** Review transaction boundaries, locking, and connection usage.
- **Performance testing:** Use query-count assertions and targeted tests to prevent regressions.
- **API optimization:** Review Django REST Framework views, viewsets, serializers, and `SerializerMethodField` implementations.

## Core Principles

1. **Measure before optimizing.** Use profiling, query counts, and execution plans when runtime access is available.
2. **Prioritize by impact.** Focus on high-cost queries and frequently used endpoints first.
3. **Preserve behavior.** Ensure optimizations do not change API responses, permissions, filtering, or business logic.
4. **Avoid premature optimization.** If the evidence does not justify a change, document the finding and leave the code unchanged.
5. **Respect project compatibility.** Check the installed Django version and use the matching documentation and supported ORM features.
6. **Be transparent about evidence.** Clearly distinguish confirmed issues from potential problems identified through static analysis.

## How to Use

1. Add the skill to your coding agent's available skills.
2. Ask the agent to audit a Django API, serializer, viewset, queryset, or the entire application.
3. Review the prioritized findings and recommended changes.
4. Implement justified fixes and add regression tests.
5. Compare query counts or execution plans before and after the changes when possible.

### Example Prompts

**Audit an entire application**

```text
Audit the Django APIs for ORM query optimization opportunities.
Identify N+1 queries, inefficient aggregations, unnecessary database
hits, over-fetching, and pagination issues. Prioritize findings by
severity and impact. Do not modify code.
```

**Review a specific endpoint**

```text
Review this API endpoint and its serializers for database query
inefficiencies. Explain the root cause of each issue and recommend
the smallest safe fix. Include a test to verify the improvement.
```

**Implement verified fixes**

```text
Implement the highest-priority confirmed query optimization findings.
Preserve existing API behavior, add regression tests, and compare
query counts before and after where feasible. Report any findings
that could not be verified.
```

## Project Structure

The skill uses a short main instruction file with focused reference documents to keep guidance maintainable and load detailed material only when needed.

```text
django-orm-query-optimization/
├── SKILL.md
└── references/
    ├── relationship-loading-and-tests.md
    ├── aggregation-and-expressions.md
    ├── indexes-and-query-plans.md
    ├── pagination-and-large-data.md
    ├── bulk-writes.md
    ├── caching.md
    ├── transactions-connections-async.md
    └── skill-validation-cases.md
```

## Validation Scenarios

The skill includes scenarios designed to check whether its guidance handles common optimization challenges:

- A slow serializer that triggers N+1 queries.
- Aggregations that produce incorrect results because joins multiply rows.
- Deep-offset pagination on large datasets.
- Small, already-fast endpoints where additional optimization is unnecessary.
- Static reviews where runtime measurements are unavailable.

These scenarios help evaluate whether recommendations are correct, proportionate, and verifiable.

## Contributing

Suggestions and contributions are welcome!

If you find a missing optimization pattern, an incorrect recommendation, an edge case, or a better implementation approach:

1. Open an issue describing the problem or proposed improvement.
2. Submit a pull request with the relevant changes.
3. Include examples or tests where practical.
4. Keep recommendations evidence-based, version-aware, and focused on real performance improvements.

**The goal is to make Django ORM optimization more systematic, practical, and reliable for developers.**

