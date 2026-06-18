# /init

Bootstrap the claude-workflow plugin in a new (or existing) repository. Scans the codebase, asks targeted questions, and generates all required config and docs.

**Run once per repo.** If already initialized (`.claude/workflow.config.json` exists), warn the dev and ask before overwriting.

## Step 0: Git setup

1. If the directory is not a git repo, run `git init` and create an initial commit with existing files.
2. Ensure you are on `main` (or `master`) — this is the only commit allowed directly on main: the bare repo bootstrap.
3. **Create a branch:** `git checkout -b chore/init-workflow`. All workflow files are generated on this branch — never directly on main.
4. After all files are generated and committed (Step 8), tell the dev to PR and merge the branch. If there's no remote yet, note that they'll need to add one before creating a PR.

## Step 1: Scan the codebase

Before asking any questions, gather what you can automatically:

### Language and framework detection
- Check for `requirements.txt`, `pyproject.toml`, `setup.py` → Python
- Check for `package.json` → JavaScript/TypeScript
- Check for `Cargo.toml` → Rust
- Check for `go.mod` → Go
- Check for `pom.xml`, `build.gradle` → Java
- Read the manifest to identify frameworks (streamlit, flask, fastapi, next, express, etc.)

### Toolchain detection
- **Test runner:** look for `pytest.ini`, `pyproject.toml [tool.pytest]`, `jest.config.*`, `vitest.config.*`, `Cargo.toml`
- **Linter:** look for `ruff.toml`, `.eslintrc*`, `clippy`, `golangci-lint`
- **Formatter:** look for `ruff` config, `.prettierrc*`, `rustfmt.toml`
- **Type checker:** look for `mypy.ini`, `tsconfig.json`

### Project type detection
- Has HTML/templates + a web framework → `web`
- Has route handlers but no frontend → `api`
- Has a `bin/` or CLI entry point → `cli`
- None of the above → `library`

### Existing state
- Does `.claude/workflow.config.json` already exist?
- Does `CLAUDE.md` already exist? If so, read it.
- Does `CONTRIBUTING.md` already exist? If so, read it.
- Does `.gitignore` already exist? If so, read it.
- What's the directory structure? (top-level folders)

## Step 2: Confirm and fill gaps

Present what you detected and ask the dev to confirm or correct:

```
## Detected
- Language: Python 3.11+
- Framework: Streamlit
- Test runner: pytest
- Linter: ruff
- Formatter: ruff format
- Project type: web
- Source dirs: app.py, agents/, components/, engines/, services/, config/, pages/

## Need from you
1. Project name? (used in journals and PR descriptions)
2. Is the detected project type correct? (web/api/cli/library)
3. Any import restrictions? (e.g., "don't import X in Y directories")
4. Centralized state management? (e.g., "all state goes through state.py")
5. Multi-file registration requirements? (e.g., "adding a route requires updating 3 files")
6. For web projects: list your pages/routes for browser testing
7. Any secrets files that must be gitignored? (e.g., .env, .streamlit/secrets.toml)
```

Skip questions where the answer is obvious from the scan. Only ask what can't be auto-detected.

Wait for answers before proceeding.

## Step 3: Generate workflow.config.json

Create `.claude/workflow.config.json` using the detected + confirmed values. Follow the schema at `schema/workflow-config.schema.json`.

Include only sections that are relevant:
- Always include: `project_name`, `project_type`, `commands`, `paths`, `naming`, `git`
- Include `architecture` only if the dev provided layers, import restrictions, or registration points
- Include `inspect` only if `project_type` is `web`
- Include `sweep` only if a dead-code tool and requirements file were detected
- Include `docs` only if folder-level docs exist in the repo

## Step 4: Generate or update CLAUDE.md

**If CLAUDE.md does not exist**, create it with this structure:

