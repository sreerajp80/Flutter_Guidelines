# Plan: GitHub CI does only analyze/test checks and Dependabot

**Status:** completed

## What the issue is

The guidelines now say CI MUST build a signed-style release for every declared platform
(engineering standard §19.2), and several other files build on that (CI-managed signing,
"CI builds a hardened release" in the store-ready checklist, CI secret store for signing).

The new rule: **GitHub CI in every project does only two things:**

1. **Checks** — get packages, code generation (only when needed for analyze/test), format check,
   `flutter analyze`, `flutter test` (this includes the l10n parity test and any language-pack
   check script, which are checks, not builds).
2. **Dependabot** — a committed `.github/dependabot.yml` that keeps dependencies up to date.

CI MUST NOT build release artifacts, sign anything, hold signing secrets, upload to stores, or
create releases. Release builds (with `--obfuscate --split-debug-info`) and signing are done
locally by the release owner, following `release_process.md`.

This rule is mandatory for all projects, so it also goes into the inline rules that every
project's `CLAUDE.md` and `AGENTS.md` must carry.

## Files to be changed

1. `flutter_project_engineering_standard.md`
   - §19 rewritten: "CI Standard (checks and Dependabot only)".
     - §19.1 Required CI — MUST for every repo; allowed steps only; GitHub Actions example
       workflow (`.github/workflows/ci.yml`) on pull requests and pushes to the main branch;
       no secrets; least permissions (`contents: read`).
     - §19.2 Dependabot — MUST; example `.github/dependabot.yml` with `pub` (root) and
       `github-actions` ecosystems, weekly; `gradle` for `android/` MAY be added. Dependabot PRs
       pass the same §19.1 checks before merge.
     - §19.3 What CI must not do — list (release builds, signing, secrets, store upload,
       publishing releases, deploys). Release builds are local (`release_process.md`).
     - The old "Additional recommended steps" move: generated-file staleness check stays in CI
       (it is a check); license check and size check move to the local release steps.
     - §19.4 Pre-Commit (old §19.3, unchanged text).
   - Line ~174 project tree: comment on `.github/` (`workflows/ci.yml`, `dependabot.yml`).
   - Line ~644 (`msix_config` comment): certificate passed in at local build time, not by CI.
   - Line ~2067: "Track size in CI" → track size at each local release build.
   - §1.2 profile table: "Release, CI, UX…" → "Release, UX…" (CI is now Core for everyone).
   - §23 Definition of Done line "Required CI checks pass" stays (still true).
2. `AI_AGENT_START_HERE.md`
   - Step 6 CI: "§19.1 checks + §19.2 Dependabot; no release builds in CI".
   - Store-ready checklist line "CI builds a hardened release…" → "Hardened release
     (`--obfuscate --split-debug-info`) built locally for every declared platform; CI runs only
     checks and Dependabot".
3. `guideline.md` (§4 checklist, line ~754): same change as above.
4. `flutter_build_flavors_guide.md`
   - Line ~7 and ~228: drop "CI pipelines / CI environment" wording.
   - Strategy A (~233): remove "CI writes the same file from environment variables".
   - Strategy B "CI-managed signing": remove it (CI must not sign). Renumber Strategy C → B and
     fix every reference to "Strategy C".
   - Line ~296: remove the "For CI environments…" sentence.
   - Caveats (~419): remove the Strategy B CI bullet.
5. `docs/flutter_build_flavors_guide_README.md` — signing strategy row: two strategies, no
   CI-managed; toolchain row: keep "CI image" (CI still runs analyze/test with a pinned Flutter).
6. `docs/flutter_project_engineering_standard_README.md` — §19 summary and "Set up CI" tip:
   checks + Dependabot, MUST, before the first merge.
7. `docs/release_process_README.md` (~41, ~115): "the CI workflow" no longer shaped by release
   decisions; reword to the local release build setup.
8. `release_process.md`
   - §7 signing location placeholder: drop "CI secret store"; rule "CI logs must not print
     signing secrets" → "CI never holds signing secrets".
   - Windows step 6 (~546): certificate from the local secret store, not the CI secret store.
   - Each platform's local release steps: add license check and size check if not already there.
   - Evidence line "CI run" stays (the check run for the release commit).
9. `CLAUDE_MD_GUIDELINE.md` and `AGENTS_MD_GUIDELINE.md`
   - Template workflow rules: add rule 5 **"CI is checks + Dependabot only."** (short inline
     text, link to engineering standard §19).
   - §5 / §6 "keep verbatim in spirit" and the final self-check: include the CI rule.
10. `language_packs/sanskrit_malayalam.md` (~124): "fail the build" → "fail the CI check"
    (wording only; the gate is a check, so it stays allowed).

No git submodule is touched (this repo has none).

## The plan for the fix

1. Make the edits above, keeping each file's style and section numbers (only §19 subsections
   are renumbered; search for links to "§19.2" / "§19.3" and update them).
2. Search again for `CI`, `release artifact`, `Strategy B`, `Strategy C`, `secret store` to be
   sure no old wording is left.
3. Write the change log in `change_log/` and set this plan to `completed`.

## Assumptions (please confirm or correct)

- "Analyze/test checks" includes `dart format` check, `build_runner` (when needed),
  the generated-files staleness check, and language-pack check scripts.
- Dependabot ecosystems: `pub` and `github-actions` are MUST; `gradle` is MAY.
- Release builds and signing move fully to the local machine of the release owner.
