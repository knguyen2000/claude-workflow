# /init

Bootstrap the claude-workflow plugin in a repo. Scans the codebase, asks targeted questions, generates config and docs.

**Run once per repo.** If `.claude/workflow.config.json` already exists, warn the dev and confirm before overwriting.

## Prerequisite: Visibility

Ask the developer:

> **Commit workflow files to the repo, or keep local-only?**
> - **Public** — committed and PR'd in. Collaborators see them.
> - **Local only** — generated on disk, gitignored, never staged/pushed. Use on established repos where new committed files aren't appropriate.

Store as **visibility mode** — carries through Steps 0, 4, 5, 6, 8.

## Step 0: Git setup

1. Not a git repo? `git init` + initial commit of existing files.
2. Ensure on `main`/`master` — the only commit allowed directly on main is this bare bootstrap.
3. If a remote exists: `git fetch origin && git pull` before branching. If `pull` fails, stop and ask the dev to resolve.
4. `git checkout -b chore/init-workflow` — all changes happen here, never on main.
5. After Step 8 commits, tell the dev to PR and merge. No remote yet? Note they'll need one first.

## Step 1: Scan the codebase

| Signal | Detect |
|---|---|
| Language | `requirements.txt`/`pyproject.toml`/`setup.py`→Python, `package.json`→JS/TS, `Cargo.toml`→Rust, `go.mod`→Go, `pom.xml`/`build.gradle`→Java |
| Framework | Read the manifest (streamlit, flask, fastapi, next, express, etc.) |
| Test runner | `pytest.ini`, `pyproject.toml [tool.pytest]`, `jest.config.*`, `vitest.config.*`, `Cargo.toml` |
| Linter | `ruff.toml`, `.eslintrc*`, clippy, golangci-lint |
| Formatter | ruff, `.prettierrc*`, `rustfmt.toml` |
| Type checker | `mypy.ini`, `tsconfig.json` |
| Project type | HTML/templates+web framework→`web`; route handlers, no frontend→`api`; `bin/`/CLI entry→`cli`; else→`library` |

Also check: does `.claude/workflow.config.json`/`CLAUDE.md`/`CONTRIBUTING.md`/`.gitignore` already exist (read if so), top-level directory structure.

## Step 2: Confirm and fill gaps

Present detected values, ask only what can't be auto-detected:

```
## Detected
Language: Python 3.11+ · Framework: Streamlit · Test: pytest · Lint: ruff · Type: web
Source dirs: app.py, agents/, components/, engines/, services/, config/, pages/

## Need from you
1. Project name?
2. Project type correct? (web/api/cli/library)
3. Import restrictions? (e.g. "don't import X in Y")
4. Centralized state management module?
5. Multi-file registration requirements? (e.g. "adding a route touches 3 files")
6. Web: pages/routes for browser testing?
7. Secrets files to gitignore? (.env, secrets.toml, etc.)
```

Wait for answers before proceeding.

## Step 3: Generate workflow.config.json

Follow the schema at `schema/workflow-config.schema.json`. Always include `project_name`, `project_type`, `commands`, `paths`, `naming`, `git`. Include `architecture` only if layers/restrictions/registration points were given; `inspect` only if `web`; `sweep` only if a dead-code tool + requirements file were detected; `docs` only if folder-level docs exist.

## Step 4: Generate or update CLAUDE.md

**If missing**, create with this structure (fill bracketed parts from the scan/answers):

```markdown
# [Project Name]

## Quick Start
[install + run commands]

## Stack
[language, framework, key deps]

## Architecture
[top-level folders + purpose — ask dev to verify]

## Key Files
[entry point, config, state module, other dev-named files]

## Quality Tools
[test, lint, format commands]

## Conventions
- All new features use TDD: write failing tests first
- [state management / import restriction / registration point rules if configured]
- UI changes work at mobile, tablet, desktop viewports (web only)

### Extensibility and Modification Policy
Write code that can be extended without rewriting — hooks, registries, strategy/plugin patterns, config-driven behavior.

**Extend, don't modify.** Default to adding new functions/modules over changing existing lines.

**Before touching an existing file**, read the full function and its callers (if public) — never modify code you haven't read.

**When modification is unavoidable:**
1. Justify — confirm no extend-only path exists
2. Read the full file + 2-3 callers
3. Scope — minimum lines necessary
4. Impact analysis — grep all callers/importers
5. Test all affected paths, not just the new feature; add regression tests for uncovered dependents
6. Note in commit body why modification was necessary

### Task Journal
Every task gets `JOURNAL-<branch-name>.md` in the project root (gitignored) — helps the dev follow how a feature was built and enables `/resume` after context loss.

**Write (append) at:** story completion, non-obvious design decisions, resolved unexpected problems, session end (handoff entry).
**Read:** only at session start, to recover context — never mid-session.
**Context pressure:** write the handoff entry immediately if context may run out soon.

## Doc Sync
After changing code in a folder with README.md/DESIGN.md, make targeted edits to the affected sections before committing.

## Git Workflow
Triggered by "commit", "ship it", "push", etc.

**Commits:** `<type>(<scope>): <summary>` — types: [config git.commit_types]. Subject under [config git.max_subject_length] chars, lowercase, no period, imperative mood. Body explains why, only if needed.

**Branches:** `<type>/<short-slug>`

**PRs:** title matches commit subject. Body: `## Summary` / `## Changes` / `## Test Plan`.

