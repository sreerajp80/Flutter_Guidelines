# Plan: Top-level README index + remove keystore duplication

**Status:** completed

## What the user asked for

1. Create a **top-level `README.md`** that acts as an index/hub for the whole guideline set.
2. **Remove the keystore duplication** by making `guideline.md §2` the single source of
   truth and pointing the other files at it.

## Decision already made

The user chose the authoritative keystore location: **inside `android/`** (i.e.
`android/<name>.jks` with a relative `storeFile`, protected by `.gitignore`), exactly as
`guideline.md §2` already states. So `guideline.md §2` is the source of truth and the
other files must be reconciled to it.

## The issue

- There is a real **contradiction**, not just duplication, between two files:
  - `guideline.md §2.1` — keystore MUST live in `android/`, relative `storeFile`,
    gitignore is `android/*.jks` / `android/*.keystore`.
  - `flutter_build_flavors_guide.md` Steps 1–2 (lines ~173–208) — says the opposite:
    store the `.jks` **outside** the project (`~/keys/myapp-prod.jks`), use an
    **absolute** `storeFile`, gitignore is global `*.jks` / `*.keystore`.
- Lighter overlaps that are complementary (policy, not step-by-step), so they only need a
  cross-link, not a rewrite:
  - `release_process.md §7` — signing/secret policy (backup, not-in-source-control).
  - `flutter_project_engineering_standard.md` — keystores only appear in the `.gitignore`
    list (§20) and the Definition of Done (§23).
- There is **no top-level `README.md`** yet. Each doc has its own `<name>_README.md`
  explainer, but nothing ties them together or points a newcomer to a starting point.

## Files to be changed

1. **`README.md`** (NEW) — top-level index/hub.
2. **`flutter_build_flavors_guide.md`** (EDIT) — reconcile the Android signing section to
   `guideline.md §2` and point to it.
3. **`release_process.md`** (EDIT) — add a "see `guideline.md §2`" cross-link in §7.
4. **`flutter_project_engineering_standard.md`** (EDIT) — add a "see `guideline.md §2`"
   cross-link where keystores are mentioned (§15 Security or §20 Git hygiene).

`guideline.md`, `architecture.md`, `security.md`, and all `*_README.md` files are **not**
changed.

## The plan for each file

### 1. `README.md` (new)

A short hub page, in simple English, containing:
- One line on what this repository is (a shared guideline set for the user's Flutter apps).
- A table listing each document with a one-line purpose and a link:
  - `guideline.md` — personal cross-app conventions (About-screen JSON, keystore rules,
    `lib/` layout). **Source of truth for keystore rules.**
  - `flutter_project_engineering_standard.md` — the master, project-agnostic rulebook.
  - `architecture.md` — per-project architecture blueprint template.
  - `flutter_build_flavors_guide.md` — platform-by-platform build-flavor reference.
  - `release_process.md` — release runbook.
  - `security.md` — per-project security blueprint template.
  - `my_flutter_apps.md` — the list of apps these guidelines apply to.
- A short "Where do I start?" note (new app → read `guideline.md` +
  `flutter_project_engineering_standard.md`; shipping → `release_process.md`; etc.).
- A note that each doc has a matching `<name>_README.md` plain-English explainer.

### 2. `flutter_build_flavors_guide.md`

Reconcile the Android signing section to match `guideline.md §2`:
- **Step 1 (Create the keystore):** change "store the output `.jks` file securely outside
  the project directory" and the `keytool -keystore ~/keys/myapp-prod.jks` example to place
  the keystore at `android/<name>.jks`. Keep the backup + password-manager advice.
- **Step 2 (`key.properties`):** change `storeFile=/absolute/path/to/myapp-prod.jks`
  (absolute) to a relative `storeFile=<name>.jks` consistent with `guideline.md §2.2`, and
  rewrite the "Use an absolute path" paragraph to explain the relative-path convention
  instead.
- **Step 3 (gitignore):** change global `*.jks` / `*.keystore` to `android/key.properties`,
  `android/*.jks`, `android/*.keystore` to match `guideline.md §2.3`.
- Add a short pointer at the top of the signing section:
  "Keystore location, `key.properties` naming, and the `.gitignore` rules follow
  `guideline.md §2` (the source of truth); this section only adds the flavor-specific
  Gradle wiring."
- Leave the `build.gradle.kts` example and the Gradle-guard caveats unchanged (they already
  work with a relative `storeFile` via `file(props.getProperty("storeFile"))`).

### 3. `release_process.md`

In §7 (Signing And Secret Handling), add one bullet:
"Keystore location, `key.properties` naming, and `.gitignore` rules: see `guideline.md §2`."
No other content changes — the backup/two-location policy stays.

### 4. `flutter_project_engineering_standard.md`

Add a one-line cross-reference near the keystore/`.gitignore` mention (§15 Security core
rules or §20 Git hygiene): "Keystore + `key.properties` conventions: see `guideline.md §2`."

## Out of scope (noted, not changed unless you ask)

- `architecture.md §22 Related Documents` lists files under a `docs/` prefix (e.g.
  `docs/release_process.md`) and references a `README.md`, but the actual files are flat in
  the repo root. This path mismatch is pre-existing. I can fix it in this change or leave it;
  tell me your preference. Default: leave it.

## Verification

- Re-grep for `keystore` / `key.properties` / `.jks` across all docs and confirm every
  step-by-step instance now matches `guideline.md §2` (inside `android/`, relative
  `storeFile`, `android/*.jks` gitignore) or is a cross-link to it.
- Confirm `README.md` links resolve to existing files.

## After implementation

Write a change log to `change_log/` referencing this plan.
