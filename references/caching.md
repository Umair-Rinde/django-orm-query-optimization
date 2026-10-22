# Caching

Load only when repeated reads are expensive and freshness requirements are understood.

- Optimize obvious query-shape problems first; caching should not hide a correctable N+1.
- Include all relevant filters, tenant/user scope, permissions, locale, and version context in cache keys.
- Define invalidation/expiry for updates, deletes, related-object changes, and permission changes.
- Never expose sensitive user-specific results through shared keys.
- Avoid caching huge querysets or live model instances without measuring serialization and memory costs.
- Measure hit rate, staleness risk, memory, and end-to-end latency after rollout.
- If invalidation cannot be correct, do not cache unless a bounded stale lifetime is explicitly acceptable.
