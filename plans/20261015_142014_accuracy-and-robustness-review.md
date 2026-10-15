# Accuracy and robustness review of the guideline set

**Status:** completed

## Goal

Make the guideline set accurate and version-independent. Remove instructions that are wrong, or
that break a build or a release when followed as written. Do this without tying any rule to one
Flutter version. The facts were checked against the latest stable toolchain on 2026-10-15
(Flutter 3.47.6 / Dart 3.13.5).

## Writing rule for every fix: no version lock-in

This is a generic guideline. It MUST NOT be tied to one Flutter version. Every fix in this plan
follows these rules:

- **State the rule by behavior, not by version.** Write "if `flutter build windows -h` lists
  `--flavor`, use it", not "since Flutter 3.47, use it".
- **Tell the reader how to check.** Point to `flutter --version`, `flutter <command> -h`, the files
  `flutter create` writes, and the official release notes / breaking-changes page.
- **Cover both cases where tools differ.** For example: desktop `--flavor` when the toolchain
  supports it, `--dart-define=APP_FLAVOR` when it does not. A project on an older or newer Flutter
  must still get a correct answer.
- **Version numbers appear only in the existing, dated "Reference snapshot — not a rule" boxes.**
  They stay as examples to spot a project that is clearly behind. Nowhere else.
- **Migrations are described by their trigger**, not by a release number. For example: "when
  `dart fix` offers `migrate_design_widgets`", "when the project is on AGP 9 or later", "when you
  build with an SDK that requires UIScene".

The version numbers in the issue table below only record **what was found during this check**.
They will not be copied into the guidelines as rules.

## How the facts were checked

- A scratch Flutter 3.47.6 app (outside this repo) was created with `flutter create`. Its Android
  files, `flutter gen-l10n` output, and `flutter <command> -h` text were compared with the guides.
- The Dart code in `guideline.md` §1 and engineering standard §7.8, §8.3.1 and §8.4 was compiled
  with `material_ui` 1.7.0 / `cupertino_ui` 1.1.3. It passes `flutter analyze`. The only finding
  is one doc-comment lint.
- Official sources: Flutter "What's new in 3.47", the `material_ui` migration page, the built-in
  Kotlin migration page, the UIScene adoption page, the Windows and Linux flavor pages, Android's
  16 KB page-size page, and the `msix` package page on pub.dev.

## The issues

### A. Wrong or out of date — a reader who follows them gets a failure

