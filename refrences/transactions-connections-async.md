# Transactions, Connections, and Async ORM

Load for contention, lock waits, async endpoints, or database capacity issues.

## Transactions and locks

- Keep transactions as short as correctness permits; avoid network calls and lengthy Python work while holding locks.
- Use `select_for_update()` only when a concurrency invariant requires row locks. Verify backend support and lock scope.
- Investigate deadlocks, long-running transactions, and lock waits with database-native telemetry.
- Do not weaken transaction guarantees to improve a benchmark.

## Connections and concurrency

- Review connection limits, worker counts, pooling, persistent connections, and database capacity together.
- Check server-side cursor and pool compatibility when streaming results.
- High-load latency may be caused by connection exhaustion or contention even when query count is low.

## Async ORM

- Check installed Django version and database driver support.
- Use documented async ORM methods where available. Do not call blocking synchronous ORM operations directly inside an async event loop.
- Use documented sync/async boundaries such as `sync_to_async()` when required; verify thread-sensitivity, transaction support, and connection behavior.
- Async I/O does not make inefficient SQL cheaper. Reduce query count, rows, and query cost first.
- Avoid parallelizing many small queries when one correct batched query is available.
