# Relationship Loading, Static Review, and Tests

Load this reference for N+1 queries, DRF serializers, templates, admin methods, model properties, or static-only review.

## Relationship loading

### Foreign key / one-to-one

Before:

```python
orders = Order.objects.all()
for order in orders:
    print(order.customer.name)  # likely one extra query per order
```

After:

```python
orders = Order.objects.select_related("customer")
for order in orders:
    print(order.customer.name)
```

Use `select_related()` for single-valued relationships. Confirm the relation is accessed and avoid joins that load unused data.

### Reverse FK / many-to-many

```python
orders = Order.objects.prefetch_related("items__product")
```

Use `prefetch_related()` for multi-valued relationships. Prefetching does not help a later, different query such as `order.items.filter(active=True)`.

### Filtered prefetch with `to_attr`

Before:

```python
for order in Order.objects.all():
    active_items = order.items.filter(active=True)  # repeated query
```

After:

```python
from django.db.models import Prefetch

orders = Order.objects.prefetch_related(
    Prefetch(
        "items",
        queryset=OrderItem.objects.filter(active=True).select_related("product"),
        to_attr="active_items",
    )
)
for order in orders:
    active_items = order.active_items
```

`to_attr` stores a list of the filtered subset. Use a clear name and update callers to consume that list.

## DRF `SerializerMethodField` N+1

Before (likely one query per serialized book):

```python
class BookSerializer(serializers.ModelSerializer):
    author_name = serializers.SerializerMethodField()

    def get_author_name(self, obj):
        return obj.author.name
```

If the view uses `Book.objects.all()`, the serializer may fetch each author separately.

After:

```python
class BookViewSet(ModelViewSet):
    serializer_class = BookSerializer
    queryset = Book.objects.select_related("author")
```

If the method performs a per-book count, consider an annotation:

```python
from django.db.models import Count

queryset = Book.objects.annotate(review_count=Count("reviews"))
```

Adapt field names to the actual schema. Verify permissions, filters, and response output.

## Static-only procedure

When no server/database is available:

1. Trace URL/view → queryset → serializer/template/admin → model fields and methods.
2. Search `SerializerMethodField`, nested serializers, `__str__`, `@property`, `list_display`, template loops, relation access, and queryset evaluation in loops.
3. Mark the issue **likely N+1 / runtime-unconfirmed**. Code inspection alone cannot prove query count or latency.
4. Suggest a focused test and tell the user how to run it in the project environment.

Useful commands (adapt module names):

```bash
python manage.py test path.to.tests
python -m pip show Django
```

## Query-count regression test

Example for a DRF list endpoint; adapt URL, factories, permissions, and expected count:

```python
from django.test import TestCase
from django.urls import reverse
from rest_framework.test import APIClient

class BookListQueryTests(TestCase):
    def test_list_query_count_does_not_grow_per_book(self):
        # Create multiple books with distinct authors using project factories.
        client = APIClient()
        url = reverse("book-list")

        # The number is illustrative: replace with a measured stable count.
        with self.assertNumQueries(2):
            response = client.get(url)

        self.assertEqual(response.status_code, 200)
```

`2` is a placeholder, not a universal expected count. Determine a stable count in the actual project. Use multiple parent and related rows so an N+1 cannot hide behind a one-row fixture. Account for middleware, authentication, pagination, and setup queries. A lower query count alone does not prove lower latency.
