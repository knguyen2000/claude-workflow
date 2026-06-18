# /audit-workflow

Analyze how the workflow plugin is performing in this specific project. Surfaces gaps in config, drift between docs and code, workflow adherence issues, and developer friction points. Produces concrete suggestions to improve the plugin's effectiveness for this codebase.

**Run periodically** — after a sprint, before a major feature push, or whenever things feel sluggish. Does not modify any files; only reads and reports.

## Step 0: Load context

1. Read `.claude/workflow.config.json` — if it doesn't exist, stop and tell the dev to run `/init` first.
2. Read `CLAUDE.md` — needed for architecture and convention checks.
3. Read the plugin's `schema/workflow-config.schema.json` — needed for schema drift detection.
4. Note the project type from config — some checks are type-specific.

## Category 1: Workflow Adherence

Check whether the team is actually following the workflow the plugin defines.

### 1.1 Commit message compliance
- Run `git log --oneline -50` (or since last audit if tracked).
- Check each commit against `<type>(<scope>): <summary>` format.
- Flag: wrong case, missing type/scope, subject over `git.max_subject_length` (default 60), non-imperative mood ("added" instead of "add"), trailing period.
- Report: compliance percentage + list of violations.

### 1.2 Branch naming compliance
- Run `git branch -a` and filter to recent branches (merged or not in the last 30 days).
- Check each against `<type>/<short-slug>` format using `git.commit_types` for valid types.
- Flag: branches that don't follow convention, branches with too-long slugs.

### 1.3 TDD discipline
- Look at the last 20 commits that touch source files (not tests, docs, or config).
- For each, check if the same commit (or the immediately preceding commit) also touches test files.
- Report: TDD adherence rate (percentage of source-changing commits that include test changes).
- Flag: source-only commits that add new public functions/classes without any test.

### 1.4 Modification protocol compliance
- Find commits where existing source files had significant changes (more than 10 lines modified, not just additions).
- Check if the commit body contains justification (non-empty body beyond the subject line).
- Flag: large modifications with no commit body explaining why.

### 1.5 PR quality (if GitHub remote exists)
- Check recent merged PRs (last 10) for body structure.
- Flag: PRs missing ## Summary, ## Changes, or ## Test Plan sections.
- If no GitHub remote or `gh` unavailable, skip this check with a note.

### 1.6 Workflow pattern analysis
- Count commit types over the last 50 commits (how many feat, fix, hotfix, refactor, etc.).
- Report the distribution.
- Flag if `fix` commits outnumber `feat` commits 2:1 or more — may indicate insufficient upfront planning or test coverage.
- Flag if `chore`/`style` commits are very high — may indicate the workflow is generating busywork.

## Category 2: Config Health

Check whether `workflow.config.json` is complete, accurate, and not stale.

### 2.1 Stale path references
- For every path in config (`paths.test_dir`, `architecture.layers[].path`, `inspect.pages[].path`, `docs.folders_to_check`, etc.), verify the path actually exists on disk.
- Flag: paths that don't exist (renamed, deleted, or typo).

### 2.2 Missing config sections
Evaluate which optional sections the project would benefit from but hasn't configured:

- **`architecture`** — does the project have 3+ top-level source directories? Suggest adding layers and import restrictions.
- **`inspect`** — is `project_type` set to `web` but no `inspect` section? Suggest adding pages and viewports.
- **`sweep`** — does the project have a requirements/dependencies file but no `sweep` config? Suggest adding it.
- **`docs`** — do source directories contain README.md or DESIGN.md files but no `docs` config? Suggest adding folders_to_check.
- **`preflight.secrets_patterns`** — is this empty or missing? Check if `.env`, `secrets.toml`, or similar files exist in the repo.
- **`architecture.registration_points`** — see Category 6 for emergent detection.

### 2.3 Schema drift
- Compare the keys present in the project's `workflow.config.json` against the full schema from the plugin.
- Flag: new schema fields the project isn't using that would be relevant to its project type.
- Do NOT flag optional fields that genuinely don't apply.

### 2.4 Command accuracy
- For each command in config (`commands.test`, `commands.lint`, etc.), verify the tool exists:
  - For Python: check if the binary is referenced in `requirements.txt` or `pyproject.toml`.
  - For JS/TS: check if it's in `package.json` devDependencies.
  - For others: just note if the command looks plausible.
- Flag: commands referencing tools not in the dependency list.

## Category 3: Codebase Structural Health

Detect structural issues that slow down development and cause Claude to make mistakes.

### 3.1 File size creep
- Find source files (exclude tests, configs, generated files) over 300 lines.
- Rank by size. Flag files over 500 lines as "should split."
- Report top 5 largest files.

### 3.2 Test coverage gaps
- List all source modules/files in `paths.test_dir`'s parent (or configured source directories).
- Check which ones have a corresponding test file (matching `paths.test_pattern`).
- Report: coverage percentage (files with tests / total source files).
- Flag: source files with no corresponding test file at all.

### 3.3 Naming consistency
- Sample 20 recent source files. Check function/class/constant/file names against `naming` config.
- Flag: files not following `naming.files`, functions not following `naming.functions`, etc.
- Report specific violations.

### 3.4 Layer violations (requires `architecture` config)
- If `architecture.import_restrictions` is configured, scan source files for violations.
- For each restriction `{ "module": "X", "cannot_import": ["Y", "Z"] }`, grep X's files for imports from Y or Z.
- Flag: each violation with file and line number.

