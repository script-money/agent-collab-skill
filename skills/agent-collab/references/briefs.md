# Brief templates

Copy, fill, delete unused lines. Reply language: follow the user's.

## Implementer brief

```
Goal: <one sentence>        Non-goals / exclusions: <list>
Base revision: <sha>        Uncommitted changes to keep: <paths or none>
Files you may write: <disjoint set>      Read-only context: <files, callers>
Plan / transition table: <path>
Accepted decisions (with rationale): <list>
Findings to fix (id, invariant, reproducer, status, mechanism to use): <list, <=10>
Acceptance criteria: <commands + expected result>
Verification to run: <exact commands>
Authorized operations: <e.g. local edits, mocked tests>   Forbidden: <live calls, push, commit>
Unspecified case? Stop and ask; do not invent semantics.
Return: revision/diff hash, changed files, checks run and failed, assumptions, open risks.
```

## Reviewer brief

```
Role: read-only reviewer. Do not edit files.
Revision under review: <sha or `git diff | shasum`>
Plan and acceptance criteria: <path>
Test commands: <exact commands>
Check in one pass: every acceptance criterion, edge and failure case,
secret leaks, authorization paths, concurrency, doc/code mismatch
(+ transition-table rows and crash points for stateful changes).
You may not add requirements. Report each finding as:
  [blocker|polish] location | violated invariant | reproducer | user impact
Commit reproducers as tests only if told to.
```

## Re-verification brief (round 2+)

```
Same reviewer and revision rules as before. Previous findings: <id list with status>.
Verify only these findings and the fixes' direct neighbors. Report each as
fixed / not fixed / regressed, with evidence. New findings only if caused by the patch.
```

## Handoff after interruption

```
State at <revision/diff hash>: done: <...>  half done: <...>  failing: <...>
Decisions so far: <...>   Next step: <...>   Verification commands: <...>
```
