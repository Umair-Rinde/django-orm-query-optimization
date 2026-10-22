# Bulk Writes

Load for slow loops that insert or update many records.

- Evaluate `bulk_create()`, `bulk_update()`, or queryset `update()` when per-instance lifecycle behavior is unnecessary.
- Check whether overridden `save()`, signals, validation, audit trails, timestamps, and side effects must run. Bulk methods do not behave identically to saving each instance.
- Choose batch sizes based on backend parameter limits, row width, memory, and transaction requirements.
- Verify uniqueness/constraint behavior, conflict handling, idempotency, partial failure, and transaction boundaries.
- Compare write latency and correctness on representative data. Do not trade away required business behavior for throughput.
