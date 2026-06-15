# /docsync

Audit all folder-level documentation for staleness against actual code.

## Config

Read `.claude/workflow.config.json` for `docs.folder_doc_names` and `docs.folders_to_check`.

## Scope

Check every folder listed in config `docs.folders_to_check`, plus any subfolder that contains a doc file matching config `docs.folder_doc_names`. Do NOT limit to changed files — scan everything.

## For each folder

1. List all source files in the folder.
2. Read the folder's doc files (per config `docs.folder_doc_names`).
3. Compare docs against code:
   - **Missing files** — source files that exist in the folder but are not mentioned in the docs.
   - **Ghost references** — files, classes, or functions described in the docs that no longer exist in the code.
   - **Stale descriptions** — doc describes behavior or interfaces that don't match the current code (e.g., wrong function signature, removed parameter, changed return type).
   - **New public interfaces** — classes or public functions added to existing files but not reflected in docs.

## Output

For each folder with findings, show:

```
## folder/
- MISSING: new_file.py not documented in README.md
- GHOST: README.md references old_file.py which was deleted
- STALE: DESIGN.md says function() returns X but it now returns Y
- NEW: file.py added NewClass, not in README.md
```

Skip folders with no issues.

End with a summary table (Folder, Missing, Ghost, Stale, New counts) and a one-line verdict: **All docs current** or **N folders need updates**.