```markdown
# [Project Name]

## Quick Start

[Auto-detected: install command + run command]

## Stack

[Auto-detected: language, framework, key dependencies]

## Architecture

[Fill from directory scan — list top-level folders and their purpose.
Ask the dev to verify.]

## Key Files

[List entry point, config, state management module, and any other
files the dev called out during setup.]

## Quality Tools

[Auto-detected: test, lint, format commands]

## Conventions

- All new features use TDD: write failing tests first, then implement until they pass
- [State management rule if configured]
- [Import restrictions if configured]
- [Registration points if configured]
- UI changes must work at mobile, tablet, and desktop viewports (web projects only)

### Extensibility and Modification Policy

Write code that can be extended without rewriting. Use patterns (hooks, registries, strategy/plugin, configuration-driven behavior) that let future features plug in rather than fork existing logic.

**Extend, don't modify.** When working on an existing codebase, default to extending existing code rather than modifying it. Add new functions, new branches, new modules — don't rewrite working lines unless there's no alternative.

**Before touching any existing file**, read the full function (and its callers if modifying a public interface) to understand the current behavior, contracts, and assumptions. Never modify code you haven't read.

**When modification is unavoidable**, follow this protocol:
1. **Justify** — confirm there is no extend-only path (new function, wrapper, decorator, subclass, config entry)
2. **Read** — read the full file (not just the target function) and at minimum 2-3 callers of the modified code
3. **Scope** — keep the change to the minimum lines necessary
4. **Impact analysis** — grep for all callers, importers, and dependents of the changed code
5. **Test all affected paths** — not just the new feature, but every feature that touches the modified lines. If existing tests don't cover a dependent path, add a regression test before making the change.
6. **Note in commit body** — explain why modification was necessary and what was verified

### Task Journal

Every task gets a journal file: `JOURNAL-<branch-name>.md` in the project root (gitignored). This serves two purposes: helps the dev understand how a feature was built without reading every diff, and enables session recovery when context runs out.

**Format:**
[Include the full journal format template]

**When to WRITE (append) — only at these breakpoints:**
- After completing a user story
- After making a non-obvious design decision (the "why" matters)
- After hitting and resolving an unexpected problem
- Before ending a session (handoff entry — update Status and Current State)

**When to READ:**
- At session start: if the journal exists, read Status and Current State sections to recover context.
- Never during normal work — you already have context from the current session.

**Context pressure:** If the conversation is getting long and you sense context may run out soon, write the handoff entry immediately. The dev can run `/resume` next session to pick up where you left off.

## Doc Sync (before committing)

After making code changes to any folder that contains a README.md or DESIGN.md, update those docs to reflect the current state before committing. Do not rewrite docs from scratch — make targeted edits to the sections affected by the code change.

## Git Workflow

When the user says "commit", "ship it", "push", or similar — follow this convention exactly.

### Commit Messages

Format: `<type>(<scope>): <short summary>`

Types: [from config git.commit_types]

Rules:
- Subject line under [config git.max_subject_length] characters, lowercase, no period
- Imperative mood ("add X", not "added X" or "adds X")
- Body explains **why**, not what — only if the subject isn't self-explanatory

### Branch Naming

Format: `<type>/<short-slug>`

### Pull Requests

Title matches the primary commit message format.

Body template:
## Summary
## Changes
## Test Plan

### Workflow: Commit and PR

When the user asks to commit and/or create a PR:
1. If SPEC.md exists, read it for PR context, then delete it
2. Verify /conform and /preflight have already passed
3. Stage relevant files (never git add . — be explicit)
4. Commit, push, create PR
5. Report the PR URL — do NOT merge

Never force-push, never merge without user approval, never push directly to main.

## Don'ts

- Don't commit secrets or API keys
- Don't add dependencies without checking deployment compatibility
- [Import restriction don'ts if configured]
```

**If CLAUDE.md already exists**, do NOT overwrite it. Instead:
1. Read the existing content
2. Identify which workflow sections are missing (Extensibility Policy, Task Journal, Git Workflow, Doc Sync)
3. Append only the missing sections at the end
4. Show the dev what was added and ask them to review placement

## Step 5: Generate or update CONTRIBUTING.md

**If CONTRIBUTING.md does not exist**, create it:

