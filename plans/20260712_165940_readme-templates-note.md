# Plan: Add a "these are templates" note to README.md

**Status:** completed

## The issue
The guideline documents cross-reference each other with a `docs/` prefix (e.g.
`docs/architecture.md`) in ~40 places. This is a deliberate convention: the files are
templates meant to be copied into a target Flutter app's `docs/` folder and referenced from
that app's `CLAUDE.md`. In this repository the files are flat, which can confuse a reader
into thinking the `docs/` paths are broken (it briefly confused me).

## The fix
Add one short section to the top-level `README.md` (the file created earlier) explaining:
- These files are **templates / a source collection**, not a deployed project.
- When you adopt them, most are copied into your app's `docs/` folder; the index/README
  sits at the app's project root.
- That is why cross-references inside the documents use a `docs/` prefix
  (e.g. `docs/architecture.md`) — the prefix means "once the file lives in your app's
  `docs/` folder", not a path inside this repository.

## Files to be changed
1. `README.md` (EDIT) — add the note. No other files change.

## Not changed
No `docs/` prefixes are edited anywhere — the convention is correct and stays as-is.

## Verification
- Re-read the edited `README.md` section for clarity and correct links.

## After implementation
Write a change log to `change_log/` referencing this plan.
