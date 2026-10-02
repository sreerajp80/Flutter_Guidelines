# Change log: Guidelines say "latest stable" instead of pinning exact tool versions

Implements: `plans/20261002_223442_latest-stable-not-pinned-versions.md`

## What changed

The guidelines no longer make exact tool versions (AGP 9.1.0, KGP 2.4.0, Gradle 9.3.1,
"Flutter 3.47+") into rules. The rules now say:

- use the latest stable Flutter;
- use the AGP / KGP / Gradle versions that Flutter generates (do not hand-pick or downgrade);
- each project pins its own versions in its own files;
- look up versions (`flutter --version`, `settings.gradle.kts`, Gradle wrapper) — never
  write one from memory.

Version numbers are kept only where a rule depends on them ("on AGP 9+", "since Flutter
3.47") and for hard minimums (Java 17, iOS 15, macOS 12). Exact numbers live in a clearly
marked "reference snapshot — not a rule" (last checked 2026-10).

## Files

| File | Change |
|------|--------|
| `flutter_build_flavors_guide.md` | Banner → "Last checked against…". Toolchain Prerequisites: new rules list, minimums table (Java 17, iOS 15, macOS 12, Xcode, target SDK), separate "reference snapshot" table. "AGP 9 / Gradle 9 Baseline" → "Android Toolchain Baseline (AGP 9+ / Gradle 9+)" with code samples marked as EXAMPLE versions. R8 note and iOS heading reworded as conditions. New-project checklist drops exact numbers. |
| `flutter_project_engineering_standard.md` | §5.3 "Toolchain Requirements" rewritten with the rules above plus a snapshot note; AGP 9 / Gradle 9 rules worded as "on AGP 9+ / Gradle 9+". "Upgrading To Flutter 3.47" → generic "Upgrading To A New Flutter Release" with an "extra steps when crossing 3.47" list. `flutter_lints` → "current major (`^6.0.0` at the time of writing)". Cross-reference renamed to "Android Toolchain Baseline". |
| `docs/flutter_build_flavors_guide_README.md` | Banner, summary, "Toolchain pinning" Must row, AI rules, AI instruction block and review questions — no exact AGP/KGP/Gradle numbers; "look up, don't guess" rule added. |
| `docs/flutter_project_engineering_standard_README.md` | Banner and §5–9 summary updated the same way. |
| `CLAUDE_MD_GUIDELINE.md`, `AGENTS_MD_GUIDELINE.md` | Flutter SDK and Dart SDK rows → `<project's pinned version — from flutter --version>` (the Dart row had a stale `<3.11.x or higher>`, so it was changed too for consistency). |
| `guideline.md` | §4 toolchain checklist item reworded to "latest stable + versions Flutter generates, pinned in the project's own files". |

## Not changed

"Since Flutter 3.xx" notes for features (Material packages, Impeller desktop, Widget Previews,
per-platform assets, UIScene) — these are conditions, not pins.

## Check

Search for `9.1.0`, `2.4.0`, `9.3.1`, `3.47+`, `Lock to`, `Gradle 9 Baseline`, `<3.`: matches
remain only in the snapshot tables and the marked example code blocks.
