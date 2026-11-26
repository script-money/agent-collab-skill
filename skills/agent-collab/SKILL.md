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
| Stateful | can leave external state partly changed (payments, migrations, deploys, external API writes, data deletion) | Standard + [stateful changes](references/stateful-changes.md), read before any code |

Choose by reachable effects; when unsure, pick the higher tier. A plan-only
request ends with the plan.

## 1. Plan (before any code)

Write a short plan file (project `docs/` or a scratch path) with:

1. **Goal and non-goals**: one paragraph, plus what is explicitly out of scope.
2. **Acceptance criteria**: observable and runnable (commands, expected output,
   behaviors). Measure the intent, not a proxy met by moving code (count write
   entry points, not lines in one file).
3. **Map**: files, callers, contracts and docs touched; base revision and any
   uncommitted changes. Aim for the smallest change that delivers the behavior.
4. **Slices**: ordered phases, each with one owner and a disjoint write set.
5. **Decisions and open questions**: each with rationale. Ask the user only when
   the answer changes the result; otherwise decide, record it, move on.
6. **Operating model and risks**: supported entry points, workers, failure modes
   and exclusions with how each is enforced. Define what would block acceptance.

Get user approval when scope is large, ambiguity is material, or a scope cut or
exclusion weakens the user's guarantees. For Stateful, approve a one-page
summary: write entry points, exclusions, residual risks.

**Team**: unless the user already chose, ask once during planning for the whole
team (lead, implementer, reviewer), offering the presets in AGENTS.md as options.
Record the choice in the plan. Later role changes go back to the user.

**Review bar**: in the same round, ask what may block acceptance, so review does
not grow a simple request into endless detail work:

1. Basic: acceptance criteria and correctness of the requested behavior
2. Standard: + failure paths, authorization, secrets, doc/code mismatch (default)
3. Strict: + every reachable edge, concurrency, recovery (default for Stateful)

Anything outside the bar is polish: recorded, never blocking, fixed only on request.

## 2. Lead

The lead (the main agent talking to the user) owns scope, decisions, integration
and sign-off. The lead:

- does the design itself, or uses the strongest available model for it; never
  delegates the plan;
- is the single writer for any shared path or contract; parallel workers get
  disjoint write sets and agreed interfaces;
- turns every finding into a violated invariant and a mechanism, not a task list
  forwarded verbatim; walks sibling paths of each finding before assigning a fix;
- fixes small mechanical findings directly and includes them in the reviewed revision;
- does not skip a gate to save a round, and does not self-certify its own code;
- honors explicit model choices from the user; if a model or tool is
  unavailable, reports it and asks instead of silently switching;
- keeps a running ledger in the plan file: done, in progress, failing, decisions.

## 3. Collaborate

- **Implementer**: runs one phase end to end, follows the plan, never invents
  semantics. An unspecified case goes back to the lead as a question, not into
  code. A fix that changes behavior updates the plan first, then code, then tests.
- **Reviewer**: read-only, works on a pinned revision, preferably a different
  model family from the implementer (user role assignments win). Reviews
  everything in one pass. May not set new requirements.
- **Briefs are self-contained** (templates in [briefs.md](references/briefs.md)): goal and
  exclusions, files and callers, base revision and uncommitted changes, accepted
  decisions with rationale, acceptance criteria, verification commands,
  authorized operations, expected reply format. Keep a task to about ten items,
  grouped by shared mechanism.
- **Interruption**: if a worker stops or hits a limit, the lead reads
  `git diff`, records what is done, half done and failing, and hands that to the
  replacement. On resume, confirm the working tree before retrying.
- **Test author**: start independent test work once the public contract is stable.
  Extend existing coverage for a demonstrable gap. If the same model also
  reviews, give review to a fresh task that did not write the tests, and say so.

## 4. Accept

1. **Pin the revision.** A commit, or base plus `git diff | shasum` covering
   staged and relevant untracked files; ask before committing. Give the reviewer
   the acceptance criteria and verification commands.
2. **Review once against the agreed bar.** Each finding: location, violated
   invariant, reproducer, impact. A reachable violation within the bar is a blocker. Only an exclusion listed in the approved
   plan may be recorded as residual risk, with evidence it is enforced.
3. **Classify and repair.** Fix patch regressions and existing issues that block
   the requested behavior; record other findings and ask before expanding scope.
   Inventory all callers sharing the failed invariant, fix their common boundary,
   and commit a structural check (grep, import guard, single-caller test) that
   no sibling path bypasses it. A recurring failure means redesign, not a patch.
4. **Re-verify on the integrated fix.** Recheck the reproducer, shared mechanism,
   direct neighbors and regressions introduced by the fix. Broaden review if
   the design or affected surface changes. Resolve blockers before advancing.
5. **Bound rework.** Count one cycle per fix followed by re-review. Allow up to
   two cycles per phase and three across the task. Carry the count through scope
   cuts and phase renames. If blockers persist at either limit, report the
   evidence and options for redesign, reduced scope or disabled delivery.
   Only the user can extend the budget. Exhaustion never waives a blocker.
6. **Sign off.** Demonstrate the criteria on the integrated revision. Report files,
   decisions, failed or skipped checks (never masked with `|| true`),
   verification limits and remaining risks.
   Identify any pending decision and whether the requested review cleared blockers.
