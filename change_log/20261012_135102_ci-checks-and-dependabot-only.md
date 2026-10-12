# Change log: GitHub CI does only analyze/test checks and Dependabot

Implements plan: `plans/20261012_134608_ci-checks-and-dependabot-only.md`

## Summary

New rule for all projects: GitHub CI runs only the analyze/test checks and Dependabot. CI never
builds release artifacts, signs, holds signing secrets, uploads, or publishes. Release builds
and signing are done locally by the release owner.

## What changed

- `flutter_project_engineering_standard.md`
  - §19 rewritten as "CI Standard (Checks And Dependabot Only)":
    - §19.1 Required CI Checks (MUST for every repo): allowed steps only, no secrets,
      `contents: read`, example `.github/workflows/ci.yml`.
    - §19.2 Dependabot (MUST): `pub` and `github-actions` required, `gradle` optional,
      example `.github/dependabot.yml`; major updates are reviewed and follow the
      plan → approve → log workflow.
    - §19.3 What CI Must Not Do: no release builds, signing, secrets, uploads, releases, or
      other workflows. Kept the symbols folder layout note for local release builds.
    - Old §19.3 Pre-Commit is now §19.4 (text unchanged).
    - Removed the old per-platform release build jobs example.
  - Project tree: `.github/` comment names `workflows/ci.yml` and `dependabot.yml`.
  - `msix_config` comment: certificate passed in at local release build time.
  - Size budget: size tracked at each local release build.
  - §1.2 profile table: "Release, CI, UX…" → "Release, UX…".
- `AI_AGENT_START_HERE.md`: bootstrap step 6 and the store-ready checklist use the new rule.
- `guideline.md`: §4 shipped-apps checklist line uses the new rule.
- `flutter_build_flavors_guide.md`: removed CI wording, removed the "CI-managed signing"
  strategy (old Strategy C is now Strategy B), removed the CI note under Step 2 and the CI
  caveat, added a short "signing is never done in GitHub CI" note.
- `release_process.md`: signing location and rules no longer mention a CI secret store; Windows
  certificate comes from the local secret store; release checklist adds "built locally" and a
  license check line.
- `CLAUDE_MD_GUIDELINE.md`, `AGENTS_MD_GUIDELINE.md`: new workflow rule 5 "CI is checks +
  Dependabot only" in the template; added to the "keep verbatim in spirit" list and the final
  self-check.
- `docs/flutter_build_flavors_guide_README.md`, `docs/flutter_project_engineering_standard_README.md`,
  `docs/release_process_README.md`: summaries match the new rule.
- `language_packs/sanskrit_malayalam.md`: comment wording "CI gate: fail the build" →
  "CI check: fail".
- `platform_store_readiness.md` (small extra, not in the plan's file list): Flathub note says
  "release bundle you built locally" instead of "your CI produced".

## Follow-up for projects

Each project must update its `CLAUDE.md` and `AGENTS.md` with workflow rule 5, replace any
release/signing workflows in `.github/workflows/` with the checks-only `ci.yml`, add
`.github/dependabot.yml`, and remove signing secrets from GitHub. Each project does this as its
own planned change.
