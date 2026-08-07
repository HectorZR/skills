# Skills

Agent Skills by [Hector Zurga](https://github.com/HectorZR). Compatible with Claude Code and any agent supporting the [Agent Skills](https://www.skills.sh) open standard (Cursor, Copilot, Windsurf, etc.).

## Skills

### 🎯 team-orchestration

Act as PM/Product Owner orchestrating a multi-agent team — UX Designer → Developer lead → QA, plus a transversal DevOps engineer — to build features in parallel. Each developer works in an isolated git worktree; merges are serial; every feature requires human sign-off before merging.

Highlights:

- **PM-side decomposition** — breaks your request into parallelizable features (max 3 in flight) and confirms the plan with you before spawning anyone.
- **Binding design contracts** — designers run before developers and produce state machines, exact copy, and element ids; security flows get scrutinized before code exists.
- **Scope-change protocol** — emergent-but-necessary work is implemented and reported; scope additions require your explicit approval first.
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
