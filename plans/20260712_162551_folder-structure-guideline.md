# Plan: Common Folder-Structure Guideline for All Flutter Apps

**Status:** completed

## What is the issue

The 8 Flutter apps listed in `my_flutter_apps.md` do not share a common convention for
three things the user cares about:

1. **About-screen constants** are stored in different files, folders, and class names:
   - `lib/config/app_constants.dart` (MantraJapaCounter) — class `AppConstants`
   - `lib/config/build_config.dart` (qr_reader)
   - `lib/config/` + `lib/utils/constants.dart` (Authenticator)
   - `lib/core/config/app_config.dart` (TextApp)
   - `lib/core/constants/app_constants.dart` (todo_app) — class `AppConstants`
   - `lib/src/about_constants.dart` (youtube_shortcut) — class `AboutConstants`
   - `lib/constants/app_info.dart` (ContactSphere) — class `AppInfo`
   - none / inline (daily_rule_cards)

2. **Release keystore + properties file** are named and placed inconsistently:
   - Keystore file: `keystore.jks`, `release-keystore.jks`, `todo_app.jks`
   - Properties file: mostly `key.properties`, but youtube_shortcut uses `keystore.properties`
   - Location: all already inside `android/` (this part is consistent)

3. **Overall `lib/` folder structure** ranges from flat (`config/`, `screens/`, `services/`)
   to layered clean-architecture (`domain/`, `data/`, `application/`, `presentation/`) to
   feature-first (`core/`, `formats/`, `shell/`, `sync/`). No shared baseline.

There is no single document that tells a future project (or a refactor of an existing one)
where these things MUST live.

## The plan for the fix

Create one new file, `guideline.md`, at the repo root.
It will define the shared conventions below. **No existing app is modified by this plan** —
this plan only produces the guideline document. Migrating each app to follow it will be a
separate, later effort.

### Recommended conventions the guideline will state

**A. About-screen constants (single source of truth) — JSON asset**
- Adopt the **TextApp pattern** (the user's chosen best approach):
  - Data file: `assets/config/app_config.json` — the single source of truth for the
    About screen. Editing About content is a config edit, not a code change.
  - Registered in `pubspec.yaml` under `assets:` as `- assets/config/`.
  - Typed model: `lib/core/config/app_config.dart` — class `AppConfig` with
    `fromJson`, and a built-in `AppConfig.fallback` so a missing/malformed file never
    crashes the app.
  - Loader: `lib/core/config/config_service.dart` — class `ConfigService` that loads
    from the asset bundle, degrades to `fallback` on any error, and optionally
    cross-checks `version`/`build` against `package_info_plus`.
- Standard JSON shape: `appName`, `description`, `version`, `build`, and a `details`
  map (Author, Email, License, AI used, IDE used, …).

**B. Release keystore + properties**
- Keystore file: `android/<name>.jks` — **filename is the user's choice per app**
  (e.g. `keystore.jks`, `release-keystore.jks`, `todo_app.jks`). The only fixed
  rules are: it lives in `android/`, and `key.properties` `storeFile` points to it.
- Properties file: `android/key.properties` (fixed name, in `android/`).
- Both MUST be git-ignored; guideline will show the required `.gitignore` lines and the
  standard `key.properties` key set (`storePassword`, `keyPassword`, `keyAlias`, `storeFile`).

**C. Standard `lib/` folder structure**
- Define a recommended baseline layout (e.g. `core/config/`, `models/`, `services/`,
  `repositories/`, `providers/` or `state/`, `screens/`, `widgets/`, `utils/`,
  `l10n/`, `theme/`) with a one-line purpose for each, plus a note that larger apps MAY
  layer further but MUST keep the `assets/config/app_config.json` + `core/config/` About
  pattern and the keystore rules fixed.

### Files to be changed / created

- **Create:** `guideline.md` (the new guideline document).
- **Create:** `plans/` (this plan lives here).
- No source files in any of the 8 apps are touched.

## After implementation

Write a change log to `change_log/` per the workflow rules.
