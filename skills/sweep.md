# /sweep

Audit the codebase for stale code, dead files, debug leftovers, and .gitignore gaps.

**Optional command** — requires `sweep` config section in `.claude/workflow.config.json`. If not configured, tell the developer and skip.

## Config

Read `.claude/workflow.config.json` for sweep-specific values (`sweep.dead_code_command`, `sweep.requirements_file`) and shared values (`preflight.debug_file_patterns`, `preflight.debug_code_patterns`, `preflight.debug_exclude_dirs`, `preflight.gitignore_required`).

## Checks

Run all of these and report findings grouped by category:

### 1. Dead imports and unused variables
- Run the project's dead code command (config `sweep.dead_code_command`) if configured.
- List each finding with file and line number.

### 2. Unreachable or commented-out code
- Search for blocks of commented-out code (3+ consecutive commented lines that look like code, not documentation).
- Search for functions/classes that are defined but never imported or called anywhere in the project.
- Ignore directories that should be excluded (config `preflight.debug_exclude_dirs`, plus common exclusions like `env/`, `node_modules/`, `__pycache__/`, `.claude/`).

### 3. Debug artifacts
- Search for patterns from config `preflight.debug_code_patterns` that look like debug output (not in logging or UI rendering).
- Search for tracked files matching config `preflight.debug_file_patterns`.
- Search for TODO/FIXME/HACK/XXX comments.

### 4. Empty or stub files
- Find source files that are empty or contain only imports/pass/docstrings with no real logic.
- Ignore `__init__.py` files and equivalent (those are legitimately empty).

### 5. .gitignore gaps
- Check if patterns from config `preflight.gitignore_required` are covered in `.gitignore`.
- Check if any currently tracked files should be gitignored (run `git ls-files` and flag anything matching common ignore patterns).
- Suggest additions if gaps are found.

### 6. Stale dependencies
- If config `sweep.requirements_file` is set, cross-reference it against actual imports in source files.
- Flag packages listed but never imported.
- Flag packages imported but not listed.

## Output

End with a summary table (Category, Findings count) and a one-line verdict: **Clean** or **N items to address**.