**Commit and PR workflow:** if SPEC.md exists, read for context then delete it → verify /conform and /preflight passed → stage explicit files (never `git add .`) → commit, push, create PR → report URL, do NOT merge.

Never force-push, never merge without approval, never push directly to main.

## Don'ts
- Don't commit secrets or API keys
- Don't add dependencies without checking deployment compatibility
- [import restriction don'ts if configured]
```

**If CLAUDE.md exists:** don't overwrite — append only missing sections (Extensibility Policy, Task Journal, Git Workflow, Doc Sync), then show the dev what was added.

**Local-only + CLAUDE.md exists:** it's already tracked, so `.gitignore` can't hide edits to it. Ask the dev: commit the additions via the PR (visible to collaborators), or skip and apply manually later.

## Step 5: Generate or update CONTRIBUTING.md

**If missing**, create with: Getting Started (install/run), a "Working with Claude Code" section explaining the describe→build→review loop, the command table (below), a typical-workflow list (TDD → implement → `/inspect` light → dev tests → `/retest` if bugs → commit → `/conform`+`/preflight` → PR → review/merge → `/postmerge`), `/resume` note, a "done means" checklist (tests pass, `/conform` clean, `/preflight` clean, `/inspect` passed for web, manual test confirmed, modified-code justified, docs updated, PR has Summary/Changes/Test Plan), and an "Extend, don't modify" pointer to CLAUDE.md.

Command table:

| Command | When to use |
|---|---|
| `/kickoff` | Start work — paste tasks, get a SPEC, serial/parallel/hybrid recommendation |
| `/hotfix` | Single urgent fix — same checks, less ceremony |
| `/commit` | Stage manual changes, generate message, commit, push |
| `/resume` | Continue from a previous session |
| `/preflight` | Pre-PR gate (`recheck` re-runs only failed checks) |
| `/postmerge` | After merging a PR — verify main is healthy |
| `/retest` | After a bug found in manual testing |
| `/conform` | Check changes match codebase patterns |
| `/docsync` | Audit folder-level docs for staleness |
| `/inspect` | Browser-level check (web only) — light by default |
| `/sweep` | Cleanup — dead code, debug artifacts, gitignore gaps |
| `/audit-workflow` | Periodic — config health, adherence, friction |

**If CONTRIBUTING.md exists:** append the "Working with Claude Code" section only if absent. Never overwrite existing content.

**Local-only + CONTRIBUTING.md exists:** don't append (already tracked) — instead create `CONTRIBUTING.claude.md` with that section only; gitignored in Step 6.

## Step 6: Update .gitignore

Append missing patterns only (no duplicates).

**Public mode:**
```
# Planning artifacts (workflow plugin)
/SPEC.md
/TODO.md
/TASKS.md
JOURNAL-*.md
```

**Local-only mode:**
```
# claude-workflow (local-only — machine-local files)
/.claude/workflow.config.json
/CLAUDE.md
/TASKS.md
/CONTRIBUTING.claude.md
/SPEC.md
/TODO.md
JOURNAL-*.md
```
Add `/CONTRIBUTING.md` too if it was newly created this run.

Note: `.gitignore` only stops **untracked** files from being staged — already-committed files (like an existing CONTRIBUTING.md) can't be hidden this way, hence `CONTRIBUTING.claude.md` as the workaround.

## Step 7: Create TASKS.md

```markdown
# Tasks

Paste your tasks below in any format. Then run `/kickoff`.

## This round
Dump everything you're considering — bugs, features, ideas, any format (Issues, tickets, Slack, plain text).



## How many this session
Pick a realistic number (1-3 typical) — Claude will prioritize and plan just that subset.

> Example: "2" or "the first 3" or "just the auth bug"
```

## Step 8: Summary

Report what was created/updated. Structure:

```
## Initialized claude-workflow [(local-only)]

### Created
[public: .claude/workflow.config.json, CONTRIBUTING.md (or updated), TASKS.md]
[local-only: same files, generated locally and gitignored — note "only .gitignore itself is committed"]

### Updated
[CLAUDE.md sections added, .gitignore patterns added]

### Next steps
1. Review generated files on `chore/init-workflow`
2. [public: commit, push, PR to merge] [local-only: PR+merge chore/init-workflow so .gitignore lands on main]
3. After merge — review CLAUDE.md's Architecture/Key Files
4. Paste tasks into TASKS.md, run /kickoff
```

Local-only: add a closing note that removing the claude-workflow block from `.gitignore` later makes the files public.

## Rules

- Never overwrite existing content — always extend
- Always show what was auto-detected and let the dev correct it before generating
- Skip a file's workflow sections if they already exist (no duplicates)
- Generated CLAUDE.md is a starting point — tell the dev to refine Architecture with specifics
