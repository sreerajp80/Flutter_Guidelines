# Change Log: CLAUDE.md guideline doc

**Date:** 2026-07-18
**Implements:** plans/20260718_104600_claude-md-guideline.md

## What was done

Analyzed nine existing Flutter project `CLAUDE.md` files (Devi, PDFApp, ContactSphere, TextApp,
MantraJapaCounter, todo, qr_reader, youtube_shortcut, Authenticator) and created a single
guideline document that explains how to write a consistent `CLAUDE.md` for any new Flutter
project.

## Files created

- `CLAUDE_MD_GUIDELINE.md` — the guideline. Contains:
  - Two profiles (Thin pointer vs Thick self-contained) and how to choose.
  - A canonical section order.
  - A required-vs-optional section checklist for both profiles.
  - A full fill-in-the-blanks `CLAUDE.md` template with placeholders.
  - Per-section writing tips drawn from the nine files.
  - Anti-patterns to avoid.
  - Mandatory global rules (plan/approve/log workflow + simple-English communication) kept inline.
  - A final self-check list.
- `plans/20260718_104600_claude-md-guideline.md` — the approved plan (status now completed).
- `change_log/20260718_105200_claude-md-guideline.md` — this log.

## Notes

- No existing project `CLAUDE.md` files were modified.
- Referenced `docs/` templates (architecture.md, security.md, etc.) were not created — out of
  scope; can be a follow-up if wanted.