```markdown
# Contributing

## Getting Started

[Auto-detected: install + run commands]

## Working with Claude Code

This project uses [Claude Code](https://claude.ai/code) as the primary development tool. The workflow is designed so you describe what you want, Claude builds it, and you review before anything ships.

### Available commands

| Command | When to use |
|---------|-------------|
| `/kickoff` | Start work — paste tasks, get a SPEC, serial vs parallel recommendation |
| `/hotfix` | Single urgent fix — same quality checks, less ceremony |
| `/resume` | Continue work from a previous session |
| `/preflight` | Pre-PR quality gate. Use `/preflight recheck` to re-run only failed checks. |
| `/postmerge` | After merging a PR — verify main is healthy before merging the next |
| `/retest` | After fixing a bug found during manual testing |
| `/conform` | Check if changes match codebase patterns |
| `/docsync` | Audit folder-level docs for staleness |
| `/inspect` | Browser-level check (web projects only) — light mode by default |
| `/sweep` | Codebase cleanup — dead code, debug artifacts, gitignore gaps |

### Typical workflow

1. Claude writes tests first (TDD)
2. Claude implements — extends existing code by default
3. Claude tests in browser (web projects: `/inspect` light)
4. You test manually
5. If bugs found — `/retest` → fix → `/inspect` again
6. When satisfied — tell Claude to commit
7. Claude verifies `/conform` and `/preflight` passed, commits, creates PR
8. You review the PR — approve and merge
9. `/postmerge` — verify main is healthy

### Resuming a previous session

Run `/resume` — Claude reads task journals, checks out the right branch, and shows remaining work.

### What "done" means

- [ ] Tests pass
- [ ] `/conform` shows no issues (including extensibility and TDD checks)
- [ ] `/preflight` passes all checks (including regression scope for modified code)
- [ ] `/inspect` passed (web projects — light mode at minimum)
- [ ] Manual testing confirms the feature works
- [ ] If existing code was modified: all affected features tested, commit body explains why
- [ ] Folder-level docs updated
- [ ] PR created with Summary, Changes, and Test Plan sections

## Things to know

- **Extend, don't modify** — see CLAUDE.md "Extensibility and Modification Policy"
- [Project-specific items the dev mentioned during setup]
```

**If CONTRIBUTING.md already exists**, append the "Working with Claude Code" section if it's not already present. Do not overwrite existing content.

## Step 6: Update .gitignore

Check if these patterns are already in `.gitignore`. Append any that are missing:

```
# Planning artifacts (workflow plugin)
/SPEC.md
/TODO.md
/TASKS.md
JOURNAL-*.md
```

Do not duplicate patterns that already exist.

## Step 7: Create TASKS.md template

Create `TASKS.md` in the project root:

```markdown
# Tasks

Paste your tasks below in any format. Then run `/kickoff`.

## This round

Dump everything you're considering — all tasks, bugs, features, ideas. Any format works (GitHub Issues, Linear tickets, Jira, Slack messages, plain text). This is your full backlog for this round.



## How many this session

How many of the above should Claude tackle right now? Context windows are finite — pick a realistic number (1-3 is typical). Claude will prioritize, check dependencies, and plan just that subset.

> Example: "2" or "the first 3" or "just the auth bug"
```

## Step 8: Summary

Report what was created/updated:

```
## Initialized claude-workflow

### Created
- .claude/workflow.config.json (N fields configured)
- CONTRIBUTING.md (or updated: added N sections)
- TASKS.md (template — paste your tasks and run /kickoff)

### Updated
- CLAUDE.md (added: Extensibility Policy, Task Journal, Git Workflow)
- .gitignore (added: planning artifact patterns)

### Next steps
1. Review the generated files on this branch (`chore/init-workflow`)
2. Commit, push, and create a PR to merge into main
3. After merge — review CLAUDE.md, verify Architecture and Key Files
4. Paste tasks into TASKS.md and run /kickoff
```

## Rules

- Never overwrite existing content — always extend
- Always show the dev what was auto-detected and let them correct before generating
- If a file already has the workflow sections (e.g., from a previous init), skip it — don't duplicate
- The generated CLAUDE.md is a starting point — tell the dev to refine the Architecture section with project-specific details
