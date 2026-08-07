# Worktrees, probes, and merge discipline

## Why worktrees

Parallel developer leads editing the same checkout will trample each other. Each lead gets its own git worktree on its own branch; DevOps gets one too. The PM stays on the main checkout and never edits code.

```bash
git worktree add ../<repo>-f<ID> -b feature/<ID>-<slug>
```

Use a consistent naming scheme so worktrees are identifiable at a glance (`../myrepo-f03`, branch `feature/03-user-invites`).

## Capacity probes (run these before spawning the team)

This skill runs in unfamiliar repos, so verify the two assumptions everything depends on:

**Probe 1 — Worktree inheritance.** Spawn a disposable lead-style agent inside a temp worktree and have it spawn a sub-agent that writes a marker file. Verify the marker landed in the worktree, not the main checkout. If sub-agents escape the worktree, isolation is fiction — stop and tell the user before proceeding.

**Probe 2 — QA tooling.** Spawn a disposable agent that attempts to launch the project's e2e tooling (Playwright, Cypress, or whatever the repo uses — check `package.json` / CI config to find out). If nothing runs headless, QA needs a fallback plan (manual smoke script, API-level tests); agree on it with the user now, not after development closes.

Clean up probe worktrees when done:

```bash
git worktree remove ../<probe-dir> --force
```

## Serial merge discipline

Development is parallel; **integration is serial**. One feature at a time:

1. Confirm user sign-off on the feature (never merge without it).
2. Rebase or merge mainline into the feature branch inside its worktree; resolve conflicts there.
3. Run the mainline's own checks after merging (build, tests, smoke).
4. Only then start the next feature's merge.

The reason: merging two features simultaneously makes any breakage unattributable. Serial merges keep each integration's blast radius diagnosable.

After merge + verification, the worktree can be archived/removed and the feature marked closed in `PLAN.md`.
