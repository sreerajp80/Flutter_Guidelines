# Change log: Top-level README index + keystore de-duplication

Implements plan `plans/20260712_164322_readme-index-and-keystore-dedup.md`.

## What was changed

### 1. New `README.md` (top-level index)
Created a hub page in simple English: a table of all documents with a one-line purpose and
a link each, a "where do I start?" section, and a note about the per-file `<name>_README.md`
explainers. `guideline.md` is marked as the source of truth for keystore rules.

### 2. Keystore de-duplication — `guideline.md §2` is now the single source of truth
The user chose the authoritative keystore location: **inside `android/`** (relative
`storeFile`, protected by `.gitignore`). Other files were reconciled to it.

- **`flutter_build_flavors_guide.md`**
  - Added a pointer at the top of the Android signing section stating that keystore
    location, `key.properties` naming, and `.gitignore` rules follow `guideline.md §2`.
  - Step 1: keystore is now generated into `android/<name>.jks` (was `~/keys/...` "outside
    the project directory"). Backup/password advice kept.
  - Step 2: `storeFile` is now a relative filename `myapp-prod.jks` (was an absolute path);
    rewrote the "use an absolute path" paragraph to explain the relative convention and
    point to `guideline.md §2.2`.
  - Step 3 and the "Notes For New Projects" checklist: `.gitignore` rules scoped to
    `android/*.jks` / `android/*.keystore` (were global `*.jks` / `*.keystore`).

- **`release_process.md`** — §7 (Signing And Secret Handling): added a cross-link to
  `guideline.md §2`. Existing backup/secret policy unchanged.

- **`flutter_project_engineering_standard.md`** — §20.2 (Never Commit): added a cross-link
  to `guideline.md §2` on the keystores/signing-material line.

### 3. Consistency fix in an explainer (beyond the original plan scope)
`flutter_build_flavors_guide_README.md` still described the old unscoped `*.jks` /
`*.keystore` gitignore rule in three places, which would have contradicted the guide after
this change. Updated all three to `android/*.jks` / `android/*.keystore`. This was a direct
consequence of the dedup, so it was aligned rather than left inconsistent.

## Files not changed
`guideline.md`, `architecture.md`, `security.md`, and the other `*_README.md` files.

## Verification
- Re-grepped all `.md` files for keystore/`key.properties`/`.jks` references: every
  step-by-step instance now matches `guideline.md §2` (inside `android/`, relative
  `storeFile`, `android/*.jks` gitignore) or is a cross-link to it. No `~/keys`,
  "outside the project", or absolute-path guidance remains in the guide.
- Confirmed every link in the new `README.md` resolves to an existing file.

## Known pre-existing issue (left as-is, not in scope)
`architecture.md §22 Related Documents` lists docs under a `docs/` prefix and references a
`README.md`, but the actual files are flat in the repo root. Left unchanged per the plan;
can be fixed on request.
