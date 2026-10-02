# Change Log: Guidelines updated for Flutter 3.47 / Dart 3.13 and the AGP 9 toolchain

**Date:** 2026-10-02

**Plan:** [../plans/20261002_221017_flutter-3-47-toolchain-update.md](../plans/20261002_221017_flutter-3-47-toolchain-update.md)

## 1. What changed

| File | Change |
|---|---|
| `flutter_build_flavors_guide.md` | Banner now says Flutter 3.47 / Dart 3.13. Toolchain table: Flutter 3.47, Dart 3.13, Java 17, AGP 9.1.0, KGP 2.4.0 (new row), Gradle 9.3.1 (new row), iOS 15, macOS 12 (new row), Xcode 27 / UIScene note. New section **"AGP 9 / Gradle 9 Baseline"** with `settings.gradle.kts` plugin versions, the Gradle wrapper line, the `kotlin { compilerOptions { } }` block (replaces the rejected `kotlinOptions { }`), the migrator's `gradle.properties` flags, and `ExecOperations` for custom tasks. R8 note, iOS section (iOS 15, Xcode 27) and "Notes For New Projects" updated. |
| `flutter_project_engineering_standard.md` | §5.3 Toolchain Requirements rewritten for AGP 9.1.0 / KGP 2.4.0 / Gradle 9.3.1 / Java 17 with a `compilerOptions` sample. New **"Upgrading An Existing App To Flutter 3.47"** checklist. §5.4 iOS: minimum iOS 15, macOS 12, Xcode 27 UIScene note. §6.1 "Forward-Looking: Material And Cupertino Decoupling" replaced by **"Material And Cupertino Packages"**: `material_ui` / `cupertino_ui` are required, pinned with `^`, new imports, and the `dart fix --apply --code=migrate_design_widgets` command. §8.1 sample uses `material_ui` and `GlobalMaterialLocalizations.delegates`; pubspec sample adds `material_ui` / `cupertino_ui` and explains why `flutter_localizations` stays. §8.3.1 Sanskrit delegate sample now shows the new imports and registers `...GlobalMaterialLocalizations.delegates`. §10.0 Impeller now default on desktop too. §12.1 dev-dependency note no longer claims "as of Flutter 3.41". §16 `flutter_lints` → `^6.0.0`. §18.5 Widget Previews marked stable. |
| `docs/flutter_build_flavors_guide_README.md` | Banner, section summaries, "Toolchain pinning" Must row, AI knowledge list, the `CLAUDE.md` toolchain rules block, and review questions updated (AGP 9 required, no `kotlinOptions`, iOS 15). |
| `docs/flutter_project_engineering_standard_README.md` | Banner and section summaries updated (AGP 9 toolchain, upgrade checklist, `material_ui` / `cupertino_ui`, Impeller on desktop, Widget Previews stable, §8.1 delegates). |
| `CLAUDE_MD_GUIDELINE.md`, `AGENTS_MD_GUIDELINE.md` | Identity table placeholder → `<3.47.x or higher>`. New template rule: import `material_ui` / `cupertino_ui`, never `package:flutter/material.dart`. |
| `guideline.md` | §4 checklist: two new MUST items (Flutter 3.47 Android toolchain; Material/Cupertino from `material_ui` / `cupertino_ui`). |

## 2. Sources checked

- The official "What's new in Flutter 3.47" release post.
- An app already upgraded to Flutter 3.47 (its `pubspec.yaml`, Android Gradle files, Gradle
  wrapper and Sanskrit delegate file), used as ground truth for the code samples.

## 3. Not changed (as planned)

- `architecture.md`, `security.md`, `release_process.md`, `DOCS_FOLDER_GUIDELINE.md`,
  `README.md`, `GUIDELINES_MANIFEST.md` — no version-specific content.
- Code-generation package versions in §12.1 (`build_runner`, `freezed`, …) — not verified.
- A machine-specific Kotlin incremental-build workaround seen in the upgraded app was left out,
  because it is a local setup detail, not a project rule.
