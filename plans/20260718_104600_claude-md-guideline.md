# Plan: Guideline doc to generate CLAUDE.md for future Flutter projects

**Status:** completed

## What the issue is

There are nine existing Flutter project `CLAUDE.md` files (Devi, PDFApp, ContactSphere,
TextApp, MantraJapaCounter, todo, qr_reader, youtube_shortcut, Authenticator). Each was written
by hand and they drift in structure, section order, and depth. There is no single guideline that
says *how* to write a good `CLAUDE.md` for a new Flutter project, so future files will keep
drifting.

Goal: create one guideline document that a person (or Claude) can follow to generate a
consistent, high-quality `CLAUDE.md` for any new Flutter project.

## What I found in the nine files (basis for the guideline)

Common building blocks seen across the files:

1. **Header banner** — "read this before any change"; note it is auto-loaded every session.
2. **Project identity / tech stack** — app name, type, platform(s), package/org id, versions
   (Flutter, Dart, minSdk/targetSdk), state management, navigation, database, orientation.
3. **Doc references** — links to `docs/architecture.md`, `docs/security.md`,
   `docs/release_process.md`, `docs/flutter_build_flavors_guide.md`,
   `docs/flutter_project_engineering_standard.md`, and `docs/GUIDELINES_MANIFEST.md`.
4. **Hard / non-negotiable rules** — offline-first, open-source-only, scoped storage,
   never crash on bad input, atomic/copy-on-write saves, no dead buttons, etc.
5. **Architecture rules** — Tier 1 layer-first layout, layer boundaries, dependency direction,
   "no DB access from widgets", immutable models.
6. **Build & run commands** — `flutter run --flavor`, obfuscated release apk/appbundle.
7. **Build flavors** — `dev` / `prod`, app id suffix, display names, `FLUTTER_APP_FLAVOR`.
8. **Signing / keystore** — key.properties location, gitignore rules, keep backups.
9. **Security rules** — never log secrets, minimal permissions, no INTERNET when offline.
10. **Code style / naming conventions** — snake_case files, PascalCase classes, format+analyze.
11. **Testing rules** — mirror `lib/` in `test/`, coverage targets, critical areas.
12. **Dependency constraints** — allow/block lists, vetting new packages.
13. **Workflow rules** — plan-before-changing + log-after-changing (from global rules).
14. **Communication rules** — always simple English.
15. **Dos & Don'ts** — "What Claude must always / never do".
16. **Where things live** — project tree.

Two observed styles: **thin pointer** files (delegate detail to `docs/`) and **thick
self-contained** files (inline everything). The guideline will cover both and say when to pick
which.

## The plan for the doc

Create a single guideline file that contains:

- **Purpose & how to use it** — read the nine reference files' spirit; fill the template per project.
- **Two profiles**: *Thin pointer* (project has a full `docs/` set) vs *Self-contained*
  (small project, no `docs/`). A rule for choosing.
- **Required vs optional sections** — a checklist table (section, required?, thin vs thick).
- **A canonical section order.**
- **A fill-in-the-blanks CLAUDE.md template** (fenced) with placeholders and short notes.
- **Per-section writing guidance** — what to put, what to avoid, drawn from the nine files.
- **Global-rules integration** — always include the workflow (plan/approve/log) and simple-English
  communication rules verbatim in spirit.
- **A final self-check list** before saving a new `CLAUDE.md`.

## Files to be changed / created

- `CLAUDE_MD_GUIDELINE.md` — new guideline document (the deliverable).
- `plans/20260718_104600_claude-md-guideline.md` — this plan.
- `change_log/<ts>_claude-md-guideline.md` — change log, written
  after implementation.

No existing project files will be modified. Only new files are created inside the
guidelines folder.

## Out of scope

- Editing any of the nine existing `CLAUDE.md` files.
- Creating the referenced `docs/` templates (architecture.md, security.md, etc.). The guideline
  will *point at* them but not generate them (can be a follow-up if wanted).
