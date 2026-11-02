# Skills

Agent Skills by [Hector Zurga](https://github.com/HectorZR), following the [Agent Skills](https://www.skills.sh) open standard.

> **Note:** `team-orchestration` is built for **Claude Code**: it relies on spawning sub-agents with an explicit `model`, resuming agents by ID, and git worktrees. Other agents can load it, but they need equivalent capabilities to run the workflow.

## Skills

### 🎯 team-orchestration

Act as PM/Product Owner orchestrating a flat multi-agent team — UX Designer → Developer → QA, plus a transversal DevOps engineer — to build features in parallel. The PM spawns every agent directly (no nested sub-agents). Each developer works in an isolated git worktree; merges are serial; every feature requires human sign-off before merging.

Highlights:

- **PM-side decomposition** — breaks your request into parallelizable features (max 3 in flight) and confirms the plan with you before spawning anyone.
- **Organic-Driven Development (ODD)** — developers grow each feature from the existing codebase: explore patterns, ship the smallest working slice, then extend it in small verified steps.
- **Design briefs** — for UI features, designers run first: exact copy, element ids, and security constraints are fixed; layout is guidance. Security flows get scrutinized before code exists.
- **Isolated runtimes** — each worktree gets its own deps, env, port, and database, so parallel features and QA runs don't collide.
- **Scope-change protocol** — emergent-but-necessary work is implemented and reported; scope additions stop the developer (`BLOCKED:`) until you approve, then the same agent resumes.
- **`.orchestration/` workspace** — a shared folder where agents record decisions and scope changes, so a decision in feature A reaches feature B's developer before it bites.
- **Trust nothing** — the PM verifies agent reports against the actual code, and capacity probes validate the environment before committing hours.

Trigger with phrases like *"orchestrate these features in parallel"*, *"levanta el equipo"*, *"run the team on X and Y"* — in any language — or explicitly with `/team-orchestration`.

## Install

With the [skills CLI](https://www.skills.sh) (works across supported agents):

```bash
npx skills add HectorZR/skills/team-orchestration
```

Or manually for Claude Code:

```bash
git clone https://github.com/HectorZR/skills.git
cp -r skills/team-orchestration ~/.claude/skills/          # global
# or: cp -r skills/team-orchestration .claude/skills/      # per-project
```

Then type `/` in Claude Code — `/team-orchestration` appears in the list.

## License

MIT
