# Change Log: Doc consistency fixes across the guideline set

Implements plan `plans/20260712_172254_doc-consistency-fixes.md`.

## Context

A critical review of the eight markdown files found contradictions between them, some
over-enforcing framing, and gaps in "what applies where". This change fixes the confirmed
problems. Per decision, `guideline.md`'s enforcing MUSTs were kept as-is; only outright
contradictions and wording were touched.

## Changes made

### flutter_build_flavors_guide.md
- **Symbol paths standardized.** All build-command examples now use the versioned layout
  `build/symbols/<platform>-<version>/` (Android split APK, Android appbundle, 16 KB section,
  iOS, Windows), matching the engineering standard, `release_process.md`, and `security.md`.
- **macOS flavor rule fixed.** Removed macOS from the "platforms that support `--flavor`"
  sentence so it no longer contradicts the same guide's later statement (and the engineering
  standard) that macOS desktop uses `--dart-define=APP_FLAVOR`.

### flutter_project_engineering_standard.md
- **Tier 1 layout** now shows `core/config/` instead of a top-level `config/`, with a note that
  `core/config/` is the fixed path in every tier (per `guideline.md §1`). Removes the conflict
  with `guideline.md`'s required path for a small app.
- **Tier 2 layout** now also shows `core/config/` (for `AppConfig` + `ConfigService`) alongside
  `app/config/` (app-level wiring), with a clarifying line so the "fixed across every tier"
  statement holds.
- **`intl`** example constraint bumped from `^0.19.0` to `^0.20.2` to match the Flutter 3.41 /
  Dart 3.11 era the docs target.
- **window_manager cross-note** added in §5.5: the concise `setMinimumSize` form here vs. the
  `WindowOptions` + `waitUntilReadyToShow` form in the flavors guide are two forms of the same
  setup; the flavors guide is the canonical desktop reference.

### security.md
- **Obfuscation wording (§8.1)** aligned with the measured framing used by the engineering
  standard and the flavors guide: it is a useful hardening step, not a strong security boundary
  on its own, and not a substitute for sound data security or secret management.

### architecture.md
- Added a "How to fill this in" note under the title: fill in only the sections that apply and
  mark the rest `N/A` with a reason, rather than inventing content. Matches the opt-in spirit of
  the rest of the set.

### guideline.md
- Neutralized the example `AppConfig.fallback` description from `'A local Android utility.'` to
  `'A Flutter application.'` (example wording only; the MUST rules are unchanged).

### README.md
- **Explainer links fixed.** They now point at the real `docs/` locations (previously
  root-relative and broken), list exactly which explainers exist, and state that `guideline.md`
  and `my_flutter_apps.md` have none (previously implied "most documents" had one).
- **Profile→document matrix added** ("What applies where (by profile)") so a reader can see at a
  glance which documents/sections switch on for Core Baseline / Production App Extension /
  Sensitive Data Extension, and that profiles stack.

## Out of scope (unchanged, by decision)
- `guideline.md`'s enforcing MUSTs and its About/keystore/folder rules.
- No new explainer files created; README wording simply stops implying they exist.

## Verification
- Grep confirms no non-versioned `build/symbols/...` paths remain outside the plan file.
- Grep confirms the only remaining macOS flavor reference is the correct desktop `APP_FLAVOR` rule.
