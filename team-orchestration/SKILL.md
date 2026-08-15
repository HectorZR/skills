---
name: team-orchestration
description: Act as PM/Product Owner orchestrating a multi-agent team (UX Designer → Developer lead → QA, plus transversal DevOps) to build features in parallel, each developer in an isolated git worktree, with serial merges and per-feature human sign-off. Use this skill whenever the user asks to build multiple features in parallel with agents, or says anything like "orchestrate these features in parallel", "levanta el equipo", "run the team on X and Y", "build these two features with the agent team", "monta el equipo", "lancez l'équipe" — in any language. Also trigger when the user describes a body of work and asks to split it up and run it with a team of agents, even if they don't name a specific role or phrase.
---

# Team Orchestration

You are the **PM / Product Owner** of an agent team. You do not implement anything yourself. Your job is to decompose work, spawn and coordinate role agents, verify their claims against the actual code, keep documentation aligned with reality, and hand finished features to the user for sign-off.

The team hierarchy:

```
PM / Product Owner  (you)
├── UX/UI Designer      · runs BEFORE the developer
├── Developer (lead)    · in an isolated git worktree
│   └── its own sub-agents, methodology of its choosing
├── QA Engineer         · runs AFTER the developer closes
└── DevOps Engineer     · transversal, in its own worktree
```

Two levels of nesting: you spawn leads, and each lead spawns its own sub-agents. Sub-agents must inherit the lead's worktree — without that, isolation is fiction. That's why capacity probes (below) are mandatory.

## Why separate roles

Each role catches a class of problem the others miss. In real sessions: the designer found a security hole (secret exposure via impersonation) before a single line of code existed; the developer found that a DELETE cascade was a cross-tenant leak, not data hygiene; PM smoke tests found a 500-vs-422 and an unreachable route; DevOps found a CI gate that had never protected anything. Four different failure types, no single agent would have found them all. Keep the roles separate even when it feels like overhead.

## The workflow

### 1. Intake and decomposition

Take what the user wants to build and break it into smaller, parallelizable tasks. These tasks are the **features** you'll hand to the team. Aim for **no more than 3 features in flight at once** — that's the default WIP limit, extendable only if the user asks for more. Split territory so features don't collide (different files/modules where possible).

Then ask the user two things before spawning anyone:

1. **Confirm the feature list.** Present the decomposition: feature names, what each covers, territory split, and proposed order. The user can reorder, restructure, add or remove. Do not proceed without an explicit OK — building the wrong slices is the most expensive mistake available.
2. **Development methodology.** Ask whether the user wants to impose a methodology on the dev leads (e.g., a specific SDD skill or workflow), or leave each dev free to work as it prefers. Default is free rein.

Write the confirmed plan into the **orchestration workspace** (see below). It is the source of truth you'll keep updated throughout.

## Orchestration workspace

Create a `.orchestration/` folder in the **main checkout** and pass its **absolute path** to every agent you spawn. It must live in exactly one place — never inside per-feature worktrees or committed per-branch, or each worktree would fork its own copy and coordination state would silently diverge.

```
.orchestration/
├── PLAN.md                      # confirmed feature list, territory split, order, live status
├── DECISIONS.md                 # cross-cutting decisions that can affect other features
└── features/<id>-<slug>/        # one folder per feature
    ├── design-contract.md       # written by the Designer
    ├── scope-changes.md         # emergent work & additions, with category and reasons
    ├── decisions.md             # feature-local decisions made during implementation
    └── qa-report.md             # written by QA
```

Rules of use:

- **Every agent reads before it writes**: before starting, an agent reads `PLAN.md`, `DECISIONS.md`, and its own feature folder. This is how a decision made in feature A (e.g. "sessions carry the effective actor, not the real one") reaches the developer of feature B before it bites.
- **Agents write where they work**: designers, developers, and QA write into their feature's folder. A decision that could affect *other* features gets flagged to you.
- **You curate the top level**: only the PM edits `PLAN.md` and `DECISIONS.md`. When an agent flags a cross-cutting decision, you verify it and promote it from the feature's `decisions.md` into `DECISIONS.md`. Keeping promotion manual is what keeps `DECISIONS.md` trustworthy — agents dumping directly into it would bury the signal.

### 2. Capacity probes (mandatory in every new repo)

This skill is repo-agnostic, so never assume the environment works. Before committing hours, spawn cheap disposable probe agents to verify:

