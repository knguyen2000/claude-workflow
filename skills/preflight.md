# /preflight

Pre-PR quality gate. All checks must pass before creating a PR.

## Config

Read `.claude/workflow.config.json` for commands, paths, check parameters.

## Usage

`/preflight` — all checks. `/preflight recheck` — only checks that failed last run this session (saves context).

## Checks

1. **Secrets** — scan tracked source for hardcoded keys/tokens/credentials (`preflight.secrets_patterns`). Verify `preflight.secrets_files` are in `.gitignore`.
2. **Imports** — run `commands.import_check` if configured; flag anything that'd fail in deployment.
3. **Requirements** — cross-reference imports against `sweep.requirements_file`; flag imported-but-unlisted or listed-but-unused.
4. **Lint and format** — `commands.lint` + `commands.format_check`, both zero issues. Format issues → run `commands.format`, re-check.
5. **Tests** — `commands.test`, all must pass.
6. **Config consistency** — if `architecture.registration_points` set, verify satisfied for changed/added trigger-matching files.
7. **Planning artifacts** — flag `preflight.planning_artifacts` files present in project root; must be deleted/gitignored.
8. **Debug artifacts** — tracked files matching `preflight.debug_file_patterns`; `preflight.debug_code_patterns` in tracked source (excluding `preflight.debug_exclude_dirs`).
9. **.gitignore gaps** — `preflight.gitignore_required` patterns covered; flag tracked files that should be ignored.
10. **Doc sync** — changed folders (`git diff --name-only` + `git ls-files --others --exclude-standard`) with a doc file (`docs.folder_doc_names`): docs reflect current code, no ghost references, new files/functions covered.
11. **Registration points** — `architecture.registration_points` satisfied in all listed files for matching changes.

**Modification Impact Check** — if any existing lines were modified (not just added):

12. **Regression scope** — for each modified function/method, grep callers/importers; verify test coverage. Flag untested callers — need manual verification or a new regression test.

## Journal check (soft)

`JOURNAL-<branch>.md` exists? Warn (don't fail) if completed stories aren't checked off.

## Output

Summary table (Pass/Fail per check) + verdict: **Ready for PR** or **Fix N issues first**.
