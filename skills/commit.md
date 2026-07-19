# /commit

Stage local changes, generate a commit message from the diff, commit, push. Use when you made the changes yourself and want to ship without writing the message by hand.

## Step 1: Safety checks

`git status` + `git branch --show-current`.

**On `main`/`master`:** don't stop — run Steps 2-3 first (understand changes, detect conventions), then suggest a branch name:

```
You are on main. It is not safe to commit directly here.
Suggested branch: feat/short-description-of-change
Create this branch and continue? [yes / use a different name / cancel]
```

If approved, check whether local main is behind remote first (don't `git pull` directly — uncommitted changes are already on main and a pull could conflict):

```
git fetch origin
git log HEAD..origin/main --oneline
```

Commits shown? Warn: `Local main is N commits behind origin/main. Continue anyway, or stash changes, pull, then reapply? [continue / stash-and-pull]`. Stash-and-pull → `git stash && git pull && git stash pop`, then proceed.

`git checkout -b <branch-name>` (or the dev's name). Cancelled → stop.

**Clean tree:** stop, tell the dev there's nothing to commit.

## Step 2: Detect conventions

`git log --oneline -20` + `git branch -a`.

**Branch naming:** `feat/`/`fix/`/`chore/` → type/slug; `feature/`/`bugfix/`/`hotfix/` → longer type/slug; `username/description` → owner/slug; no pattern → default `type/short-slug`. Use this for Step 1's suggestion — slug reflects the actual change, short/lowercase/hyphenated.

**Commit format:** `type(scope): description` or `type: description` → Conventional Commits; `[TAG] description` → bracket tags; none → default Conventional Commits. Apply in Step 4.

**Types:** `feat` new feature · `fix` bug fix · `docs` docs only · `refactor` no behavior change · `test` test-only · `chore` config/build/tooling · `style` formatting only · `perf` performance.

## Step 3: Read the changes

`git diff HEAD` + `git diff --staged`. Check `git status` for untracked files related to the change (same dir/feature name) — ask if they should be included.

**Exclude local-only files** — never stage or suggest staging files that are machine-specific setup, even if untracked and sitting next to the change: `.env`/`.env.*`, `*.local.*`, `.claude/settings.local.json`, IDE folders (`.vscode/`, `.idea/`), OS files (`.DS_Store`, `Thumbs.db`), or anything the dev has described as personal/local config. If a file's purpose is ambiguous, ask the dev explicitly — don't stage it by default.

Scan the diff for secrets (API keys, tokens, passwords, private keys) — found any → list file:line and stop, don't commit.

Understand all changed files before writing anything.

## Step 4: Write the commit message

One message covering all changes:
- **Subject:** <72 chars, imperative mood (`add`/`fix`/`update`, not `added`/`adds`)
- **Language:** simple, direct, no idioms — technical terms fine
- **Scope:** most specific codebase area changed
- **Body:** only if the subject can't capture full intent; 2-3 lines max, explains *why*
- **Unrelated changes:** split into separate commits (ask first)

## Step 5: Confirm with the dev

```
## Files to stage
- modified: path/to/file.py

## Commit message
type(scope): short description

Optional body line explaining why.

Proceed? [yes / edit / cancel]
```

Wait for confirmation, an edited message, or cancel.

## Step 6: Stage, commit, push

Stage each file explicitly by name — never `git add .` or `git add -A`. Commit. Push to upstream (`git push -u origin <branch-name>` if none set). Report the commit hash and confirm the push.

## Step 7: Create PR

`gh pr create` with title = commit subject and body:

```
## Summary
- [1-3 bullets: what changed and why]

## Changes
- [files/areas changed, one line each]

## Test Plan
- [ ] [thing to verify manually]
```

Keep sections short, same plain language as the commit message. Target `main`/`master`. Report the PR URL — don't merge.

No `gh` or not authenticated → print title+body as text for the dev to create manually.

## Rules

- **Never commit without the dev's explicit confirmation.** Step 5's "Proceed?" gate is mandatory every time this command runs — no confirmation, no commit, no exceptions.
- **Never stage or commit local-only files.** Machine-specific setup files (see Step 3) stay out of every commit by default, regardless of how they ended up in the working tree. Only include one if the dev explicitly names it and confirms.