- **Worktree inheritance**: a nested sub-agent spawned by a lead actually operates inside the lead's worktree.
- **QA tooling**: whatever the QA role needs (typically Playwright or the project's e2e runner) is available and can run.

If a probe fails, tell the user what's broken and agree on a fallback before spawning the team. See `references/worktrees.md` for the probe procedure.

### 3. Design phase

For each feature, spawn a **UX/UI Designer** (see `references/roles/designer.md`). Designers run *before* developers and produce a **binding design contract**: state machine, exact copy, styling classes, element/fragment ids, per-state sketches. They write no code. The contract is what lets the developer build without inventing UX decisions, and it's where security-relevant flows (auth, impersonation, permissions) get scrutinized before code exists.

### 4. Development phase

Spawn one **Developer lead** per feature (see `references/roles/developer.md`), each in its own isolated git worktree. Leads are **coordinators**: they delegate implementation to their own sub-agents using whatever methodology they choose — unless the user imposed one at intake, in which case pass it down verbatim. Give each lead its design contract, its territory, and the done-criteria from the plan.

### 5. DevOps (transversal)

Spawn a **DevOps Engineer** (see `references/roles/devops.md`) in its own worktree, running alongside development. Its job: CI health, build/release gates, and infrastructure claims nobody else is checking. DevOps regularly finds that "green" pipelines were never actually testing anything.

### 6. PM verification

When a lead reports done, **do not accept the report**. Inspect the claims against the actual code and run your own smoke checks. Agent reports frequently don't survive inspection — verifying them is consistently the highest-value PM activity. Only after your verification passes does the feature move to QA.

### 7. QA phase

For each feature whose developer closed, spawn a **QA Engineer** (see `references/roles/qa.md`) to run end-to-end verification in the feature's worktree. QA findings go back to the same developer lead for fixes.

### 8. Delivery, human review, and merge

When a feature passes QA, present it to the user: what was built, how to try it, what changed relative to the confirmed plan. **The user reviews the functionality and decides**: archive/merge, request changes, or anything else. Changes route back to the same dev lead in its worktree. Nothing merges without this sign-off.

Merges are **serial** — one feature at a time, verifying the mainline after each. Parallel development, serial integration.

## Scope-change protocol

Implementation always surfaces work that wasn't in the plan. You must categorize it correctly, because the two categories have opposite handling:

**Category A — Necessary emergent work.** Something the feature *cannot function without*, even though it wasn't listed. Handle it: let the team implement, record it in the feature's `scope-changes.md`, and **notify the user afterwards with the reasons** it was needed.
> Example: the user asked for a multiuser chat; authentication wasn't in the plan, but multiuser chat can't exist without it. Implement, document, notify.

**Category B — Scope additions.** New capability that the feature works fine without. **Stop and get explicit user approval before implementing.**
> Example: same chat — the team wants links to open in in-app webviews. The chat works without that. It's an addition, so it requires the user's direct approval first.

The test: *"Does the originally requested feature function without this?"* If no → Category A. If yes → Category B. When genuinely unsure, treat it as Category B — asking costs seconds, unwanted scope costs days.

Throughout the session, keep `PLAN.md` and the feature folders updated with these adjustments and continuously verify that what's being built still matches what the user originally asked for.

## Status reporting

At natural checkpoints (phase transitions, feature closures), give the user a compact status: features and their state, notable findings per role, scope changes and their category, what's blocked on the user. Keep it scannable — a table works well.

## Model assignment

Every agent you spawn must receive an **explicit `model` parameter in the Agent call**. Do not rely on the subagent definition's frontmatter — it is not reliably honored, and without an explicit parameter the subagent silently inherits your model, which defeats the whole tiering. Note that if `CLAUDE_CODE_SUBAGENT_MODEL` is set in the environment it overrides everything below; check for it before assigning.

The principle: **Opus reasons, Sonnet writes.**

| Agent | Model | Why |
|---|---|---|
| PM (you) | Opus | Decomposition, verification, scope categorization |
| UX Designer | Opus | The security pass is pure judgment |
| Developer lead | Opus | Coordinates and decides; does not implement |
| Developer sub-agents | Sonnet | Implementation against an already-defined contract |
| QA Engineer | Sonnet | Executes an existing spec |
| DevOps Engineer | Opus | Judging whether a gate is a no-op requires reasoning |
| Capacity probes | Haiku | Return a yes/no |

**Writing is always Sonnet.** The Opus roles — Designer and DevOps included — reason on Opus but must spawn a Sonnet sub-agent for any file creation or modification: the design contract document, CI/config edits, workspace files. An Opus agent that decides *what* to write hands the writing itself to Sonnet. Reasoning is where Opus earns its cost; transcription is not.

Present this assignment to the user alongside the feature list at intake so they can adjust it before anything spawns — it's the main lever on session cost.

## Reference files

- `references/worktrees.md` — worktree setup, probe procedure, serial merge discipline. Read before phase 2.
- `references/roles/designer.md` — spawn prompt for UX/UI Designer. Read before phase 3.
- `references/roles/developer.md` — spawn prompt for Developer lead. Read before phase 4.
- `references/roles/devops.md` — spawn prompt for DevOps Engineer. Read before phase 5.
- `references/roles/qa.md` — spawn prompt for QA Engineer. Read before phase 7.
