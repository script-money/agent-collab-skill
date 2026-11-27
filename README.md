# agent-collab-skill

A system-level skill for planning, leading, delegating and accepting non-trivial
engineering work across multiple agents or models. Tool-agnostic: works with any
agent that reads `SKILL.md` skills (Claude Code, Codex, Antigravity, ...).

```
skills/agent-collab/
  SKILL.md                       plan -> lead -> collaborate -> accept (~100 lines)
  references/briefs.md           implementer / test author / reviewer templates
  references/stateful-changes.md optional module for payments, migrations, deploys, data deletion
```

## Install into a project (when needed)

```bash
# project scope, from a local clone of this repo
npx skills add /path/to/agent-collab-skill

# or manually
mkdir -p <project>/.agents/skills && cp -R skills/agent-collab <project>/.agents/skills/
ln -s ../../.agents/skills/agent-collab <project>/.claude/skills/agent-collab   # Claude Code
```

Once pushed to GitHub: `npx skills add script-money/agent-collab-skill`.
Remove with `rm -rf <project>/.agents/skills/agent-collab` (and the symlink).

## Check an installed copy

From the consuming project, set `skill_repo` to the source checkout and compare
the full skill directory. A difference exits with status 1 and lists affected files.

```bash
skill_repo=/path/to/agent-collab-skill
diff -qr "$skill_repo/skills/agent-collab" .agents/skills/agent-collab
```

Review local differences before reinstalling with the original installation method.
Repeat the comparison after updating and inspect any installer-managed lockfile.

## Project-side contract

Keep project-specific facts in the project's AGENTS.md, not here. A short hook is
enough; team presets are offered when the skill asks for the team during planning:

```markdown
## Collaboration
For multi-agent work or changes that can leave external state partly changed, use
the `agent-collab` skill. Authorized operations and forbidden paths are listed above;
the skill never overrides them.
Team presets:
1. Opus 5.5 (medium) lead + Gemini 4 Hydrogen implementer + GPT-6.2 Sol reviewer
2. Orchestrator (Neon) as lead and implementer (schedules models internally)
   + GPT-6.2 Astra reviewer
```
