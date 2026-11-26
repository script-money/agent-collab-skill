# Stateful changes

## Choose the smallest authorized write path

Trace the normal entry points to the external operation. Reuse a reviewed writer
when it enforces the required guarantees. A preview or human-executed step can
be proposed if it would meet the user's goal. Get agreement before replacing
requested automation with a manual step. For a new writer, record the missing
capability, who can invoke it and which existing authorization rules apply.

## Establish the external contract

Before designing recovery or drafting fixtures, check the operations used against
versioned official documentation or schemas. Record the source and the details
the decision relies on, such as identity scope, idempotency, status meanings,
ordering and pagination. Captured responses illustrate payloads. They cannot
establish a general ordering or retry guarantee. If a required fact is unknown,
keep the affected operation blocked and expose the unresolved outcome.

## Design the reachable transitions

Route every in-scope mutation of the same resource through a common guarded
boundary, including cancellation, recovery, CLI and background paths. Inventory
the callers. Keep decisions about transitions separate from transport and storage
where practical. Apply the gates required for each operation at that boundary.

For each reachable state and event, record the evidence, next state, intended
side effect and reported outcome. Distinguish acceptance, execution and confirmed
cancellation. Include each side effect of a multi-step flow. Cover unknown outcomes,
partial success, stale or replayed observations, restart and mid-flight cancellation
where reachable. Exclude concurrency only when ownership is enforced across entry
points. Choose durability mechanisms from the failures they must survive;
defaults: write-ahead intent with a stable client id before any send, one send
lease per intent, readback as the only resolver of an unknown outcome.
Around twenty rows is a cue to look for independent slices or removable states.
Keep cross-slice failure sequences visible.

## Invariants at the write boundary

- Verify account, environment, target resource and request identity; reconcile by stable ids.
- A write with unknown outcome needs readback before any retry.
- Revalidate stale inputs; document residual races.
- Check limits against the exact submitted values at the remote system's
  precision; an estimate is not a bound on the real outcome.
- Missing, partial or malformed evidence fails closed.

## Verify through the public path

Use existing fixtures first. Table-driven cases can exercise decisions with verified,
stale and malformed observations. Where ordering matters, replay event sequences
or use a reproducible model-based test that checks the invariant after each step.

At the transport boundary, script only the responses, timeouts and recorded calls
needed for the scenario. Avoid implementing the provider's matching, scheduling or
storage logic. Exercise actual local persistence and restart at distinct ambiguity
windows. For example, lose the reply after a send, restart and assert that replay
cannot send again while the outcome is unknown. Extend an existing invariant test
when a review probe reveals a gap. Demonstrate the regression on the old behavior.

## Release gates

Keep the feature disabled or shadow-only until independent review of the integrated
revision clears blockers. Trace each external mutation to its guard, check contract
evidence, and exercise the applicable failure and authorization paths. A committed
structural check must show the guarded boundary is the only external writer.

Live account or production checks need explicit user authorization. Use mocks or
disposable fixtures otherwise. Inspect a preview or dry run before treating it as
read-only. Review clearance alone grants no permission to activate the feature.
