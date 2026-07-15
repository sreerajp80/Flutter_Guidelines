# Change log: Single "guidelines manifest" document

Implements plan `plans/20260712_180000_guidelines-manifest.md`.

## What changed

- **Added** `GUIDELINES_MANIFEST.md` at the repository root — a single, portable pointer file
  that lists the **full absolute paths** to every guideline document. It is meant to be copied
  into each Flutter app so the app can reference the master source-of-truth guidelines and stay
  consistent.

## Contents of the new file

- "How to use this file" instructions (copy into project root, reference from `CLAUDE.md`).
- A note that the master path is used only when there is no local copy in the app's `docs/`.
- Core documents table: 7 core docs + the README index, each with its full path and a one-line
  description (reused from `README.md`).
- Explainers table: the 5 `docs/*_README.md` explainers with full paths.
- The applicability-profile table (Core Baseline / Production App Extension / Sensitive Data
  Extension).
- A "where to start" quick guide.

## Notes

- No existing files were modified; only one new file was created.
- Paths are relative to the repository root. A previous absolute path layout was updated to relative paths.
