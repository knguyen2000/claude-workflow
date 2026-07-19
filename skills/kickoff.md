# /kickoff

Intake raw tasks from the backlog, clarify requirements, generate SPEC.md, recommend a workflow.

## Config

Read `.claude/workflow.config.json`. Missing? Stop and tell the dev to copy `templates/workflow-config.example.json` and adapt.

## Prerequisite: Sync main

```
git checkout main
git pull
```

Uncommitted changes on main? Stop, ask the dev to resolve first — never plan or branch from a stale or dirty main.

All planning happens on main. `SPEC.md`/`TASKS.md` are gitignored — no tracked changes during planning.

## Input

Dev pastes raw tasks into `TASKS.md` (any format — Issues, Linear, Jira, Slack, plain text), noting how many to tackle this round. Missing? Ask them to create it.

## Step 0: Size check

Single, clearly small task (bug, config change, <5 files, one story) in `TASKS.md`? Suggest `/hotfix` instead — same checks, faster on-ramp. Confirmed → stop, run `/hotfix`. Declined or multiple tasks → continue.

## Step 1: Parse and understand

For each task in `TASKS.md`, extract: the goal, acceptance criteria, technical constraints, dependencies between tasks (explicit or implied).

## Step 2: Identify gaps

Compare each task against the SPEC template. Flag: unclear scope, missing acceptance criteria, ambiguous impact, unknown technical constraints, unstated dependencies.

**Never assume answers.** Ask the dev, questions grouped by task and numbered. Wait for answers.

## Step 3: Generate SPEC.md

Once gaps are filled, follow the plugin's SPEC template: goal per task, user stories with verifiable acceptance criteria, "files likely touched" per story (scan the codebase), Modification Risk Assessment (extend-only vs modify+extend per story), technical notes, out of scope, Phases section if any task is 15+ files.

## Step 4: Independence analysis

For each pair of this round's tasks, check: overlapping "files likely touched"? import dependencies between them? shared state/config keys? shared UI (same page/component)?

Report as an independence matrix; recommend **Serial**, **Parallel**, or **Hybrid**.

## Step 5: Confirm and start

Present SPEC.md + recommendation. Wait for approval before implementing.

Once approved, create `JOURNAL-<branch-name>.md` per task (gitignored) — task name, branch, SPEC reference, story checklist.

## Step 6: Re-sync before branching

Steps 1-5 (Q&A, approval) take time — main may have moved since the Prerequisite sync.

```
git fetch origin
git log HEAD..origin/main --oneline
```

Moved? `git pull` (tree should still be clean — no implementation started yet). Not clean? Stop, ask the dev to resolve. Only proceed to Execution once this passes.

---

## Worktree Setup (parallel/hybrid only)

Gitignored files (secrets, config) don't exist in new worktrees — after creating one, copy required config files per `.gitignore`. Serial doesn't use worktrees.

---

## Per-Task Cycle

Every task, any workflow, follows this cycle:

```
implement (TDD, extend-first) → commit per story
  │
  /inspect (light) [web only — skip cli/api/library]
  │  fix issues found, commit fixes
  │
  ← dev manual tests (business logic, UX, edge cases —
  │  Claude already verified it runs and renders)
  │
  if dev found bugs: /retest → fix → /inspect again (only then)
  │
  /conform (on committed diff)
  │  fix conformance issues, commit fixes
  │
  /preflight (pre-PR gate — all checks incl. regression scope)
  │
  done — ready for PR
```

`/conform` runs post-commit; `/preflight` runs last. `/preflight` fails → fix → `/preflight recheck` (failed checks only).

`/inspect` defaults to light mode (web only); use `full` before a UI-heavy branch's first PR or on request; re-run only if bugfixes changed code.

**Extend-first:** prefer new functions/modules. If modification is needed, follow the Modification Protocol — read code + callers first, justify, minimize scope, impact analysis, test all affected paths.

---

## Execution: Serial

```
main (up to date)
  │
  git checkout -b <type>/task-1 → [per-task cycle] → push → PR #1
  │  ← dev reviews + merges
  git checkout main && git pull → /postmerge
  │
  git checkout -b <type>/task-2 → [per-task cycle] → push → PR #2
  │  ← dev reviews + merges
  git checkout main && git pull → /postmerge
  │
  (repeat per task)
```

Each task starts from a fresh, verified main. Never stack branches. Never start the next task until `/postmerge` confirms main is healthy.

## Execution: Parallel / Hybrid

**Parallel:** one worktree per task. **Hybrid:** one worktree per independent group; tasks within a group run serially (`git checkout -b <type>/group-a-task-2` from task-1's branch, etc.).

```
main (up to date)
  │
  ── Branch out ──────────────────────────────────
  git worktree add ../task-N -b <type>/task-N   (one per task, or per group for hybrid)
  + copy gitignored config (Worktree Setup)

  ── Implement ───────────────────────────────────
  Each agent in its worktree: [per-task cycle]
  (hybrid: tasks within a group run serially, same worktree)

  ── Sequential integration test ─────────────────
  Simulate real merge order on a throwaway branch — never pushed:
  git checkout main && git checkout -b test/integration

  For each branch in planned merge order:
    git merge <type>/task-N
    Run test command
    Smoke test (web: growing set of merged features each step)
  Final step: full manual test of everything together + /inspect light (web)

  Failure → the issue is between that task and what's already merged.
  Fix on the failing branch, re-run integration from the top.

  ── Create PRs ───────────────────────────────────
  One PR per task (hybrid: per task, not per group).
  Report all PR URLs with recommended merge order.

  ── Merge with post-merge verification ──────────
  For each PR in order:
    dev reviews + merges → /postmerge → only then proceed to the next
    (before merging, rebase the next PR's branch onto updated main → retest)
  (parallel: strictly sequential; hybrid: groups can interleave if they don't conflict)

  ── If rebase conflicts ─────────────────────────
  Resolve on the conflicting branch → run tests → /inspect light only if UI
  files conflicted (web) → resolution commit (no force-push) → push →
  dev re-reviews before merging.

  ── Cleanup ──────────────────────────────────────
  git worktree remove ../task-N (each)
  git branch -D test/integration
```

## Rules

- Always `git pull` on main before planning or branching
- Never edit main directly — all work happens on feature branches
- Never assume missing information — always ask the dev
- Never start implementing before the dev approves the SPEC and workflow
- Never skip independence analysis, even if the dev says "just do them in parallel"
- Never merge PRs without dev approval — Claude creates PRs, dev merges
- Never merge the next PR until `/postmerge` passes on main (all workflows)
- Never push or PR `test/integration` — local and throwaway
- Dev overrides the recommendation (e.g. forces parallel on conflicting tasks)? Warn about the specific conflicts, proceed if they insist
- **Extend, don't modify** — prefer new code over changing existing lines; if unavoidable, follow the Modification Protocol (read first, justify, minimize scope, impact analysis, test all affected paths)
