# Change log: Docs folder guideline

**Date:** 2026-07-18

Implements plan
[../plans/20260718_105956_docs-folder-guideline.md](../plans/20260718_105956_docs-folder-guideline.md).

## What was changed

- **Created** [../DOCS_FOLDER_GUIDELINE.md](../DOCS_FOLDER_GUIDELINE.md) — a guideline for how to
  create files in a Flutter project's `docs/` folder.

## Summary of the new guideline

Built by analyzing the `docs/` folders of nine Flutter projects (Devi, PDFApp, ContactSphere,
TextApp, MantraJapaCounter, todo, qr_reader, youtube_shortcut, Authenticator). It covers:

1. **Scope** — applies to per-app docs; explicitly excludes `GUIDELINES_MANIFEST.md` and the
   `guidelines/` submodule (never edit submodule files locally).
2. **Local copy vs submodule** — fill in blueprints locally, link to reference docs.
3. **Naming** — canonical `snake_case` (decided with the user); existing `kebab-case` files
   tolerated, no forced rename; no date prefix.
4. **Standard file anatomy** — H1 title (+ app name), purpose paragraph, "read first" relative
   links, numbered `##` sections split by `---`, callouts, language-tagged code fences, tables,
   checklists, simple English.
5. **Living vs point-in-time docs** — the latter carry an absolute `**Date:**` line.
6. **Catalog of recognized doc types** — architecture, security, release_process/signing,
   workflow_rules, idea, implementation_plan, implementation_progress, known_gaps, dependencies,
   project_structure, security_audit — each with a section skeleton.
7. **New doc vs new section** — keep the set small; prefer sections.
8. **Cross-linking rules** — relative links up/sideways/to the submodule.
9. **Pre-save checklist.**

It pairs with the existing `CLAUDE_MD_GUIDELINE.md`.

## Notes

- No files in the nine analyzed projects were modified; they were read-only sources.
- Plan status set to `completed`.
