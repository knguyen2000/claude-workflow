# /conform

Check whether staged or recent changes follow this codebase's established conventions and patterns.

## Config

Read `.claude/workflow.config.json` for architecture, naming, convention rules.

## What to check

`git diff --cached` (staged) or `git diff HEAD~1` (last commit) for changed files. Verify against actual codebase patterns, not just config rules.

1. **Architecture** — new files in the correct layer (`architecture.layers`); import restrictions respected (`architecture.layers[].forbidden_imports`); state access goes through the designated module if `architecture.state_management` is set; data files live in `paths.data_dir`, not hardcoded.

2. **Interface contracts** — new/modified files matching an `architecture.registration_points` trigger are registered everywhere required; new modules match the interface of 2-3 neighboring files.

3. **Naming, style, structure** — read 2-3 files in the same directory first. Check `naming` config (functions/classes/constants/files); function signatures (param order, defaults, return types) match neighbors; logging matches the local approach; exception types match local conventions.

4. **Imports** — style matches neighbors (relative vs absolute, ordering, grouping); no unused imports.

5. **Error handling** — matches neighboring files; no bare `except:`; no swallowed exceptions (catch-and-pass with no logging).

6. **Config and constants** — new thresholds/model IDs/magic numbers go in `paths.config_dir`, not inline.

7. **Commit message** — if checking a committed change: `<type>(<scope>): <summary>`, type in `git.commit_types`, imperative, under `git.max_subject_length`, lowercase, no period.

8. **Test coverage** — every new public function/class has a test in `paths.test_dir` following `paths.test_pattern`; new behavior on an existing module has appended (not modified) test functions. **Hard gate** — new code without tests fails; require tests before proceeding.

9. **Extensibility scope** — `git diff` modified (not just added) lines: was an extend-only alternative available (new function, wrapper, config entry, subclass)? If modification was necessary, is it minimal and are all callers tested? **Hard gate** — unjustified modification, or modified code with untested affected callers, fails. Dev must refactor extend-only or explicitly approve with a reason.

## Output

Group findings by category. Each: file:line, actual vs expected, suggested fix. End with **Conforms** or **N items to align**.
