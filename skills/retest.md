# /retest

Re-validate a fix after manual testing revealed an issue. Enforces test integrity — existing tests are never modified or deleted without explicit approval.

## Config

Read `.claude/workflow.config.json` for test command and naming convention.

## Context

Used when: TDD-implemented feature passed tests → manual testing found a bug/edge case → source was fixed → now verify without corrupting the test suite.

## Steps

1. **Identify what changed** — `git diff` on the fix; note existing test files for affected modules.
2. **Classify** — **existing story** (behavior already covered by existing tests, e.g. off-by-one) → skip to step 4. **New story** (not covered by any test, e.g. new edge case/input/interaction) → step 3.
3. **Append new tests (new story only)** — append new test functions at the end of the relevant file; never modify/reorder/delete existing lines. Clear names per edge case. No file exists → create one per `paths.test_pattern`.
4. **Run the full suite** — no file filters, everything must pass, not just new tests.
5. **Any failure** — don't touch the failing test. It means the fix is incomplete or regressed something. Fix the source, return to step 4.
6. **Verify conformance** — fix follows neighboring naming/import/error-handling patterns; modified existing code was minimal with tested callers; new functions each have a test. Fix any issue before reporting.
7. **Report** — source files fixed, new test cases added (by name), conformance status, test output summary, verdict: **All tests pass** or **N failures remain**.

## Rules

- Default: never delete, rename, or modify existing test lines — not even whitespace/comments
- **Exception:** if a test asserts behavior the bugfix *intentionally and correctly* changes (the old assertion encoded the bug), explain why and get explicit approval before modifying — show old assertion, new assertion, and why the new behavior is correct
- Never use `--no-verify` or skip hooks
- Uncertain whether a test is wrong or the fix is incomplete? Stop and ask
