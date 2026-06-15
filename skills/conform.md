# /conform

Check whether staged or recent changes follow the conventions and patterns established in this codebase.

## Config

Read `.claude/workflow.config.json` for project-specific architecture, naming, and convention rules.

## What to check

Run `git diff --cached` (staged) or `git diff HEAD~1` (last commit) to identify changed files. Then verify each item below against the actual codebase patterns, not just config rules.

### 1. Architecture conformance
- New files are in the correct layer (per config `architecture.layers`)
- Import restrictions are respected (per config `architecture.layers[].forbidden_imports`)
- If centralized state management is configured, verify state access goes through the designated module (per config `architecture.state_management`)
- Data files live in the data directory (per config `paths.data_dir`), not hardcoded in source

### 2. Interface contracts
- If the project defines `architecture.registration_points`, verify any new/modified files that match a trigger are registered in all required locations
- New modules expose interfaces matching the existing contract in their directory (read 2-3 neighboring files to verify)

### 3. Naming, style, and structural patterns
- Read 2-3 existing files in the same directory to learn the local patterns — not just naming, but structure
- **Naming:** verify against config `naming` rules (functions, classes, constants, files)
- **Function signatures:** parameter ordering, default values, and return types should match the conventions of neighboring functions
- **Logging patterns:** if neighboring files use a specific logging approach, new code must match — not ad-hoc alternatives
- **Exception types:** if neighboring files use specific exception types for specific cases, new code should follow

### 4. Import patterns
- Compare import style with existing files in the same directory (relative vs absolute, ordering, grouping)
- No unused imports

### 5. Error handling patterns
- Compare with how neighboring files handle errors — does this code match?
- No bare `except:` clauses
- No swallowed exceptions (catch-and-pass with no logging)

### 6. Config and constants
- New thresholds, model IDs, or magic numbers belong in the config directory (per config `paths.config_dir`), not inline
- No hardcoded values that should be configurable

### 7. Commit message format
- If checking a committed change, verify the commit message follows: `<type>(<scope>): <summary>`
- Type must be in config `git.commit_types`
- Imperative mood, under config `git.max_subject_length` chars, lowercase, no period

### 8. Test coverage for new code
- For every new public function or class added, verify a corresponding test exists in the test directory (config `paths.test_dir`)
- Check that tests follow the naming convention (config `paths.test_pattern`)
- If new behavior was added to an existing module, verify the test file for that module was updated (new test functions appended, not existing tests modified)
- **This is a hard gate.** New code without tests means TDD was skipped — fail and require tests before proceeding.

### 9. Extensibility and modification scope
- Check `git diff` for modified (not just added) lines in existing files
- For each modified function or class: is there an extend-only alternative? (new function, wrapper, config entry, subclass)
- If modification was necessary: are the changes minimal? Are all callers/dependents tested?
- Flag cases where a large block of existing code was rewritten when an additive approach was possible
- **This is a hard gate.** If existing code was modified without justification (no extend-only alternative), or modified code lacks test coverage for affected callers, this is a fail. The dev must either refactor to an extend-only approach or explicitly approve the modification with a reason.

## Output

Group findings by category. For each finding, show:
- File and line number
- What the code does vs what the convention expects
- A suggested fix

End with: **Conforms** or **N items to align**.
