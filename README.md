# claude-workflow

[![License](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)

> Created and maintained by **Ryan Nguyen** ([knguyen2000](https://github.com/knguyen2000))

Strict, extensibility-first development workflow for Claude Code. Provides slash commands for task intake, quality gates, post-merge verification, and session recovery.

Works with any stack. You configure your project's specifics once in a JSON file; the workflow commands adapt.

## Install

Add to your project's `.claude/settings.json`:

```json
{
  "plugins": ["github:knguyen2000/claude-workflow"]
}
```

Then run `/init` in your repo — it scans your codebase, asks a few questions, and generates all config and docs. Or create `.claude/workflow.config.json` manually (see [Configuration](#configuration)).

## Commands

| Command | Purpose |
|---------|---------|
| `/init` | Bootstrap the plugin — scans your repo, generates config, CLAUDE.md, CONTRIBUTING.md |
| `/kickoff` | Start work — paste tasks, get a SPEC, serial/parallel/hybrid recommendation |
| `/hotfix` | Single urgent fix — same quality bar, less ceremony |
| `/commit` | Stage your manual changes, generate a commit message, commit, and push |
| `/preflight` | Pre-PR quality gate (tests, lint, format, secrets, docs, regression scope) |
| `/postmerge` | Post-merge verification — required after every PR merge |
| `/resume` | Recover context from a previous session via task journals |
| `/conform` | Verify changes match codebase patterns (hard-gate on extensibility and TDD) |
| `/retest` | Re-validate after a bug found during manual testing |
| `/inspect` | Browser-level check via Chrome DevTools MCP (web projects only) |
| `/docsync` | Audit folder-level docs for staleness |
| `/sweep` | Codebase cleanup — dead code, debug artifacts, gitignore gaps (optional) |
| `/audit-workflow` | Audit how the plugin is performing — config health, adherence, friction detection |

## Core Principles

**Extend, don't modify.** Default to adding new code. When modification is unavoidable: read the code and callers first, justify the change, minimize scope, run impact analysis, test all affected paths, and explain why in the commit body.

**TDD is not optional.** Every new feature starts with a failing test. `/conform` hard-gates this — new public code without tests is a fail.

**Every merge is verified.** `/postmerge` runs after every PR merge (serial, parallel, and hybrid workflows). No PR merges until the previous merge is verified healthy.

**Sequential integration testing.** For parallel/hybrid workflows, branches are merged one at a time in order (not all at once) on a throwaway branch, with tests after each merge. This simulates the real merge sequence.

## Configuration

Create `.claude/workflow.config.json` in your repo. Copy `templates/workflow-config.example.json` from this plugin as a starting point.

### Required fields

```json
{
  "project_name": "My Project",
  "project_type": "web",
  "commands": {
    "test": "pytest",
    "lint": "ruff check .",
    "format_check": "ruff format --check ."
  },
  "paths": {
    "test_dir": "tests/",
    "test_pattern": "test_{module}.py"
  },
  "naming": {
    "functions": "snake_case",
    "classes": "PascalCase",
    "constants": "UPPER_CASE",
    "files": "snake_case"
  },
  "git": {
    "commit_types": ["feat", "fix", "docs", "refactor", "test", "chore", "style", "perf"]
  }
}
```

### Optional sections

| Section | What it enables |
|---------|----------------|
| `commands.start` | `/inspect` can auto-start the app |
| `commands.import_check` | `/preflight` verifies imports resolve |
| `commands.lint_fix`, `commands.format` | Auto-fix capability |
| `architecture` | `/conform` checks layer boundaries, import restrictions, state management, registration points |
| `inspect` | Pages, viewports, lighthouse threshold, smoke test for `/inspect` |
| `preflight` | Secrets patterns, debug patterns, gitignore requirements for `/preflight` |
| `sweep` | Dead code command, requirements file for `/sweep` |
| `docs` | Folder doc names and folders to audit for `/docsync` |

See `schema/workflow-config.schema.json` for the full schema with descriptions of every field.

### Project types

| Type | `/inspect` | Notes |
|------|-----------|-------|
| `web` | Full support | Pages, viewports, Lighthouse, smoke test |
| `api` | Skipped | Test via curl/httpie in `/preflight` custom checks |
| `cli` | Skipped | Test via shell commands |
| `library` | Skipped | Tests only |

## Workflow at a Glance

```
/init (once per repo)
  │
/kickoff (or /hotfix for single fixes)
  │
  implement (TDD, extend-first) → commit per story
  │
  /inspect light (web projects)
  │
  ← dev manual tests
  │
  /conform → /preflight
  │
  PR → dev reviews + merges
  │
  /postmerge → verify main is healthy
  │
  next task (or done)
  │
/audit-workflow (periodically — after sprint, before major push)
```

## Overriding a command

If a repo needs to customize one command, create a local `.claude/commands/<name>.md` — Claude Code resolves local commands over plugin commands. No need to fork the plugin.

## What this plugin does NOT include

- **`/fix`** — too tool-specific and trivially simple. If lint/format fails in `/preflight`, run your stack's fixer directly.
- **Project-specific scaffolds** (like "add an agent mode") — these belong in the repo's local `.claude/commands/`.
- **CLAUDE.md content** — `/init` generates a starting-point CLAUDE.md, but the plugin itself doesn't mandate its contents. Your repo's CLAUDE.md is yours to customize.

## License

Copyright 2025-2026 Ryan Nguyen ([knguyen2000](https://github.com/knguyen2000))

Licensed under the **Apache License, Version 2.0**. You may use, fork, and modify this plugin freely, provided you:

1. **Keep the LICENSE and NOTICE files** in any copy or derivative
2. **Credit the original author** (Ryan Nguyen) — do not present this work as your own
3. **State any changes** you make to the original

See [LICENSE](LICENSE) for the full terms.

## Contributing

The plugin is designed to be universal. Before adding a feature, verify it applies to at least 3 different stack types (Python, JS/TS, Rust/Go). Stack-specific logic belongs in config, not in commands.
