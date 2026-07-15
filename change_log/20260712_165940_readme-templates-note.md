# Change log: "These are templates" note in README.md

Implements plan `plans/20260712_165940_readme-templates-note.md`.

## What was changed
Added a short **"These are templates"** section to the top-level `README.md`, placed after
the intro paragraph and before "The documents".

The note explains:
- This repository is a source collection of templates, not a deployed app.
- When adopted, most documents are copied into a target app's `docs/` folder and referenced
  from that app's `CLAUDE.md`; the index/README sits at the app's project root.
- The `docs/` prefix in the documents' cross-references (e.g. `docs/architecture.md`) means
  "once the file lives in your app's `docs/` folder", not a path inside this repository,
  where the files are flat.

## Why
The `docs/` prefix is used consistently across all documents (~40 references). Because this
repo stores the files flat, a reader can mistake the `docs/` paths for broken links. This
note removes that confusion without touching any of the intended `docs/` cross-references.

## Files changed
- `README.md` (added the section).

## Files not changed
No `docs/` prefixes were edited anywhere — the convention is correct and left intact.
