# Skill Validation Cases

Use these cases to check that the skill triggers appropriately and avoids unsupported claims. These are illustrative, not benchmark results.

## 1. Slow DRF serializer

**Pattern:** A list endpoint serializes many books; `SerializerMethodField` reads `obj.author.name`; queryset is `Book.objects.all()`.

**Expected:** Static-only mode reports a *likely N+1, runtime-unconfirmed* and suggests `select_related("author")`. Runtime mode measures before/after using multiple books and authors. Never invent a query count.

## 2. Incorrect aggregate from join multiplication

**Pattern:** An order has 3 line items and 2 payments; one query annotates both `Count("lines")` and `Sum("payments__amount")`.

**Expected:** Explain that joining two multi-valued relations may multiply rows and inflate aggregates. Validate with fixtures; evaluate separate subqueries or aggregation queries. Do not blindly add `distinct=True` to sums.

## 3. Deep-offset pagination

**Pattern:** An endpoint uses `Event.objects.order_by("-created_at")[200000:200050]`.

**Expected:** Flag likely deep-offset cost, not proven slowness. Propose keyset pagination only if API semantics permit it; require stable ordering, a unique tie-breaker, safe cursor handling, and index evaluation.

## 4. Tiny, already-fast admin list

**Pattern:** A bounded admin page shows 20 rows, measured latency is low, and no problem is reported.

**Expected:** Recommend no change unless there is a measurable or user-visible issue. Do not add speculative indexes, caches, or eager loading.

## 5. Static-only repository review

**Pattern:** Cursor can inspect code but cannot start the app or connect to the database.

**Expected:** Trace likely query access from view to serializer/model/template; separate code-confirmed patterns from unconfirmed impact; supply a focused test and commands; never fabricate query counts, query plans, latency, or improvement percentages.
