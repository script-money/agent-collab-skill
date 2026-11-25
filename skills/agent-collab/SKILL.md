---
name: agent-collab
description: "System-level workflow for planning, leading, delegating and accepting non-trivial engineering work across multiple agents or models: write the plan and acceptance criteria first, run as lead, brief implementers and reviewers, review a fixed revision, and sign off. Use when the user asks for a plan, delegation, multi-model collaboration, an independent review/验收, or the change is large or risky (multi-file features, external side effects such as payments, migrations, deployments). 触发词: 做计划, 主控, 协作, 分工, 子agent, 评审, 验收. Skip for small single-agent edits."
---

# Agent collaboration: plan, lead, collaborate, accept

Project facts (commands, authorized operations, forbidden paths, preferred models)
come from the project's own AGENTS.md / README. This skill sets process only and
never grants authority for live, destructive or publishing actions; those need
explicit user authorization.

## 0. Pick the tier

| Tier | When | Process |
|---|---|---|
| Light | one agent, reversible, small | skip this skill |
| Standard | multi-file feature, refactor, anything delegated or independently reviewed | sections 1–4 |
| Stateful | can leave external state partly changed (payments, migrations, deploys, external API writes, data deletion) | Standard + `references/stateful-changes.md`, read before any code |

When unsure, pick the higher tier.

## 1. Plan (before any code)

Write a short plan file (project `docs/` or a scratch path) with:

1. **Goal and non-goals**: one paragraph, plus what is explicitly out of scope.
2. **Acceptance criteria**: observable and runnable (commands, expected output,
   behaviors). Written before implementation so the reviewer can use them.
3. **Map**: files, callers, contracts and docs touched; base revision and any
   uncommitted changes.
4. **Slices**: ordered phases, each with one owner and a disjoint write set.
5. **Decisions and open questions**: each with rationale. Ask the user only when
   the answer changes the result; otherwise decide, record it, move on.
6. **Risks and tier** (section 0).

Get the user's approval of the plan when scope is large or ambiguity is material.

## 2. Lead

The lead (the main agent talking to the user) owns scope, decisions, integration
and sign-off. The lead:

- does the design itself, or uses the strongest available model for it; never
  delegates the plan;
- is the single writer for any shared path or contract; parallel workers get
  disjoint write sets;
- turns every finding into a violated invariant and a mechanism, not a task list
  forwarded verbatim; walks sibling paths of each finding before assigning a fix;
- does not skip a gate to save a round, and does not self-certify its own code;
- honors explicit model choices from the user; if a model or tool is
  unavailable, reports it and asks instead of silently switching;
- keeps a running ledger in the plan file: done, in progress, failing, decisions.

## 3. Collaborate

- **Implementer**: runs one phase end to end, follows the plan, never invents
  semantics. An unspecified case goes back to the lead as a question, not into
  code. A fix that changes behavior updates the plan first, then code, then tests.
- **Reviewer**: read-only, works on a fixed revision, uses a different model
  family from the implementer, and stays the same across rounds so probes are
  comparable. Reviews everything in one pass instead of drip-feeding findings.
  May not set new requirements.
- **Briefs are self-contained** (templates in `references/briefs.md`): goal and
  exclusions, files and callers, base revision and uncommitted changes, accepted
  decisions with rationale, acceptance criteria, verification commands,
  authorized operations, expected reply format. Keep a task to about ten items,
  grouped by shared mechanism.
- **Interruption**: if a worker stops or hits a limit, the lead reads
  `git diff`, records what is done, half done and failing, and hands that to the
  replacement. On resume, confirm the working tree before retrying.
- Tests written only by the implementer share its blind spots. Reviewer probes
  that found a bug are committed as tests too.

## 4. Accept

1. **Fix the revision**: record `git diff | shasum` (or the commit) and the exact
   test commands with the review request. If the tree is uncommitted, ask before
   committing.
2. **Review pass**: the reviewer checks every acceptance criterion, edge case,
   secret leak, authorization path and doc/code mismatch. Each finding is tagged
   `blocker` or `polish` and gives location, violated invariant, reproducer and
   user impact.
3. **Classify** each finding: patch regression, existing issue blocking the
   requested behavior, or separate finding. Fix the first two, record the third,
   ask before expanding scope.
4. **Fix classes, not instances.** The same invariant failing twice means the
   mechanism is wrong: stop patching and revise the design.
5. **Narrow re-verification** of the previous findings is mandatory and is not
   folded into the next phase's audit. Do not start phase N+1 on open blockers.
6. **Convergence budget**: after two fix-and-review cycles in a phase without a
   clean pass, stop and report full state and options (redesign, narrow scope,
   ship disabled/shadow-only) to the user.
7. **Done** means: acceptance criteria demonstrated by commands run on the
   integrated revision, no in-scope blockers from the requested review, failed or
   skipped checks reported accurately (never converted to success with
   `|| true`), and remaining verification limits stated.

Final report to the user: revision, changed files, checks run and failed,
decisions made, assumptions, unresolved risks, and what needs their decision.
