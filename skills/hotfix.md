# /hotfix

Fast on-ramp for urgent or trivial fixes. Same quality bar as `/kickoff`, skips multi-task ceremony (independence analysis, workflow recommendation, journal).

## Config

Read `.claude/workflow.config.json` for commands and paths.

## When to use

Single task, describable in one sentence, fits in one session. Turns out larger than expected (>5 files, new behavior emerging, multiple stories)? Stop, switch to `/kickoff`.

## Step 1: Sync main

```
git checkout main
git pull
```

Dirty? Stop, ask the dev to resolve first.

## Step 2: Understand the fix

Ask for a one-sentence description if not given. What's broken? What's the expected behavior after?

## Step 3: Generate a minimal SPEC

`SPEC.md` with one story, using the plugin's SPEC template — tight, single story, must include Modification Risk Assessment. Present for approval, wait before implementing.

## Step 4: Branch and implement

SPEC approval may have taken a while — re-check main hasn't moved since Step 1:

```
git fetch origin
git log HEAD..origin/main --oneline
```

Moved? `git pull` (tree should still be clean — nothing implemented yet). Not clean? Stop, ask the dev to resolve.

```
git checkout -b fix/<short-slug>
```

Standard per-task cycle:

```
implement (TDD, extend-first)
  │
  /inspect (light) [web only]
  │  fix issues, commit fixes
  │
  ← dev manual tests
  │
  if bugs found: /retest → fix → /inspect again
  │
  /conform (on committed diff)
  │
  /preflight (full gate incl. regression scope)
  │
  done — ready for PR
```

Commit per meaningful unit (usually one commit; split if there's a distinct refactor step).

## Step 5: PR

Read `SPEC.md` for PR context, then delete it. Create PR against main (Summary/Changes/Test Plan template). Report URL — dev reviews and merges.

## Step 6: Post-merge verification

After merge, run `/postmerge` — especially important when the hotfix modified existing code.

## What hotfix does NOT skip

SPEC (minimal), TDD, extend-first policy, `/inspect` light, `/conform`, `/preflight`, dev approval before merge.

## What hotfix skips

Independence analysis, workflow recommendation (always serial/single branch), journal file (switch to `/kickoff` if it doesn't fit one session), multi-story SPEC structure.
