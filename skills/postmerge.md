# /postmerge

Verify main is healthy after a PR merge. Run this after every PR merge before merging the next one. Catches integration issues that per-branch testing misses.

## Config

Read `.claude/workflow.config.json` for project-specific commands and paths.

## When to use

- After merging any PR into main (required for all workflows — serial, parallel, hybrid)
- The only exception: final PR in a round with no more tasks queued (recommended but dev may skip)

## Steps

### 1. Sync main

```
git checkout main
git pull
```

### 2. Test suite

Run the project's full test command (config `commands.test`). All tests must pass. If any fail, the merge introduced a regression — investigate and fix before merging the next PR.

### 3. Lint and format check

Run the project's `commands.lint` and `commands.format_check`. Zero issues required. A merge can introduce lint/format issues via conflict resolutions or rebase artifacts.

### 4. Import check

Run the project's `commands.import_check` (if configured). Catches broken imports when two PRs independently add/rename modules.

### 5. Conformance spot-check

Run a quick pattern check on the merged result — two conforming branches can produce a non-conforming merge:
- Check for duplicate imports (both branches added the same import)
- Check for naming conflicts (both branches added similarly-named functions)
- Run lint and format check — merge can introduce issues via conflict resolution artifacts
- If issues found, they must be fixed before proceeding

### 6. Browser verification (web projects only)

If config `project_type` is `web`, run `/inspect` (light mode) on main:
- Page loads for affected pages + home
- Console errors
- Interactive smoke test

This is the check most likely to catch real integration issues — two features can both pass tests independently but break each other's UI.

### 7. Feature spot-check

For each PR that has been merged into main so far in this round, verify its primary feature still works:
- Navigate to the page or trigger the feature
- Confirm the expected behavior is present
- Check that it hasn't been broken by subsequent merges

This is a quick manual verification, not a full retest — just confirm each feature's golden path.

## Output

Report merged PRs verified, checks table (Pass/Fail), and verdict: **Main is healthy — safe to merge next PR** or **Main has issues — fix before merging PR #N**.

## If issues are found

1. **Identify which merge caused it** — was it the most recent merge, or a combination?
2. **Do NOT revert unless the dev asks** — instead, create a fix branch from main:
   ```
   git checkout -b fix/postmerge-<issue>
   ```
3. Fix the issue using the standard per-task cycle (TDD, extend-first, `/preflight`)
4. PR the fix, merge, then re-run `/postmerge` to confirm
5. Only then continue merging remaining PRs

## Rules

- **`/postmerge` is required after every PR merge** — serial, parallel, and hybrid. Every workflow can break main; every workflow gets the gate.
- Never merge the next PR until `/postmerge` passes
