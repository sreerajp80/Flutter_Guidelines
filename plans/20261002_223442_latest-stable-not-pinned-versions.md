# Plan: Make the guidelines say "latest stable" instead of pinning exact tool versions

**Status:** completed

## The issue

These are generic guidelines for all Flutter projects. The last change
(`plans/20261002_221017_flutter-3-47-toolchain-update.md`) put exact version numbers into
MUST rules, for example "Lock to Flutter 3.47+, AGP 9.1.0, KGP 2.4.0, Gradle 9.3.1".

Problems:

1. The rules go stale with every Flutter release. The old text ("AGP 8.x, NOT 9.x") became
   harmful advice as soon as Flutter moved on.
2. Picking exact versions is the job of each project (its Gradle wrapper, `pubspec.lock`,
   `settings.gradle.kts`), not of a shared guideline.

But a plain "use latest" is not enough:

- AI agents do not know what "latest" is. They use versions from their training memory
  (this is how AGP 8 and `kotlinOptions { }` come back).
- Some rules only apply from a certain version on (`kotlinOptions` is rejected on AGP 9+;
  `material_ui` exists from Flutter 3.47). Those rules need the version to stay correct.
- Some numbers are hard minimums (Java 17, iOS 15, macOS 12), not preferences.

## The approach

1. **Rules say "latest stable".** Projects use the latest stable Flutter, and the AGP / KGP /
   Gradle versions that `flutter create` (or the Flutter migrator) writes for it. Do not
   hand-pick or hand-downgrade them. Each project pins its own versions in its own files.
2. **Version numbers stay only where a rule depends on them**, written as a condition:
   "on AGP 9+", "since Flutter 3.47", "Java 17 minimum", "iOS 15 minimum (since 3.47)".
3. **Exact numbers move to one "last checked" reference table** per guide, clearly marked as
   a snapshot (not a rule), with the date it was checked.
4. **Agents must look up current versions, not guess.** New rule: read `flutter --version`,
   `android/settings.gradle.kts`, `android/gradle/wrapper/gradle-wrapper.properties` and
   `pubspec.lock`; never pick a version from memory; to update, run
   `flutter upgrade` and let the migrator change the Android files.
5. **Templates** (CLAUDE.md / AGENTS.md): the Flutter SDK row becomes a project-filled
   placeholder (`<the project's pinned Flutter version, e.g. from flutter --version>`).

## Files to be changed

| File | Change |
|------|--------|
| `flutter_build_flavors_guide.md` | Toolchain table (lines ~20–30) → rename to "Toolchain reference (last checked 2026-10, Flutter 3.47)", add a line above it: "Use latest stable; this table is a snapshot, not a rule". "AGP 9 / Gradle 9 Baseline" (~116–162): rule becomes "use the AGP / KGP / Gradle versions Flutter generates; on AGP 9+ use `compilerOptions`; on Gradle 9+ use `ExecOperations`"; the `settings.gradle.kts` / wrapper samples keep numbers but are marked "example — use the versions your Flutter generates". "Notes For New Projects" (~982–1003): drop exact AGP/KGP/Gradle numbers, keep Java 17 / iOS 15 minimums. Add the "look up, don't guess" rule. |
| `flutter_project_engineering_standard.md` | §5.3 "Toolchain Requirements (Flutter ≥ 3.47)" → "Toolchain Requirements": latest stable rule, Java 17 minimum, conditional AGP 9+/Gradle 9+ rules, "look up, don't guess" rule; exact numbers only as "at the time of writing". "Upgrading An Existing App To Flutter 3.47" → "Upgrading An Existing App To A New Flutter Release" (generic steps), with the 3.47-specific steps kept as a short sub-list "if crossing 3.47". Version-gated feature notes (Material packages, Impeller desktop, Widget Previews, per-platform assets, iOS 15) stay as "since 3.xx" — they are conditions, not pins. `flutter_lints: ^6.0.0` → "pin the current major (`^6.0.0` at the time of writing)". |
| `docs/flutter_build_flavors_guide_README.md` | "Must — Toolchain pinning" row → "Toolchain: latest stable Flutter + the Android versions it generates; each project pins its own in the wrapper / settings files". AI-rules block and "Know the … toolchain" line: drop exact AGP/KGP/Gradle numbers, add "look up versions in the project files, never from memory". Review question (line ~161): "Does any Android file use versions older than what the current Flutter generates, or `kotlinOptions` on AGP 9+?". Summary line 11: drop exact numbers. |
| `docs/flutter_project_engineering_standard_README.md` | Section summary (line 14): drop exact AGP/KGP/Gradle numbers; "upgrade checklist for a new Flutter release". Banner keeps "last checked against Flutter 3.47 / Dart 3.13". |
| `CLAUDE_MD_GUIDELINE.md`, `AGENTS_MD_GUIDELINE.md` | `<3.47.x or higher>` → `<project's pinned Flutter version — see flutter --version>`. Keep the `material_ui` import line (it is a condition of current Flutter). |
| `guideline.md` | §4 checklist line 690 → "Toolchain is the latest stable Flutter with the Android versions it generates (Java 17 minimum); versions are pinned in the project's own files, not copied from this guide". |

Not changed: the banner lines "Reflects Flutter 3.47 / Dart 3.13" — they are reworded to
"Last checked against Flutter 3.47 / Dart 3.13 (2026-10)" so they read as a date stamp, not a
requirement. `security.md` and other files with only "since 3.xx" history notes stay as they are.

## Steps

1. Make the edits above, keeping the existing MUST/SHOULD style and simple English.
2. Search again for `9.1.0`, `2.4.0`, `9.3.1`, `3.47+`, `Lock to` to confirm exact numbers
   remain only in the reference tables, the marked examples, and "since" notes.
3. Write a change log in `change_log/`.
