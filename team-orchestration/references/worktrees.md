# Worktrees, probes, and merge discipline

## Why worktrees

Parallel developers editing the same checkout will trample each other. Each feature gets its own git worktree on its own branch; DevOps gets one too. The PM stays on the main checkout and never edits code.

```bash
git worktree add ../<repo>-f<ID> -b feature/<ID>-<slug>
```

Use a consistent naming scheme (`../myrepo-f03`, branch `feature/03-user-invites`) and record each path in `PLAN.md`.

## Confinement

Putting a path in a prompt does not change an agent's working directory. Every spawn prompt must tell the agent to:

1. `cd <WORKTREE_PATH>` at the start of **every** shell command (shell state may not persist), and use absolute paths under the worktree for file tools.
2. Run `git rev-parse --show-toplevel` and `git branch --show-current` first, and stop with `BLOCKED:` if they don't match.

If your agent runtime offers native worktree isolation (e.g. an `isolation: "worktree"` spawn option), you may use it instead — but you still need the resulting path and branch recorded in `PLAN.md`, because QA and resumed developers must land in the same worktree.

## Capacity probes (before spawning the team)

**Probe 1 — Confinement.** Create a temp worktree. Spawn a disposable agent with the confinement instructions above and have it write a marker file in the worktree and one in `.orchestration/`. Verify both landed where expected and nothing appeared in the main checkout. If confinement fails, stop and tell the user.

**Probe 2 — App boots per worktree.** In the temp worktree, run the bootstrap below and start the app on a non-default port. Confirm it responds. Discover commands from `package.json` / Makefile / README / CI config — assume nothing.

**Probe 3 — QA tooling.** Launch the project's e2e tooling (Playwright, Cypress, or whatever the repo uses) headless against the probe app. If nothing runs, agree on a fallback with the user now (API-level tests, scripted smoke) — not after development closes.

Clean up probe worktrees when done:

```bash
git worktree remove ../<probe-dir> --force
```

## Worktree bootstrap

A fresh worktree is only source code. Before handing it to an agent, for each worktree:

- **Dependencies**: install them (`node_modules` etc. aren't shared between worktrees).
- **Env files**: copy gitignored `.env*` files from the main checkout.
- **Port**: assign a unique port per feature (e.g. base + feature ID) and record it in `PLAN.md`.
- **Database**: give each feature its own database or schema (e.g. `app_f03`) and run migrations. Parallel QA against one shared DB will corrupt each other's state.
- Any other shared resource (queues, caches, upload dirs): namespace per feature or document the collision risk.

## Serial merge discipline

Development is parallel; **integration is serial**. One feature at a time:

1. Confirm user sign-off on the feature (never merge without it).
2. Bring mainline into the feature branch inside its worktree. **Conflicts go to the feature's developer** (resume by ID) — the PM does not edit code.
3. Re-run the feature's smoke checks. If behavior changed relative to what the user signed off, show the user again.
4. Merge into mainline (fast-forward when possible).
5. Spawn DevOps for the post-merge check (build, tests, release path).
6. Only then start the next feature's merge.

Merging two features at once makes any breakage unattributable. Serial merges keep each integration's blast radius diagnosable.

After merge + verification, remove the worktree, drop its feature DB if disposable, and mark the feature closed in `PLAN.md`. Also remove worktrees of abandoned features.
