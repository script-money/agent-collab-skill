# Stateful changes (read before any code)

For changes that can submit or cancel external requests, alter remote settings,
move money, run migrations, delete data or otherwise leave external state partly changed.
Designing through review findings costs many fix/review cycles; this module moves
the design and test work to the start.

## Design first

The lead writes a **transition table** in the feature doc and approves it before
implementation. Rows are state x event. Events must include:

- crash and restart after every durable write
- timeout with unknown outcome
- partial success (e.g. a batch where only some items applied) and trigger without execution
- duplicate or replayed event
- a second worker or process
- cancel, kill or expiry arriving mid-flight

Each row states the evidence required, next state, side effect and reported
result. In the same step choose the durability mechanisms:

- write-ahead intent with a stable client-side id before any send
- one send lease per intent
- persisted cancel and replace requests
- atomic journal-plus-snapshot commits
- authoritative readback as the only way to resolve an unknown outcome

Distinguish API acceptance, trigger, execution and confirmed cancellation. List
each side effect of a multi-step flow separately (e.g. a settings change before the main request).

## Invariants at the write boundary

- Verify account, environment, target resource and request identity; reconcile by stable ids.
- A write with unknown outcome needs readback before any retry.
- Revalidate stale inputs; document residual races.
- Check limits against the exact submitted values at the remote system's
  precision; an estimate is not a bound on the real outcome.
- Missing, partial or malformed evidence fails closed.

## Tests come from the table

- Build a fault-injecting fake of the external system before the state machine.
- One test per table row; a crash-point sweep (crash after each durable write,
  restart, replay); invariant checks (no double send, no orphaned side effects,
  idempotent ledger).
- Each regression test must fail on the old code. Passing counts are not evidence.

## Gates

- Keep the feature disabled or shadow-only until a review of the integrated
  revision reports no blockers.
- Live account or production checks need explicit user authorization; use mocks or
  disposable fixtures otherwise. Inspect a command's behavior before treating a
  preview or dry run as read-only.
- The reviewer additionally checks every table row, crash point, concurrency case
  and authorization path.
