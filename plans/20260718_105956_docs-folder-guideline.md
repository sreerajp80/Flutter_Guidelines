# Plan: Guideline for creating files in a project's `docs/` folder

**Status:** completed

## What the issue is

Nine existing Flutter projects each keep a `docs/` folder. Two items in that folder are
already fixed and out of scope:

- `GUIDELINES_MANIFEST.md` — the portable pointer file (identical across apps).
- `guidelines/` — the shared guidelines Git submodule.

Everything **else** in `docs/` is the per-app documentation. By studying the nine folders,
these per-app files recur but drift in name, structure, and depth. Examples seen:

- `architecture.md` (every app)
- `security.md` / `security-rules.md` (every app)
- `release_process.md` / `release-signing.md` (most apps)
- `workflow-rules.md` (several apps)
- `*_Idea.md` / `*-Idea.md` (product concept)
- `implementation-plan.md` / `implementation_plan.md`
- `implementation-progress.md` / `implementation_progress.md`
- Occasional extras: `known-gaps.md`, `dependencies.md`, `project_structure.md`,
  `security-audit-phase*.md`, `phase8_performance.md`,
  `flutter_project_engineering_standard.md` + `flutter_build_flavors_guide.md`
  (these last two are really submodule docs copied locally in the older apps).

There is **no single guideline** that says *how* to create a file in a project's `docs/`
folder — what to name it, what sections it needs, what style to use — so future files keep
drifting.

Goal: create one guideline document that a person (or Claude) can follow to create a
consistent, high-quality file in any project's `docs/` folder, excluding
`GUIDELINES_MANIFEST.md` and the `guidelines/` submodule.

## What I found in the nine `docs/` folders (basis for the guideline)

Common conventions observed:

1. **Naming** — lowercase, mostly `snake_case` or `kebab-case` (both appear; needs a rule to
   pick one). Descriptive, no date prefix (unlike `plans/` and `change_log/`).
2. **Title line** — every file starts with an `# H1` title; app-specific docs append the app
   name (e.g. `# Architecture — AppName`).
3. **One-paragraph purpose** — a short "what this file is / read this before X" intro under the
   title.
4. **Cross-links** — relative markdown links to `../.agents/AGENTS.md`, sibling docs, and
   `guidelines/…` submodule docs. "Read X first" pointers.
5. **`---` section separators** and numbered `##` sections.
6. **Callouts** — `>` blockquote warnings for secrets/security.
7. **Checklists** — `- [ ]` / `[x]` for progress and backup checklists.
8. **Dated point-in-time docs** — audits/progress carry a `**Date:**` line.
9. **Simple English**, code fences with language, tables for structured data.
10. **Recognizable file "types"** each with a typical section skeleton (architecture, security,
    release, idea, implementation-plan, implementation-progress, known-gaps, dependencies).

## The plan for the fix

Create one new guideline file at the project root:

- **`DOCS_FOLDER_GUIDELINE.md`** (sits beside the existing `CLAUDE_MD_GUIDELINE.md`, same
  house style).

It will contain:

1. **Scope note** — applies to per-app files in `docs/`; explicitly excludes
   `GUIDELINES_MANIFEST.md` and the `guidelines/` submodule (never edit submodule contents
   locally).
2. **Local-copy vs submodule rule** — when to keep a doc local vs rely on the submodule
   (mirrors the manifest's "local copy wins" note); do not duplicate submodule docs locally
   unless intentionally overriding.
3. **Naming convention** — **canonical case is `snake_case`** (decided): lowercase, words
   joined by underscores, descriptive, no date prefix; reserve date-prefixed names for
   `plans/` and `change_log/`. Existing `kebab-case` files are tolerated (no forced rename)
   but new files must use `snake_case`.
4. **Standard file anatomy** — the shared skeleton every docs file follows: H1 title (+ app
   name), purpose paragraph, "read first" links, `---`-separated numbered sections, callouts,
   code fences, tables, checklists, simple English.
5. **Catalog of recognized file types** — a table listing each standard doc (architecture,
   security, release_process, workflow-rules, idea, implementation-plan,
   implementation-progress, known-gaps, dependencies, project_structure, audit), what it is
   for, and its recommended section skeleton.
6. **Cross-linking rules** — how to link to `AGENTS.md`/`CLAUDE.md`, siblings, and the
   submodule; relative paths.
7. **When to create a new doc vs extend an existing one** — keep the set small; prefer
   sections over new files.
8. **A short "new docs file" checklist** to run before saving.
9. **Note** that this pairs with `CLAUDE_MD_GUIDELINE.md` (which decides thin vs thick
   `CLAUDE.md`).

### Files to be changed / created

- **Create** `DOCS_FOLDER_GUIDELINE.md` (the deliverable).
- **Create** `plans/20260718_105956_docs-folder-guideline.md`
  (this plan).
- **Create** a change log under `change_log/` after implementation.

No files in the nine analyzed projects will be modified — they are read-only sources.

## Verification

- Confirm the guideline covers every per-app file type actually seen in the nine folders.
- Confirm `GUIDELINES_MANIFEST.md` and `guidelines/` are explicitly excluded.
- Confirm house style matches the existing `CLAUDE_MD_GUIDELINE.md`.