| # | Where | Today it says | Fact |
|---|---|---|---|
| A1 | Standard §5.2, §5.5; flavors guide ("How the flavor reaches Dart", Windows / macOS / Linux sections, release matrix); `release_process.md` §5, §6.3, §11–§11B; `CLAUDE_MD_GUIDELINE.md`, `AGENTS_MD_GUIDELINE.md` command blocks; flavors explainer | Windows and Linux do not accept `--flavor` (flutter/flutter#98994), so desktop MUST use `--dart-define=APP_FLAVOR` | Since Flutter 3.47, Windows and Linux support `--flavor` without project changes. It sets `appFlavor`, gives a flavor-specific build folder, and filters flavor assets. `flutter build windows -h` / `linux -h` list `--flavor`. |
| A2 | Standard §5.3.1, §10.6, §10.7; `release_process.md` §6.3, §9 | Run `--analyze-size` together with the release build, or as `flutter build apk --flavor prod --release --analyze-size` | `flutter build -h`: `--analyze-size` "cannot be combined with `--split-debug-info`" (so not with `--obfuscate`). On Android it needs one ABI (`--target-platform android-arm64`). It also writes to the same output folder, so running it after the hardened build **replaces the hardened artifact with an un-obfuscated one**. |
| A3 | Flavors guide (MSIX), standard §5.5.1, `release_process.md` §11 | `flutter build windows --obfuscate ...` then `dart run msix:create` | `msix:create` runs `flutter build windows` again by default (`build_windows: true`), without our flags. The packaged app is **not obfuscated** and has no `APP_FLAVOR`. Fix: set `build_windows: false` (package the build we just made), or pass the flags in `windows_build_args`. |
| A4 | Flavors guide (MSIX config), `release_process.md` §12, `platform_store_readiness.md` §4.2 | `architecture: x64, arm64` builds a `.msixbundle` | `msix` takes one architecture (`x64` or `arm64`) per package. Ship one package per architecture. Flutter builds Windows arm64 only on an arm64 machine. |
| A5 | `platform_store_readiness.md` §2.3 | Upload `build/symbols/` to Play as native debug symbols so traces de-obfuscate | Play does not use Flutter's Dart symbol files. They are used with `flutter symbolize` (or a crash tool that accepts them). Play "native debug symbols" are a different thing: AGP `ndk.debugSymbolLevel`. |
| A6 | Standard §5.3.1, flavors guide (16 KB) | Play deadlines Nov 1 2025 / May 31 2026; check with `appbundle --analyze-size` / `unzip -l`; emulator "Edit → Advanced → Page size: 16 KB" | Android's page now says: from **Feb 1, 2027** updates that do not support 16 KB cannot be released (apps targeting Android 15+). Real checks: APK Analyzer alignment column, `zipalign -c -P 16 -v 4`, `check_elf_alignment.sh`, `bundletool dump config`. Emulator: install a "Google APIs Experimental 16 KB Page Size" system image; confirm with `adb shell getconf PAGE_SIZE` = `16384`. The "Flutter ≥ 3.24" claim has no official source; remove it. |
| A7 | Standard §5.4, §15.2; flavors guide (iOS); `security.md` §10 | Apple requires UIScene for apps built with the iOS 26 SDK; UIScene since Flutter 3.38 | UIScene is required by the **iOS 27 SDK (Xcode 27)**. Flutter added it in 3.38 and made it the default in 3.41. The iOS 26 SDK is the App Store **upload** minimum (since April 2026), which is a separate rule. |
| A8 | Flavors guide (iOS `AppDelegate` sample) | `func didInitializeImplicitFlutterEngine(_ engine: FlutterEngine)` and `GeneratedPluginRegistrant.register(with: engine)` | Official signature: `didInitializeImplicitFlutterEngine(_ engineBridge: FlutterImplicitEngineBridge)` and `GeneratedPluginRegistrant.register(with: engineBridge.pluginRegistry)`. The current sample does not compile. |
| A9 | Standard §5.2, flavors guide (Android) | Kotlin DSL is the default since Flutter 3.41 | Since Flutter **3.29**. |
| A10 | Standard §5.3; flavors guide ("Android Toolchain Baseline"); `guideline.md` §4 checklist | "Set `android.builtInKotlin=true`"; "AGP 9 rejects `kotlinOptions`" | `flutter create` on 3.47.6 writes `android.builtInKotlin=false` and `android.newDsl=false`, and does not apply `kotlin-android`. Flutter says to turn built-in Kotlin on only after all plugins support it. `kotlinOptions` is the legacy DSL (it fails once built-in Kotlin is on). Using `kotlin { compilerOptions { } }` is still correct in both cases. |
| A11 | Standard §17.3, flavors guide (release matrix note) | Tree-shaking needs `const Icon(Icons.x)`; never `Icon(Icons.add)` | What matters is that the `IconData` is a constant (`Icons.add` always is). Only non-constant `IconData(...)` breaks tree-shaking. |
| A12 | Standard §17.2 | Register `assets/images/2.0x/` and `3.0x/` as separate asset entries | Flutter finds the variants by itself when `assets/images/` is listed. Listing the variant folders is not needed. |
| A13 | Standard §13.2 | `db.execute('PRAGMA journal_mode=WAL;')` | This PRAGMA returns a row. sqflite on Android rejects it in `execute`. Use `rawQuery`. Also: sqflite already runs `onCreate` / `onUpgrade` inside a transaction (§13.1 says "wrap in a transaction"). Say not to open a second one there. |
| A14 | Standard §16.6, `release_process.md` §8 | `flutter pub licenses` | No such command (`flutter pub -h`). Replace with a real method: a license-listing package or `dart pub deps` plus a manual check, and the in-app `LicensePage`. |
| A15 | Standard §18.4 | `flutter test --profile`; `driver.traceAction` (flutter_driver) | `flutter test` has no `--profile`. Use `flutter drive --profile` with `integration_test` and `binding.traceAction` / `watchPerformance`. |
| A16 | Standard §18.5 | `@Preview` imported from `package:flutter/widgets.dart` | It is in `package:flutter/widget_previews.dart` (`widgets.dart` does not export it). |
| A17 | Standard §7.8 | `IconSlider`; `NavigationRailDestination` has `tooltip:` | `IconSlider` is not a Flutter widget. `NavigationRailDestination` has no `tooltip` (checked in `material_ui` 1.7.0). Its `label` is the accessible name; wrap the icon in `Tooltip` when labels are hidden. |
| A18 | Standard §5.5 | `ContextMenuRegion` | Not a Flutter widget. Use `MenuAnchor` / `showMenu` with `onSecondaryTapDown`. |
| A19 | Standard §7.2 | Large text = "18 sp or above, or 14 sp bold" | WCAG uses points: 18 pt ≈ **24 sp**, 14 pt bold ≈ **18.7 sp** bold. |
| A20 | Standard §7.3 | `excludeSemantics: true` on decorative images and icons; tooltips add semantics "on long-press" | Use `ExcludeSemantics` or `Image(excludeFromSemantics: true)`. An `Icon` without `semanticLabel` is already silent. Tooltip text is always in the semantics tree, not only on long-press. |
| A21 | Standard §6.5 | Animating `Opacity` is cheap and causes no paint | Animating the `Opacity` widget rebuilds and repaints. Use `FadeTransition` / `AnimatedOpacity`, and `*Transition` widgets for transforms. |
| A22 | Standard §6.8 | Edge-to-edge opt-out possible with a justification | Flutter 3.47 targets API 36. On Android 16 with target 36 the opt-out no longer exists. |
| A23 | Standard §8.8 | Mirror icons via "textDirection in Icon semantics" | Use `Icon(matchTextDirection: true)`; many directional `Icons.*` already set it. |
| A24 | Standard §10.0 | Desktop Skia fallback options "being removed" | They still exist in 3.47 (removal planned later). Add how to opt out on each desktop, as the official post gives it. |
| A25 | `release_process.md` §6.4; `security.md` §10; store §2.4 | `allowBackup=false` stops `adb backup`; use `fullBackupContent` | For target API 31+: `adb backup` no longer copies app data for release apps, and `allowBackup=false` does **not** stop device-to-device transfer. Exclusions go in `android:dataExtractionRules` (API 31+) plus `fullBackupContent` (API 30 and below). |
| A26 | Flavors guide (ProGuard), `release_process.md` §6.2, `security.md` §8.2 | Always add `-keep class io.flutter.** { *; }`; keep rules for sqflite and "JSON serialization annotations" | The Flutter Gradle plugin adds the engine's rules itself. Plugins ship their own consumer rules. JSON code is Dart, not JVM. Add only rules a package documents, and then test the release build. |
| A27 | Standard §15.2; `security.md` §10 | `FlutterWindowManager` | That package is no longer maintained. Set `FLAG_SECURE` in `MainActivity` (or via a method channel / a maintained plugin). |

### B. Missing pieces that make the rules fragile

| # | Where | What to add |
|---|---|---|
| B1 | Standard §6.1, §8.1 | `material_ui` is opt-in upstream in 3.47 (deprecation of the SDK copies is planned). Keep our MUST, but add: use `MaterialUiCompatibilityBridge` while some packages still use the SDK libraries; values of SDK types cannot be passed to `material_ui` types (type mismatch); do **not** use the generated `AppLocalizations.localizationsDelegates` list (it lists the SDK `flutter_localizations` delegates, as checked with gen-l10n on 3.47.6). |
| B2 | Standard §5.5.2, store §5.1, `release_process.md` §11A | Flutter 3.47 warns on Intel Macs and on universal (x86_64 + arm64) macOS builds; `flutter config --enable-macos-arm64-only`; Xcode 27 runs only on Apple silicon. Make "universal binary" a choice with this note. |
| B3 | Standard §5.3 "Upgrading" | Template defaults on 3.47: `compileSdk` / `targetSdk` 36, `minSdk` 24 (as a reference snapshot, not a rule). |
| B4 | `release_process.md` §6.3, §9 | Run size analysis as a **separate** build into its own folder (`--code-size-directory`), *before* the hardened build, or copy the hardened artifacts out first. Never ship the analysis build. |
| B5 | Standard §19.1 | Pin the Flutter version in CI (`flutter-version-file: pubspec.yaml` or an exact version), not only `channel: stable`. Use current major versions of the actions (Dependabot keeps them updated). |
| B6 | `guideline.md` §1.4 | Wrap `{"<lang>": ...}` in backticks in the doc comment (fixes the one analyzer info found). |

### C. Explainers and agent-file templates

Update the matching text in `docs/*_README.md`, `CLAUDE_MD_GUIDELINE.md` and
`AGENTS_MD_GUIDELINE.md` so they say the same as the documents above (mainly A1, A2, A3, A6, A7).
The dated "Reference snapshot — not a rule" boxes get the 2026-10-15 values; no other text gets a
version number.

### D. Remove version lock-in that is already in the guidelines

The current text has about 64 version-tied phrases ("Flutter ≥ 3.19", "since Flutter 3.47",
"(Flutter ≥ 3.12)", "Extra steps when crossing Flutter 3.47", "Stable Since 3.47", …):

| File | Count |
|---|---|
| `flutter_project_engineering_standard.md` | 28 |
| `flutter_build_flavors_guide.md` | 16 |
| `docs/flutter_build_flavors_guide_README.md` | 11 |
| `docs/flutter_project_engineering_standard_README.md` | 8 |
| `security.md` | 1 |

Rewrite each one with the writing rule above. Examples:

- "Extra steps when crossing Flutter 3.47" → "Known migrations — apply each one when its trigger
  applies to your project" (Material/Cupertino packages, AGP 9 / Gradle 9, raised iOS/macOS
  minimums, UIScene).
