# Brief templates

Copy, fill, delete unused lines. Reply language: follow the user's.

## Implementer brief

```
Goal: <one sentence>        Non-goals / exclusions: <list>
Base revision: <sha>        Uncommitted changes to keep: <paths or none>
Files you may write: <disjoint set>      Read-only context: <files, callers>
Plan / operating model: <path>
Shared boundary and callers affected by this fix: <paths or none>
Accepted decisions (with rationale): <list>
Findings to fix (id, invariant, reproducer, status, mechanism to use): <list, <=10>
Acceptance criteria: <commands + expected result>
Verification to run: <exact commands, each with a timeout>
Authorized operations: <e.g. local edits, mocked tests>   Forbidden: <live calls, push, commit>
Unspecified case? Stop and ask; do not invent semantics.
Return: revision/diff hash, changed files, checks run and failed, assumptions, open risks.
```

## Test author brief

```
Public contract and stable entry points: <revision, paths, accepted behavior>
Existing tests and uncovered invariant: <paths, gap, failure sequence>
Writable tests and fixtures: <paths>    Authorized checks: <commands>
Extend existing coverage where possible.
Return: behavior exercised, old-revision failure for regressions, checks and limits.
```

## Reviewer brief

```
Role: read-only reviewer. Do not edit files.
Revision under review: <integrated commit or complete identified snapshot>
Plan and acceptance criteria: <path>
Test commands: <exact commands, each with a timeout; a timeout is a failed check>
Review bar: <basic|standard|strict, from the plan>. Outside the bar = polish, never blocker.
Check in one pass everything the bar covers.
For stateful changes, trace all mutation paths and verify contract evidence,
reachable failure sequences and enforced operating assumptions. List every
external write entry point (file:function); one the plan does not name is a blocker.
You may not add requirements. Report each finding as:
  [blocker|residual-risk|polish] location | violated invariant | reproducer | user impact
residual-risk only for exclusions in the approved plan, with enforcement evidence.
```

## Re-verification brief (round 2+)

```
Read-only as before. New pinned revision: <sha or diff hash>. Previous findings: <id list with status>.
Recheck the findings, shared mechanism, direct neighbors and patch regressions.
Report fixed / not fixed / regressed with evidence, including sibling bypasses.
Flag changes to the design or affected surface that need wider review.
```

## Handoff after interruption

```
State at <revision/diff hash>: done: <...>  half done: <...>  failing: <...>
Decisions so far: <...>   Next step: <...>   Verification commands: <...>
```
