# /sweep

Audit the codebase for stale code, dead files, debug leftovers, .gitignore gaps.

**Optional** — requires a `sweep` config section in `.claude/workflow.config.json`; missing → tell the dev and skip.

## Config

Read `sweep.dead_code_command`, `sweep.requirements_file`, plus shared `preflight.debug_file_patterns`, `preflight.debug_code_patterns`, `preflight.debug_exclude_dirs`, `preflight.gitignore_required`.

## Checks

1. **Dead imports/unused vars** — `sweep.dead_code_command` if configured; list file:line.
2. **Unreachable/commented-out code** — 3+ consecutive commented lines that look like code (not docs); functions/classes never imported or called anywhere. Exclude `preflight.debug_exclude_dirs` plus `env/`, `node_modules/`, `__pycache__/`, `.claude/`.
3. **Debug artifacts** — `preflight.debug_code_patterns` in tracked source (not logging/UI rendering); tracked files matching `preflight.debug_file_patterns`; TODO/FIXME/HACK/XXX comments.
4. **Empty/stub files** — source files empty or only imports/pass/docstrings. Ignore `__init__.py` equivalents.
5. **.gitignore gaps** — `preflight.gitignore_required` covered; `git ls-files` for tracked files that should be ignored. Suggest additions.
6. **Stale dependencies** — `sweep.requirements_file` vs actual imports: listed-but-unused, imported-but-unlisted.

## Output

Summary table (Category, Findings count) + verdict: **Clean** or **N items to address**.
