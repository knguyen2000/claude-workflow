# /resume

Recover context and continue work from a previous session using task journals.

## Steps

1. **Discover journals** — list `JOURNAL-*.md` in the project root, read only Status + Current State (not the full log). None → tell the dev, suggest `/kickoff`.

2. **Present options** — multiple journals:
   ```
   1. JOURNAL-feat-feature-name.md — Branch: feat/feature-name
      Status: 3/5 stories done · Current: Implementing story 4
   2. JOURNAL-fix-bug-name.md — Branch: fix/bug-name
      Status: 2/2 done, pending PR · Current: Ready for /preflight and PR
   ```
   Ask which to resume. Only one → proceed automatically.

3. **Restore branch** — `git checkout <branch> && git status`. Not local → `git fetch origin && git checkout -b <branch> origin/<branch>`. Neither exists → warn, journal may be stale.

4. **Check for rebase need** — `git fetch origin && git log HEAD..origin/main --oneline`. Main moved → warn:
   ```
   Main has N new commits since this branch was created.
   Recommend rebasing before continuing: git rebase origin/main
   May conflict if new commits touch files this branch modified.
   ```
   Wait for the dev's decision. If they rebase, re-run step 5 after.

5. **Verify current state** — run the test command, run lint, read the journal's Current State + latest Log entry.

6. **Present a continuation plan** — from the journal's Next section and unchecked Status items:
   ```
   ## Resuming: [task name]
   Branch: feat/xxx

   ### Completed
   - [x] Story 1 ... - [x] Story 2 ...

   ### Remaining
   - [ ] Story 3 ... (next up) - [ ] Story 4 ...

   ### Context from last session
   [Current State + last Log entry]

   Ready to continue with Story 3?
   ```
   Wait for approval before implementing.

## Rules

- Never start implementing without dev confirmation
- Journal says "blocked"/"waiting on dev" — surface it prominently
- Tests fail on checkout — report failures before proposing to continue (branch may need fixes from a merged dependency)
