# Change Log: Common Folder-Structure Guideline

**Implements plan:** `plans/20260712_162551_folder-structure-guideline.md`
**Date:** 2026-07-12

## What was done

Analysed the 8 Flutter apps in `my_flutter_apps.md` for how they store About-screen
constants, the release keystore, and their `lib/` folder layout, then created a single
shared guideline document.

### Files created

- `guideline.md` — the common convention for all Flutter apps, covering:
  1. **About-screen constants** — the reference Text App JSON pattern:
     `assets/config/app_config.json` (source of truth) loaded via
     `lib/core/config/app_config.dart` (`AppConfig` model with `fromJson` + `fallback`)
     and `lib/core/config/config_service.dart` (`ConfigService` with graceful fallback and
     `package_info_plus` version check). Includes reference code and JSON shape.
  2. **Release keystore + `key.properties`** — both in `android/`; `.jks` filename is the
     user's choice per app; properties file fixed as `android/key.properties`; both
     git-ignored with the required `.gitignore` lines.
  3. **Standard `lib/` folder structure** — a baseline layout with per-folder purpose and
     a rule that larger apps may layer further but must keep the About and keystore rules
     fixed.
  4. A quick checklist for new or migrated apps.

### Notes / deviations from the original draft

Two changes were made after user feedback before implementation:
- About constants use the **JSON-asset** approach (TextApp) instead of the originally
  proposed Dart `AppInfo` class (ContactSphere).
- Keystore `.jks` **filename is left to the user per app** instead of a fixed
  `release-keystore.jks`.

### Not changed

No source files in any of the 8 apps were modified. Migrating each app to follow the
guideline remains a separate, future task.
