# /postmerge

Verify main is healthy after a PR merge. Run after every merge, before merging the next — catches integration issues per-branch testing misses.

## Config

Read `.claude/workflow.config.json` for commands and paths.

## When

After merging any PR (all workflows — serial, parallel, hybrid). Only skippable exception: final PR in a round with nothing queued next (recommended, not required).

## Steps

1. **Sync main** — `git checkout main && git pull`
2. **Tests** — `commands.test`, all must pass; a failure means the merge regressed something — fix before the next merge.
3. **Lint/format** — `commands.lint` + `commands.format_check`, zero issues (conflict resolutions/rebases can introduce these).
4. **Imports** — `commands.import_check` if configured — catches breaks when two PRs independently add/rename modules.
5. **Conformance spot-check** — duplicate imports across branches, naming conflicts between branches' new functions, lint/format clean. Fix any issues found.
6. **Browser verification** (web only) — `/inspect` light on main: affected pages + home, console errors, smoke test. Most likely check to catch real integration issues (independently-passing features breaking each other's UI).
7. **Feature spot-check** — for every PR merged so far this round, confirm its primary feature/golden path still works and wasn't broken by later merges. Quick manual check, not a full retest.

## Output

Merged PRs verified, checks table (Pass/Fail), verdict: **Main is healthy — safe to merge next PR** or **Main has issues — fix before merging PR #N**.

## If issues found

1. Identify which merge caused it (most recent, or a combination).
2. Don't revert unless the dev asks — instead `git checkout -b fix/postmerge-<issue>`.
3. Fix via the standard cycle (TDD, extend-first, `/preflight`).
4. PR the fix, merge, re-run `/postmerge` to confirm.
5. Only then continue merging remaining PRs.

## Rules

- Required after every PR merge, every workflow — every workflow can break main
- Never merge the next PR until `/postmerge` passes
