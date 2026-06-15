# /preflight

Pre-PR quality gate. All checks must pass before creating a PR.

## Config

Read `.claude/workflow.config.json` for project-specific commands, paths, and check parameters.

## Usage

- `/preflight` — run all checks
- `/preflight recheck` — re-run only checks that failed in the previous run this session (skips already-passed checks to save context)

## Checks

Run all of these and report a pass/fail summary:

1. **Secrets** — Scan all tracked source files for hardcoded API keys, tokens, or credentials (match patterns from config `preflight.secrets_patterns`). Verify files listed in `preflight.secrets_files` are in `.gitignore`.

2. **Imports** — Run the project's `commands.import_check` command (if configured) to catch broken imports. Flag any import that would fail in the deployment environment (e.g., missing from the dependency manifest).

3. **Requirements** — Cross-reference `import` statements across source files against the dependency manifest (config `sweep.requirements_file`). Flag packages imported but not listed, or listed but never imported.

4. **Lint and format** — Run the project's `commands.lint` and `commands.format_check` commands. Both must report zero issues. If format issues exist, run `commands.format` to fix them automatically, then re-check.

5. **Tests** — Run the project's `commands.test` command. All tests must pass.

6. **Config consistency** — If the project defines `architecture.registration_points` in config, verify all registration requirements are satisfied for any changed/added files that match the triggers.

7. **Planning artifacts** — Flag files listed in config `preflight.planning_artifacts` if they exist in the project root. These must be deleted or gitignored before proceeding.

8. **Debug artifacts** — Check for tracked files matching config `preflight.debug_file_patterns`. Check for patterns from `preflight.debug_code_patterns` in tracked source files (exclude directories listed in `preflight.debug_exclude_dirs`).

9. **.gitignore gaps** — Check if patterns from config `preflight.gitignore_required` are covered in `.gitignore`. Flag any tracked files that should be gitignored.

10. **Doc sync** — Identify all changed folders by combining `git diff --name-only` (staged + modified) AND `git ls-files --others --exclude-standard` (untracked). For every such folder that has a doc file (per config `docs.folder_doc_names`), verify the docs reflect the current code. Flag docs that describe things that no longer exist, or miss newly added files/functions.

11. **Registration points** — If the project defines `architecture.registration_points` and any matching files were changed or added, verify the requirement is met in all listed files. Flag if any registration point is missing.

## Modification Impact Check

If any existing code lines were modified (not just new code added), run this additional check:

12. **Regression scope** — For each modified function/method, grep for all callers and importers. Verify that existing tests cover those call paths. Flag any caller that has no test coverage — these need manual verification or a new regression test before committing.

## Journal Check (soft)

If a `JOURNAL-<current-branch>.md` exists, check whether its Status section is up to date with the current state of the branch. If stories are completed but not checked off, warn (don't fail).

## Output

End with a summary table of all checks (Pass/Fail) and a one-line verdict: **Ready for PR** or **Fix N issues first**.
