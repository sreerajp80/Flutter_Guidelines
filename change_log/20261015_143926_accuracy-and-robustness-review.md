# Change log — accuracy and robustness review

Implements plan: `plans/20261015_142014_accuracy-and-robustness-review.md`

## Summary

Fixed statements that were wrong, or that broke a build or a release when followed as written.
Removed version lock-in, so no rule is tied to one Flutter release. Rules now describe behavior
and say how to check the project's own toolchain (`flutter <command> -h`, the files
`flutter create` writes, official release notes). Version numbers remain only in the dated
"Reference snapshot — not a rule" boxes.

## Changes by file

### `flutter_project_engineering_standard.md`
- §5.2: one rule for passing the flavor on every platform — `--flavor` where
  `flutter build <platform> -h` lists it and the native setup exists, otherwise
  `--dart-define=APP_FLAVOR`. Removed "Windows and Linux do not accept `--flavor`". Kotlin DSL
  wording no longer names a release.
- §5.3: the JDK rule says "what AGP needs". The reference snapshot was refreshed (2026-10-15)
  and now lists the SDK levels and the JDK. New "Android Build Files — Rules For Any AGP
  Version": `kotlin { compilerOptions { } }` always; keep the `gradle.properties` flags Flutter
  writes; built-in Kotlin is an optional, separate migration. The "Extra steps when crossing
  Flutter 3.47" list became a "Known migrations" table keyed by trigger.
- §5.3.1: 16 KB — the deadline is now read from Android's page (Feb 1, 2027 when checked). Real
  checks: APK Analyzer, `zipalign -c -P 16`, `check_elf_alignment.sh`. Emulator: the "16 KB Page
  Size" system image. Removed the wrong `--analyze-size` check.
- §5.4: deployment target is "at least the Flutter minimum" (read from `flutter create`). The
  upload-SDK rule and the UIScene rule are now separate (UIScene is required by the SDK after
  iOS 26). Correct `FlutterImplicitEngineBridge` signature.
- §5.5: desktop flavor text follows §5.2; added the sqflite FFI native-library note. Replaced
  `ContextMenuRegion` (not a Flutter widget) with `MenuAnchor` / `showMenu`. MSIX: warning that
  `msix:create` rebuilds without hardening unless `build_windows: false`; one package per
  architecture. macOS: flavors via schemes or fallback; Intel / universal-binary note. Linux:
  package list points to the official setup page.
- §6.1: Material 3 / `material_ui` text without release numbers. Added
  `MaterialUiCompatibilityBridge` and the type-mismatch limit.
- §6.5: animate with `FadeTransition` / `*Transition` / `AnimatedOpacity`, not by rebuilding
  `Opacity`. Uses `MediaQuery.disableAnimationsOf`.
- §6.8: edge-to-edge treated as permanent (the opt-out ends at target API 36).
- §7.2: WCAG large text in sp (≈24 sp, ≈18.7 sp bold). §7.3: `ExcludeSemantics` /
  `excludeFromSemantics`; tooltip semantics are not "long-press only". §7.4: no version.
- §7.8: removed `IconSlider`; `NavigationRailDestination` has no `tooltip:` — wrap its icon in
  a `Tooltip`.
- §8.1: do not use the generated `AppLocalizations.localizationsDelegates`; corrected why
  `flutter_localizations` is still needed; gen-l10n output text without a version.
- §8.8: `Icon(matchTextDirection: true)` for RTL mirroring.
- §9.1, §11.1: version numbers removed.
- §10.0: Impeller described without versions, with the opt-out switch for each platform.
- §10.6 / §10.7: size analysis is a separate throw-away build (cannot combine with
  `--split-debug-info`; one ABI on Android; run before the hardened build).
- §13.1 / §13.2: sqflite already wraps migrations in a transaction; WAL via `rawQuery`.
- §15.2: `FLAG_SECURE` without the unmaintained `flutter_windowmanager`; UIScene hook wording.
- §16.5 / §16.6: build hooks without version; replaced the non-existent `flutter pub licenses`.
- §17.1–§17.3: SVG not "tree-shakeable"; per-platform asset filter "where supported";
  resolution variants need only the main folder; icon tree-shaking rule corrected.
