# Plan: Update the guidelines for Flutter 3.47 / Dart 3.13 and the AGP 9 toolchain

**Status:** completed

## The issue

The guidelines say they "reflect Flutter 3.44 / Dart 3.12". Flutter 3.47.6 / Dart 3.13.5 is now
in use. Several rules in the guides are now wrong, and following them would break a project:

1. **AGP 9 is now required, not forbidden.** The guides say "AGP 8.x — NOT 9.x, migration is
   paused". Flutter 3.47 makes new projects with AGP 9.1.0, Kotlin Gradle Plugin 2.4.0,
   Gradle 9.3.1 and Java 17. AGP 9 rejects the old `kotlinOptions { }` block; the
   `kotlin { compilerOptions { jvmTarget... } }` block replaces it. The Flutter migrator also
   adds `android.builtInKotlin=false` and `android.newDsl=false` to `gradle.properties`.
   Gradle 9 removed `project.exec` (use `ExecOperations`).
2. **Material and Cupertino left the SDK.** They are now the pub packages `material_ui` and
   `cupertino_ui`. Imports change from `package:flutter/material.dart` /
   `package:flutter/cupertino.dart` to `package:material_ui/material_ui.dart` /
   `package:cupertino_ui/cupertino_ui.dart`. The migration is
   `dart fix --apply --code=migrate_design_widgets` (plain `dart fix --apply` does not do it).
   The tool may write `material_ui: any` — it must be pinned with `^`. The old SDK libraries
   are due to be formally deprecated in the November stable release. The guides still say
   "No action is required today".
3. **Localization delegates moved.** `GlobalMaterialLocalizations` now comes from `material_ui`
   and `GlobalCupertinoLocalizations` from `cupertino_ui`; only `GlobalWidgetsLocalizations`
   stays in `flutter_localizations`. `GlobalMaterialLocalizations.delegates` gives all three.
   The §8.1 and §8.3.1 code samples use the old imports.
4. **Other minimums moved:** iOS 13 → **iOS 15**, macOS 10.15 → **12**, Xcode 27 pipelines
   (UIScene is now mandatory with Xcode 27 builds).
5. **Other status changes:** Widget Previews are now **stable** (guide says experimental);
   Impeller is the default on **desktop** too (macOS, Windows, Linux) and Skia fallbacks are
   being removed; `flutter_lints` current line is `^6.0.0` (guide says `^5.0.0`).

Ground truth was checked against the real upgraded app `MantraJapaCounter` (its
`pubspec.yaml`, `android/settings.gradle.kts`, `android/app/build.gradle.kts`,
`android/gradle.properties`, gradle wrapper, and `lib/l10n/sa_material_localizations.dart`),
plus the official "What's new in Flutter 3.47" post.

## Files to be changed

| File | Change |
|------|--------|
| `flutter_build_flavors_guide.md` | "Reflects" banner → 3.47 / 3.13. Toolchain table: Flutter 3.47, Dart 3.13, Java 17, **AGP 9.1.0**, **KGP 2.4.0**, **Gradle 9.3.1**, iOS 15, Xcode 27 note. Kotlin DSL note: add `kotlin { compilerOptions }` sample, `kotlinOptions` removed, the two `gradle.properties` migrator flags, `ExecOperations` note for custom tasks. R8 note: drop "on AGP 8.x" wording. iOS section: iOS 15 minimum, UIScene mandatory with Xcode 27. "Notes For New Projects" list: same updates. |
| `flutter_project_engineering_standard.md` | §5.3 "Toolchain Requirements" → Flutter ≥ 3.47, AGP 9.1 / KGP 2.4 / Gradle 9.3.1 / Java 17, with `compilerOptions` sample. iOS minimum → 15 (+ macOS 12). §6.1 "Forward-Looking: Material And Cupertino Decoupling" → rewrite as "Material And Cupertino Packages" (MUST use `material_ui` / `cupertino_ui`, pinned with `^`; the `dart fix` command; new code MUST NOT import `package:flutter/material.dart`). §8.1 sample + pubspec: new imports and `material_ui` / `cupertino_ui` deps. §8.3.1 Sanskrit delegate sample: new imports. §10.0 Impeller: add desktop default. §12.1 version note "as of Flutter 3.41" → reword to "pin current major at project start" (versions themselves not changed — not verified). §16 `flutter_lints` → `^6.0.0`. §18.5 Widget Previewer → stable. Add a short **"Upgrading an existing app to Flutter 3.47"** checklist (in §5.3). |
| `docs/flutter_build_flavors_guide_README.md` | Banner, toolchain summary, "Must" pinning row, AI rules ("refuse AGP 9" → "use AGP 9.1 / KGP 2.4 / Gradle 9.3.1"), review questions (iOS 15, no `kotlinOptions`). |
| `docs/flutter_project_engineering_standard_README.md` | Banner, section summaries (AGP 9, iOS 15, material_ui/cupertino_ui, Widget Previews stable, Impeller desktop). |
| `CLAUDE_MD_GUIDELINE.md`, `AGENTS_MD_GUIDELINE.md` | Identity table placeholder `<3.41.x or higher>` → `<3.47.x or higher>`; add one line to the template rules: import `material_ui` / `cupertino_ui`, never `package:flutter/material.dart`. |
| `guideline.md` | §4 checklist: add "Material/Cupertino come from `material_ui` / `cupertino_ui` (pinned), not the SDK" and "Android toolchain is AGP 9.1 / KGP 2.4 / Gradle 9.3.1 / Java 17". |
| `README.md` / `GUIDELINES_MANIFEST.md` | Only if they name a Flutter version (current search found none) — otherwise untouched. |

No changes to `architecture.md`, `security.md`, `release_process.md`, `DOCS_FOLDER_GUIDELINE.md`
(no version-specific content found).

## Plan for the fix

1. Edit each file above, keeping the existing style (MUST/SHOULD wording, tables, short notes).
2. Keep project-neutral: no local paths, drive letters or machine details (the
   `kotlin.incremental=false` cross-drive workaround in MantraJapaCounter is a local quirk and
   will NOT go into the guides).
3. Re-run a search for `AGP 8`, `NOT 9`, `iOS 13`, `3.41`, `3.44`, `Dart 3.12`,
   `package:flutter/material.dart`, `experimental` to confirm nothing stale is left.
4. Write the change log in `change_log/` and set this plan to `completed`.

## Out of scope

- Updating your app projects themselves (each app runs its own `dart fix` + Gradle migration).
- Changing pinned code-gen package versions (`build_runner`, `freezed`, …) — not verified here.