- "Minimum iOS deployment target: iOS 15 (Flutter 3.47 raised it)" → "Use at least the iOS
  minimum your Flutter version requires (see `ios/Podfile` from `flutter create`, or the build
  error). Raise it when an upgrade raises it."
- "Material 3 is the default since Flutter 3.16" → "Material 3 is the default; do not depend on
  Material 2."

## How the two open choices are handled (generic, no decision needed)

1. **Desktop flavors (A1).** One generic rule: use `--flavor <name>` on any platform where
   `flutter build <platform> -h` lists it and the project has the native setup it needs (Xcode
   schemes on macOS). Otherwise pass `--dart-define=APP_FLAVOR=<name>`. Keep the two-variable
   `AppFlavorConfig`, so the same Dart code works in both cases.
2. **Built-in Kotlin (A10).** Keep whatever `flutter create` / the Flutter migrator writes to
   `android/gradle.properties`. Switching `android.builtInKotlin` on yourself is an optional,
   separate migration, done only after every Android plugin is checked. Always use
   `kotlin { compilerOptions { } }`, which works in both modes.

## Files to change

- `guideline.md`
- `flutter_project_engineering_standard.md`
- `flutter_build_flavors_guide.md`
- `platform_store_readiness.md`
- `release_process.md`
- `security.md`
- `CLAUDE_MD_GUIDELINE.md`
- `AGENTS_MD_GUIDELINE.md`
- `docs/flutter_build_flavors_guide_README.md`
- `docs/flutter_project_engineering_standard_README.md`
- `docs/release_process_README.md`
- `docs/platform_store_readiness_README.md` and `docs/security_README.md` (only if they repeat a
  changed fact)