- §18.4: performance tests with `integration_test` + `watchPerformance` + `flutter drive
  --profile` (removed `flutter test --profile`). §18.5: previews import
  `package:flutter/widget_previews.dart`, no version claims.
- §19.1: CI pins the exact Flutter version, not only `channel: stable`.

### `flutter_build_flavors_guide.md`
- Header and Toolchain Prerequisites: version-independent, with where to find each minimum;
  snapshot refreshed.
- "How The Flavor Reaches Dart Code": one rule for all platforms (`--flavor` or fallback).
- Android Toolchain Baseline: keep Flutter's `gradle.properties` flags; built-in Kotlin optional
  with steps and rollback.
- ProGuard / R8: Flutter adds engine rules and the app's `proguard-rules.pro` itself; no
  blanket rules.
- 16 KB: real checks; points to the standard for the deadline.
- iOS: UIScene rules corrected; `AppDelegate` sample now uses `FlutterImplicitEngineBridge`.
- New "Desktop Flavors — Which Flag To Use"; Windows / macOS / Linux sections rewritten (both
  flag forms; MSIX `build_windows: false`; one architecture per MSIX; Linux CMake snippet that
  works with `--flavor` and with the fallback). Release matrix and "Notes For New Projects"
  updated.

### `release_process.md`
- §5 flavor note, §6.1 symbols (`flutter symbolize`), §6.2 R8, §6.3 size analysis (separate
  build, run first), §6.4 backup and device-transfer rules for API 31+.
- §8 checklist: no `flutter pub licenses`; packaging tools must package the hardened build;
  backup rules.
- §9–§11B: size analysis moved before the hardened build; Windows packages the hardened build
  (`build_windows: false`); macOS architecture check; `--flavor` / fallback in commands; snap /
  Flatpak hardening note. §12: no `.msixbundle`.

### `platform_store_readiness.md`
- §2.2–§2.4: Play does not use Flutter's Dart symbol files (native symbols are separate);
  16 KB and edge-to-edge pointers; backup rules.
- §4.2: one MSIX per architecture. §5.1: universal vs arm64 as a decision.
- §5.2: Xcode archive reuses the obfuscation flags from `Flutter-Generated.xcconfig`.
- §5.3: note on `codesign --deep`. §6.2 / §6.3: snap and Flatpak builds must be hardened too.

### `security.md`
- §8.2 R8 wording; §10 Android backup rules (`dataExtractionRules`, device transfer),
  `FLAG_SECURE` without the unmaintained plugin; iOS hook wording without versions; §13 Android
  and Windows purge notes corrected.

### `guideline.md`
- §1.4 doc comment fixed (analyzer info). §2.4 obfuscation wording; size-analysis warning.
  §4 checklist: JDK and `gradle.properties` wording.

### `CLAUDE_MD_GUIDELINE.md`, `AGENTS_MD_GUIDELINE.md`
- Command block and flavor note use the generic flavor rule, the MSIX step and the
  size-analysis warning.

### Explainers
- `docs/flutter_build_flavors_guide_README.md` rewritten to match the guide.
- `docs/flutter_project_engineering_standard_README.md` and `docs/release_process_README.md`
  updated.

## Verification

- All Dart samples from `guideline.md` and the engineering standard (config model and loader,
  badge widget, fallback delegates, locale controller, tooltip test helper, performance test
  and driver, preview) were copied into a scratch app outside the repository and pass
  `flutter analyze` with no issues on the latest stable Flutter. The original preview sample
  failed (missing `widgets.dart` import); it was fixed.
- Commands were checked against `flutter <command> -h` (desktop `--flavor`, `--analyze-size`
  limits, `flutter pub` subcommands, `flutter test` flags) and the `msix` package page
  (`build_windows`, `windows_build_args`, `--identity-name`, `--display-name`).
- A grep for the old claims and for version-tied wording finds only intentional hits: negative
  examples ("do not add …"), the `*.msixbundle` ignore pattern, and the dated snapshot boxes.
- No Gradle build was run, so no Gradle daemons were started.

## Not changed

`architecture.md`, `DOCS_FOLDER_GUIDELINE.md`, `GUIDELINES_MANIFEST.md`, `README.md`,
`AI_AGENT_START_HERE.md`, the language pack and the profile example. Their cross-references to
section numbers still resolve; section numbering in the edited files is unchanged.
