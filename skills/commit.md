# /commit

Stage all local changes, generate a commit message from the diff, commit, and push.

Use this when you made changes yourself and want to ship them without writing the commit message manually.

## Step 1: Safety checks

Run `git status` and `git branch --show-current`.

**If on `main` or `master`:** Do not stop — instead, run Steps 2 and 3 first to understand the changes and detect conventions, then come back here to suggest a branch name based on what changed. Ask the dev:

```
You are on main. It is not safe to commit directly here.

Suggested branch: feat/short-description-of-change

Create this branch and continue? [yes / use a different name / cancel]
```

If approved, run `git checkout -b <branch-name>` and continue. If the dev provides a different name, use that instead. If cancelled, stop.

**If no changes exist** (clean working tree): stop and tell the dev there is nothing to commit.

## Step 2: Detect conventions

Run `git log --oneline -20` and `git branch -a` to detect both commit message and branch naming conventions.

**Branch naming** — look for a pattern in existing branch names:

| Pattern | Convention |
|---------|-----------|
| `feat/`, `fix/`, `chore/` prefixes | type/slug |
| `feature/`, `bugfix/`, `hotfix/` prefixes | longer type/slug |
| `username/description` | owner/slug |
| No pattern | Use `type/short-slug` as default |

Use the detected branch convention when suggesting a branch name in Step 1. The slug should reflect the actual changes — short, lowercase, hyphen-separated words.

**Commit message** — look for a consistent pattern across the last 20 commits:

| Pattern | Convention |
|---------|-----------|
| `type(scope): description` | Conventional Commits (with scope) |
| `type: description` | Conventional Commits (no scope) |
| `[TAG] description` | Bracket tags |
| No clear pattern | Use Conventional Commits as default |

Note what was detected. Apply it when writing the message in Step 4.

**Conventional Commits types:**

| Type | When to use |
|------|------------|
| `feat` | new feature or new behavior |
| `fix` | bug fix |
| `docs` | documentation only |
| `refactor` | code change with no behavior change |
| `test` | add or update tests |
| `chore` | config, build, tooling, dependencies |
| `style` | formatting only, no logic change |
| `perf` | performance improvement |

## Step 3: Read the changes

Run `git diff HEAD` for unstaged changes and `git diff --staged` for staged changes.

Check `git status` for untracked files. If any untracked files look related to the changes (same directory, same feature name), list them and ask the dev if they should be included before staging.

Scan the diff for secrets patterns (API keys, tokens, passwords, private keys). If any are found, list the file and line, and stop — do not proceed to commit.

Understand what changed across all files before writing anything.

## Step 4: Write the commit message

Write one commit message that covers all the changes. Rules:

- **Subject line:** under 72 characters, imperative mood (`add`, `fix`, `update` — not `added` or `adds`)
- **Language:** simple and direct — short words, no idioms, no analogies; technical terms are fine
- **Scope:** use the most specific part of the codebase that was changed (file name, module, command name, etc.)
- **Body:** include only if the subject line cannot capture the full intent; 2–3 lines max, explain *why* not *what*
- **Multiple unrelated changes:** split into separate commits (ask the dev first)

## Step 5: Confirm with the dev

Show the proposed message and the files that will be staged:

```
## Files to stage
- modified: path/to/file.py
- modified: path/to/other.py

## Commit message
type(scope): short description

Optional body line explaining why.

Proceed? [yes / edit / cancel]
```

Wait for the dev to confirm, provide an edited message, or cancel.

## Step 6: Stage, commit, push

Stage each file explicitly by name — never use `git add .` or `git add -A`.

Commit with the confirmed message.

Push to the current branch's upstream. If no upstream is set, push with `git push -u origin <branch-name>`.

Report the commit hash and confirm the push succeeded.

## Step 7: Create PR

Generate a PR title and body from the commit message and diff, then create the PR with `gh pr create`.

**Title:** same as the commit subject line.

**Body:**

```
## Summary
- [1–3 bullet points describing what changed and why]

## Changes
- [list of files or areas changed, one line each]

## Test Plan
- [ ] [specific thing to verify manually]
- [ ] [another check if needed]
```

Keep all sections short. Use the same simple, direct language as the commit message.

Create the PR against `main` (or `master`). Report the PR URL. Do not merge.

If `gh` is not available or not authenticated, print the PR title and body as text so the dev can create it manually on GitHub.
