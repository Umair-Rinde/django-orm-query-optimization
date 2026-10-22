# Aggregation, Expressions, and Query Correctness

Load for Python loops over related rows, slow summaries, or suspicious aggregate results.

## Move repeated calculations into the database

Before (may load many rows and perform work in Python):

```python
orders = Order.objects.filter(status="paid")
total = sum(
    line.amount
    for order in orders
    for line in order.lines.all()
)
```

After (illustrative field names):

```python
from django.db.models import Sum

summary = OrderLine.objects.filter(
    order__status="paid"
).aggregate(total=Sum("amount"))
total = summary["total"] or 0
```

For per-parent summaries:

```python
from django.db.models import Count

orders = Order.objects.annotate(line_count=Count("lines"))
```

Use `aggregate()` for a single summary and `annotate()` for per-row values. Validate null handling, numeric types, filters, and grouping semantics.

## Avoid join multiplication

Joining two multi-valued relations can multiply rows and inflate counts or sums. An order with 3 lines and 2 payments can produce 6 joined rows. `Count(..., distinct=True)` does not automatically fix every sum; distinct sums can be wrong when equal amounts are legitimate.

Consider separate correlated subqueries, filtered aggregates, or separate summary queries. Example shape:

```python
from django.db.models import OuterRef, Subquery, Sum

line_totals = (
    OrderLine.objects
    .filter(order_id=OuterRef("pk"))
    .order_by()
    .values("order_id")
    .annotate(total=Sum("amount"))
    .values("total")[:1]
)
orders = Order.objects.annotate(line_total=Subquery(line_totals))
```

Adapt field names and output types. Inspect SQL and compare results against known fixtures.

## Expressions

- `F()` for database-side arithmetic/comparisons.
- `Case` / `When` for conditional values.
- `Exists()` when only related-row existence matters.
- `Subquery()` / `OuterRef()` for correlated values when appropriate.
- `values(...).annotate(...)` for grouped reports.

None is always faster. Compare query plans and results on representative data.
