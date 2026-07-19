# /docsync

Audit all folder-level documentation for staleness against actual code.

## Config

Read `.claude/workflow.config.json` for `docs.folder_doc_names` and `docs.folders_to_check`.

## Scope

Every folder in `docs.folders_to_check`, plus any subfolder with a doc file matching `docs.folder_doc_names`. Scan everything — don't limit to changed files.

## For each folder

List source files, read the folder's doc files, compare:
- **Missing** — source files not mentioned in docs
- **Ghost** — docs reference files/classes/functions that no longer exist
- **Stale** — docs describe behavior that doesn't match current code (wrong signature, removed param, changed return type)
- **New** — new public interfaces added but not reflected in docs

## Output

```
## folder/
- MISSING: new_file.py not documented in README.md
- GHOST: README.md references old_file.py which was deleted
- STALE: DESIGN.md says function() returns X but it now returns Y
- NEW: file.py added NewClass, not in README.md
```

Skip folders with no issues. End with a summary table (Folder, Missing, Ghost, Stale, New counts) and verdict: **All docs current** or **N folders need updates**.
