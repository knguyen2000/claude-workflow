# /audit-workflow

Analyze how the plugin is performing in this project: config gaps, doc/code drift, adherence issues, friction points. Read-only — never modifies files.

**Run periodically** — after a sprint, before a major push, or when things feel sluggish.

## Step 0: Load context

Read `.claude/workflow.config.json` (missing → stop, tell dev to run `/init`), `CLAUDE.md`, the plugin's `schema/workflow-config.schema.json`. Note project type — some checks are type-specific.

## Category 1: Workflow Adherence

1.1 **Commits** — `git log --oneline -50`. Check `<type>(<scope>): <summary>` format. Flag wrong case, missing type/scope, subject over `git.max_subject_length` (default 60), non-imperative mood, trailing period. Report compliance % + violations.

1.2 **Branches** — `git branch -a`, recent (merged or <30d). Check `<type>/<short-slug>` against `git.commit_types`. Flag non-conforming or overlong slugs.

1.3 **TDD** — last 20 source-touching commits (exclude tests/docs/config): does the same or immediately preceding commit touch tests? Report adherence rate. Flag source-only commits adding new public functions/classes with no test.

1.4 **Modification protocol** — commits with >10 modified (not just added) lines: does the commit body justify it? Flag large modifications with no body.

1.5 **PR quality** (if GitHub remote) — last 10 merged PRs: body has `## Summary`/`## Changes`/`## Test Plan`? Flag missing sections. No `gh`/remote → skip, note why.

1.6 **Pattern analysis** — commit type distribution over last 50. Flag `fix:feat` ratio ≥2:1 (weak planning/coverage) or high `chore`/`style` share (busywork signal).

## Category 2: Config Health

2.1 **Stale paths** — every config path (`paths.test_dir`, `architecture.layers[].path`, `inspect.pages[].path`, `docs.folders_to_check`, etc.) exists on disk? Flag missing.

2.2 **Missing sections** — suggest adding: `architecture` if 3+ top-level source dirs and unconfigured; `inspect` if `project_type: web` and unconfigured; `sweep` if a deps file exists but unconfigured; `docs` if READMEs/DESIGNs exist but unconfigured; `preflight.secrets_patterns` if empty and `.env`/`secrets.toml`-like files exist. (Registration points: see 6.1.)

2.3 **Schema drift** — project config keys vs full plugin schema. Flag relevant-but-unused new fields. Don't flag genuinely inapplicable optional fields.

2.4 **Command accuracy** — for each `commands.*`, does the referenced tool appear in the dependency manifest (`requirements.txt`/`pyproject.toml` for Python, `package.json` devDeps for JS)? Flag mismatches; for other stacks just note plausibility.

## Category 3: Codebase Structural Health

3.1 **File size** — source files (exclude tests/config/generated) over 300 lines, ranked; flag 500+ as "should split." Report top 5.

3.2 **Test coverage gaps** — source files vs `paths.test_dir`/`paths.test_pattern` matches. Report coverage %, flag files with none.

3.3 **Naming consistency** — sample 20 recent files against `naming` config (functions/classes/constants/files). Report specific violations.

3.4 **Layer violations** (needs `architecture.import_restrictions`) — grep each restricted module's files for forbidden imports. Flag with file:line.

3.5 **Circular deps** — build a top-level import graph, detect cycles. Flag each with files involved.

## Category 4: Git Hygiene

4.1 **Stale branches** — no commits in 14+ days, unmerged. Suggest delete or merge.
4.2 **Divergence risk** — open branches 20+ commits behind main. Suggest rebase/merge.
4.3 **Working dir drift** — `git status` shows 5+ uncommitted files → suggests uncommitted WIP.

## Category 5: Documentation Health

5.1 CLAUDE.md Architecture section directories vs actual top-level dirs — flag stale entries and undocumented new dirs.
5.2 CLAUDE.md Key Files vs disk — flag deleted/renamed files still referenced.
5.3 CONTRIBUTING.md command table vs plugin's actual `skills/` — flag removed/renamed commands listed, and plugin commands missing from the table.
5.4 Top 5 most-recently-changed source dirs — folder docs updated in the same window? Flag likely-stale ones (code changed in last 2wk, docs didn't).

## Category 6: Developer Friction Detection

The highest-value category — patterns that could be automated or codified.

6.1 **Emergent registration points** — last 30 commits: file-groups of 3+ changed together repeatedly (2+ times). Compare to `architecture.registration_points`; suggest adding newly discovered ones so `/conform` can catch missed updates.

6.2 **Unconfigured import restrictions** — from the import graph (3.5), find natural clusters not covered by `architecture.import_restrictions`; suggest restrictions to prevent accidental coupling.

6.3 **Undocumented conventions** — sample files for repeated patterns absent from CLAUDE.md Conventions (decorator usage, error-handling style, logging pattern, file headers). Flag patterns in 70%+ of files but undocumented; suggest adding so Claude follows them without rediscovering each session.

6.4 **Suggested CLAUDE.md additions** — new Architecture dirs, new Key Files (most frequently changed), new conventions from 6.3, new don'ts from common commit-history mistakes.

6.5 **Session recovery health** — `JOURNAL-*.md` files: Current State updated recently? Flag journals with unchecked Status items but no recent Log entry, and journals for already-merged branches.

## Output Format

```
## Audit Results

### Category 1: Workflow Adherence
| Check | Result | Severity |
|-------|--------|----------|
| Commit messages | 82% compliant (9/50 violations) | WARN |
| Branch naming | 100% compliant | OK |
| TDD discipline | 65% adherence | ACTION |
| Modification protocol | 3 unjustified large changes | WARN |
| PR quality | 8/10 have proper structure | OK |
| Workflow patterns | fix:feat ratio 1.8:1 | OK |

[Details for WARN/ACTION items...]

### Category 2-6: ...
```

Severity: **OK** none · **INFO** observation only · **WARN** address when convenient · **ACTION** address before next feature push.

## Summary

**Quick Wins** — 3-5 specific one-sentence changes, e.g.:
- "Add `architecture.registration_points`: `[{ files: [...], trigger: 'adding an agent mode' }]`"
- "Document logging pattern in CLAUDE.md: `logger = logging.getLogger(__name__)` — 85% of modules"
- "Delete stale branch `feat/old-feature` (35 days, unmerged)"

**Suggested Config Patch** — exact JSON to add/modify in `workflow.config.json`, copy-pasteable.

## Rules

- Read-only — report and suggest, never modify
- Be specific — exact file paths, line numbers, config keys; no vague "improve test coverage"
- If everything's clean, say so in two lines — don't pad
- Skip checks that don't apply to the project type (e.g. no inspect checks for `cli`, no layer checks without `architecture`)
- Default to last 50 commits or 30 days, whichever is less — don't scan full history