- New: `change_log/<timestamp>_accuracy-and-robustness-review.md`

Not changed: `architecture.md`, `DOCS_FOLDER_GUIDELINE.md`, `GUIDELINES_MANIFEST.md`,
`README.md`, `AI_AGENT_START_HERE.md`, the language pack and the profile example. No factual
errors were found there; cross-references to section numbers were checked and are correct.

## Plan for the fix

1. Apply the A items in each file, keeping the current structure and section numbers so existing
   cross-references stay valid.
2. Apply the B items.
3. Bring the explainers and the CLAUDE/AGENTS templates in line (C).
3a. Remove the version lock-in (D). Afterwards, grep for `Flutter ≥`, `since Flutter`,
    `Flutter 3.`, `Since 3.`: hits are allowed only inside the "Reference snapshot — not a rule"
    boxes.
4. Re-run the scratch-app checks: compile the changed Dart samples with `flutter analyze`, and
   check every changed command against `flutter <command> -h`.
5. Grep the whole set for the old claims (`98994`, `3.41` Kotlin DSL, `pub licenses`,
   `--analyze-size` next to `--split-debug-info`, `iOS 26 SDK` + UIScene, `architecture: x64, arm64`,
   `IconSlider`, `ContextMenuRegion`) to confirm none remain.
6. Write the change log.

## Out of scope

- Store rules that need a logged-in console to confirm (exact screenshot sizes, Partner Center
  fields). They stay as they are, with the existing "check before each release" warning.