### 3.5 Circular dependency detection
- Build a simplified import graph from source files (top-level imports only).
- Detect cycles (A imports B, B imports A — or longer chains).
- Flag: each cycle with the files involved.

## Category 4: Git Hygiene

### 4.1 Stale branches
- List all local and remote branches. Check their last commit date.
- Flag: branches with no commits in 14+ days that aren't merged.
- Suggest: delete or merge.

### 4.2 Divergence risk
- For each open feature branch, count commits behind main.
- Flag: branches 20+ commits behind — merge conflicts are likely.
- Suggest: rebase or merge main in.

### 4.3 Working directory drift
- Run `git status` (not in the audit output — just check).
- Flag: if there are uncommitted changes to 5+ files, it suggests work in progress that should be committed or stashed.

## Category 5: Documentation Health

### 5.1 CLAUDE.md architecture drift
- If CLAUDE.md has an "Architecture" section, extract the listed directories/layers.
- Compare against actual top-level directories on disk.
- Flag: directories listed in CLAUDE.md that don't exist.
- Flag: new source directories on disk that aren't mentioned in CLAUDE.md.

### 5.2 CLAUDE.md key files drift
- If CLAUDE.md has a "Key Files" section, extract the file paths mentioned.
- Check each exists on disk.
- Flag: referenced files that were deleted or renamed.

### 5.3 CONTRIBUTING.md command table
- If CONTRIBUTING.md has a command table, extract the commands listed.
- Compare against the plugin's actual skills (from the skills/ directory).
- Flag: commands listed that don't exist in the plugin (removed or renamed).
- Flag: plugin commands not listed in CONTRIBUTING.md.

### 5.4 Quick doc freshness
- For the top 5 most-recently-modified source directories, check if their folder docs were updated in the same time window.
- Flag: directories with code changes in the last 2 weeks but no doc changes. These are likely stale.

## Category 6: Developer Friction Detection

The highest-value category. Detects patterns that could be automated or codified.

### 6.1 Emergent registration points
- Analyze the last 30 commits. Find commits where 3+ files were changed together repeatedly (same combination in 2+ commits).
- These are likely registration points — adding a feature requires touching multiple specific files.
- Compare against `architecture.registration_points` in config.
- Suggest: adding newly discovered registration points to config so `/conform` catches missing updates.

### 6.2 Unconfigured import restrictions
- Build the import graph (same as 3.5).
- Identify natural clusters (groups of files that import each other but not files outside the group).
- If `architecture.import_restrictions` doesn't exist or doesn't cover these clusters, suggest adding restrictions.
- This prevents accidental coupling.

### 6.3 Undocumented conventions
- Sample source files and look for repeated patterns that aren't in CLAUDE.md:
  - Common decorator usage (e.g., every route uses `@auth_required`)
  - Consistent error handling patterns (e.g., all functions return `Result` type)
  - Consistent logging patterns (e.g., all modules use `logger = logging.getLogger(__name__)`)
  - Consistent file headers or imports
- Flag patterns appearing in 70%+ of files but not documented in CLAUDE.md Conventions section.
- Suggest: adding them so Claude follows them automatically without rediscovering each session.

### 6.4 Suggested CLAUDE.md additions
Based on everything discovered, generate specific additions:
- New directories for Architecture section
- New key files (files changed most frequently in recent commits)
- New conventions detected in 6.3
- New don'ts based on common mistakes in commit history

### 6.5 Session recovery health
- Check for `JOURNAL-*.md` files in the project root.
- If they exist, check if they have a "Current State" section that's been updated recently.
- Flag: journals with no handoff entry (Status section has unchecked items but no recent Log entry).
- Flag: journals for branches that are already merged (should have been cleaned up).

## Output Format

Present results grouped by category with a severity marker:

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

[Details for WARN and ACTION items...]

### Category 2: Config Health
...

### Category 6: Developer Friction Detection
...
```

Severity levels:
- **OK** — no issues
- **INFO** — observation, no action needed
- **WARN** — worth addressing when convenient
- **ACTION** — address before next feature push

## Summary

End with two sections:

### Quick Wins
List 3-5 specific, concrete changes that would have the most impact. Each should be one sentence with exactly what to do:
- "Add `architecture.registration_points` to config: `[{ files: ['config/app_config.py', 'components/agent_dispatch.py', 'app.py'], trigger: 'adding an agent mode' }]`"
- "Document the logging pattern in CLAUDE.md Conventions: `logger = logging.getLogger(__name__)` — appears in 85% of modules"
- "Delete stale branches: `feat/old-feature` (35 days, not merged)"

### Suggested Config Patch
If config improvements were found, output the exact JSON to add/modify in `workflow.config.json` — the dev can copy-paste it directly.

## Rules

- **Read-only** — never modify files. Only report and suggest.
- **Be specific** — every suggestion must include exact file paths, line numbers, or config keys. No vague "consider improving test coverage."
- **Prioritize signal** — if everything is clean, say so in two lines. Don't pad the report. A clean audit is a good audit.
- **Project-type aware** — skip checks that don't apply (e.g., no inspect checks for `cli` projects, no layer violations without `architecture` config).
- **Git history depth** — default to last 50 commits or 30 days, whichever is less. Don't scan the entire history of a large project.
