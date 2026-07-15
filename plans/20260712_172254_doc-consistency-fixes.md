# Plan: Fix contradictions and improve navigability across the guideline docs

**Status:** completed

## What the issue is

A critical read of the eight markdown files found real contradictions between them,
some over-enforcing (rather than "implement what's needed") framing, and gaps that make
it hard to find "what is needed where". This plan fixes the confirmed problems only.
It does **not** rewrite the personal `guideline.md` MUSTs (kept as-is by decision).

### Confirmed contradictions to fix

1. **Debug-symbol path layout is inconsistent.**
   - `flutter_project_engineering_standard.md §19.2` requires `build/symbols/<platform>-<version>/`
     and says `release_process.md §6.1` and `security.md §8.1` depend on it (they do follow it).
   - `flutter_build_flavors_guide.md` breaks it in every build command
     (`build/symbols/prod/`, `.../android-prod/`, `.../ios-prod/`, `.../windows-prod/`) — no version —
     while its own "Debug Symbol Management" section mandates `<platform>-<version>/`.
   - **Decision:** standardize on `build/symbols/<platform>-<version>/` everywhere.

2. **`config/` folder location conflicts for Tier 1 apps.**
   - `guideline.md §1.1 / §3` mandate the fixed path `lib/core/config/` for `AppConfig` + `ConfigService`.
   - `flutter_project_engineering_standard.md §3.1` Tier 1 diagram shows `lib/config/` (no `core/`).
   - **Decision (guideline MUSTs kept):** `lib/core/config/` wins. Fix the Tier 1 diagram to show
     `core/config/`. The `assets/config/app_config.json` asset path is unchanged.

3. **macOS flavor mechanism contradicts itself.**
   - `flutter_build_flavors_guide.md` line ~61 lists macOS as a `--flavor` platform, but line ~78
     says macOS should use `APP_FLAVOR` like Windows; `engineering_standard §5.2` puts macOS in the
     `APP_FLAVOR` camp.
   - **Decision:** align on the doc's stated policy — remove macOS from the `--flavor` grouping so it
     matches the "desktop uses APP_FLAVOR" rule used consistently elsewhere.

4. **README explainer links are broken / overstated.**
   - `README.md §Plain-English explainers` links `[architecture_README.md](architecture_README.md)`
     (repo root), but the explainers actually live in `docs/`.
   - It also says "Most documents have a matching explainer" — but there is no `guideline_README.md`
     and no `my_flutter_apps_README.md`.
   - **Decision:** fix the links to `docs/…`, and reword so the claim matches which explainers exist.

### Softer inconsistencies to harmonize

5. **Obfuscation-strength wording** differs in confidence between `security.md §8.1` (most bullish),
   `engineering_standard §15.1`, and `flavors_guide` (measured). Align `security.md`'s wording with the
   measured "raises cost, not a strong boundary" framing already used by the other two.

6. **Two `window_manager` idioms** (`setMinimumSize` in engineering §5.5 vs. `WindowOptions +
   waitUntilReadyToShow` in flavors guide). Add a one-line cross-note so a reader knows they are two
   forms of the same setup and which doc is canonical (flavors guide), rather than editing the code.

7. **`intl: ^0.19.0`** in engineering §8.1 trails the "Flutter 3.41 / Dart 3.11" era the docs target.
   Bump the illustrative constraint to the current `^0.20.x` line (kept as an example, "pin at project start").

### "Implement what's needed" / findability improvements

8. **`architecture.md`** — add one line near the top: fill in only the sections that apply and mark the
   rest `N/A` rather than inventing content. Keeps it consistent with the opt-in spirit of the other docs.

9. **`guideline.md` fallback** hardcodes `'A local Android utility.'` (baked-in Android/local) — soften the
   *example* fallback description to a neutral general-Flutter string. (Wording of the example only; the
   MUST rules stay.)

10. **README profile→document matrix** — add a short table showing which documents/sections switch on per
    profile (Core Baseline / Production App Extension / Sensitive Data Extension), so a reader can see
    "what is needed where" at a glance.

## Files to be changed

- `flutter_build_flavors_guide.md` — items 1 (symbol paths in all build commands), 3 (macOS grouping).
- `flutter_project_engineering_standard.md` — items 2 (Tier 1 diagram → `core/config/`), 7 (`intl` bump).
- `security.md` — item 5 (obfuscation wording).
- `architecture.md` — item 8 (fill-in guidance line).
- `guideline.md` — item 9 (neutral example fallback string only).
- `README.md` — items 4 (explainer links + claim), 6 (window_manager cross-note if placed here; otherwise
  in the two platform docs), 10 (profile→document matrix).

No changes to: `my_flutter_apps.md`, and no changes to the `docs/*_README.md` explainer files unless the
README rewording requires a matching tweak (will note in the change log if so).

## Plan for the fix

1. Edit `flutter_build_flavors_guide.md`: replace every `--split-debug-info=build/symbols/...` example with the
   `build/symbols/<platform>-<version>/` form; remove macOS from the `--flavor`-supported sentence so it agrees
   with the APP_FLAVOR desktop rule.
2. Edit `flutter_project_engineering_standard.md`: change the Tier 1 diagram `config/` → `core/config/` and add a
   short note that `core/config/` is fixed (ref `guideline.md §1`); bump the `intl` example constraint.
3. Edit `security.md §8.1`: align obfuscation wording with the measured framing.
4. Edit `architecture.md`: add the "fill in only what applies, mark the rest N/A" line under the title.
5. Edit `guideline.md`: neutralize the example fallback description string.
6. Edit `README.md`: fix explainer links to `docs/…`, correct the "most documents" claim, add the
   profile→document matrix, and add the window_manager cross-note.
7. Re-scan the edited docs for any newly introduced cross-reference drift.
8. Write the change log to `change_log/`.

## Out of scope (by decision)

- Rewriting `guideline.md`'s enforcing MUSTs or its About/keystore/folder rules (kept as-is).
- Creating missing explainer files (`guideline_README.md`, `my_flutter_apps_README.md`) — README wording
  will simply stop implying they exist.
