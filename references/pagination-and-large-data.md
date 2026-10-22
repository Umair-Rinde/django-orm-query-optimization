# Pagination and Large Data

Load for unbounded endpoints, exports, memory pressure, or slow deep pages.

- Bound list endpoints and admin lists; filter before materializing.
- Ensure ordering is deterministic and includes a unique tie-breaker when needed.
- For large one-pass jobs, consider `iterator(chunk_size=...)` or chunked primary-key batches. Check prefetch support, cursor behavior, and connection lifetime.
- Avoid prefetching huge child collections if a count, existence check, or limited subset is enough.
- Avoid `list(queryset)` on unbounded data unless full materialization is required.
- Change exact-count behavior only if the API/product contract permits it.

## Keyset pagination example

Deep offset pagination can be expensive:

```python
page = Event.objects.order_by("-created_at", "-id")[100000:100050]
```

A keyset query uses the last row of the previous page as a cursor. For descending `(created_at, id)` ordering:

```python
from django.db.models import Q

def next_events_page(last_created_at=None, last_id=None, page_size=50):
    qs = Event.objects.order_by("-created_at", "-id")
    if last_created_at is not None and last_id is not None:
        qs = qs.filter(
            Q(created_at__lt=last_created_at)
            | Q(created_at=last_created_at, id__lt=last_id)
        )
    return list(qs[:page_size])
```

Return the last row's `(created_at, id)` as the next cursor. Validate/encode cursor values safely, use stable ordering, and evaluate a matching index. Adapt nullable fields and database behavior. Keyset pagination changes navigation semantics (such as arbitrary page jumps); use only when the API contract permits it.
