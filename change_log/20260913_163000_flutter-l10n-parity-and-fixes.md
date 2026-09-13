# Change Log: Flutter Guidelines Parity with Kotlin — Localization, Play Readiness, and Badge Fixes

**Date:** 2026-09-13

**Plan:** [../plans/20260913_162500_flutter-l10n-parity-and-fixes.md](../plans/20260913_162500_flutter-l10n-parity-and-fixes.md)

> **Corrected on 2026-09-13** by
> [20260913_184500_flutter-review-fixes.md](20260913_184500_flutter-review-fixes.md): the Profile
> term in item 1 was wrong, and the verification line wrongly said the terms were already reviewed.

## 1. Summary

The Flutter guidelines have been updated to achieve full parity with the Kotlin guidelines across six areas:

1. **Terminology conflicts resolved**: "About" is now consistently defined as `ആപ്പിനെക്കുറിച്ച്` (Malayalam) and `विषयपरिचयः` (Sanskrit) across all tables, eliminating collisions with "Profile" (`പ്രൊഫൈൽ` / `परिचयः`) and "Description" (`വിവരണം`). "Exit" is split into button action (`പുറത്തുകടക്കുക` / `निष्क्रम्यताम्`) and noun label (`പുറത്തുകടക്കൽ` / `निष्क्रमणम्`). Direction words on buttons (`प्रत्यागमनम्`, `अग्रिमम्`, `पूर्वम्`, `अधिकम्`) stay nominal. A mandatory rule was added requiring native-reader review for any new or modified glossary terms.
2. **Play Store language splitting**: Added requirement and Gradle configuration (`bundle.language.enableSplit = false`) across the engineering standard, build flavors guide, and release process to prevent missing strings on Play Store installs.
3. **Heart badge rendering**: `MadeWithLove` widget in `guideline.md` now uses `WidgetSpan` with `Icon(Icons.favorite, color: Color(0xFFE53935))` instead of a text emoji character, ensuring an exact vector red heart on all OEM devices without system emoji font overrides.
4. **Visible character counting**: Engineering standard §8.6 clarifies that characters are visible grapheme clusters counted using Dart's `characters.length` (`package:characters`).
5. **Asset and help file translation checks**: Sanskrit CI gate expanded to scan `app_sa.arb`, `assets/config/app_config.json`, and all `assets/**/*_sa.*` files. Parity tests required for JSON and markdown help assets.
6. **About row labels exemption**: Documented the 22-character short-text budget exemption for `aboutDetail*` row labels so they can wrap to two lines in About screens.

## 2. Files changed

| File | What changed |
|---|---|
| `guideline.md` | §1.7 `MadeWithLove` widget updated to use `WidgetSpan` with `Icon(Icons.favorite)` and explanatory note. Added `bundle.language.enableSplit = false` check to §4 checklist. |
| `flutter_project_engineering_standard.md` | §8.1 added `bundle.language.enableSplit = false` requirement and explanation. §8.5.1 added direction words button form rule and updated CI gate script to scan ARB, JSON config, and markdown assets with self-test. §8.5.2 updated Malayalam About postposition rule. §8.5.3 corrected "About" row in Bad → Good table. §8.5.4 added native-reader review rule and split "Exit" into button and label forms. §8.6 updated character counting explanation to `package:characters` and added `aboutDetail*` exemption. §8.7 added JSON config and asset parity requirements. |
| `flutter_build_flavors_guide.md` | Added `bundle { language { enableSplit = false } }` to `android { ... }` block with explanatory note on Play Store App Bundles. |
| `release_process.md` | Added language splitting checks to Pre-Release Checklist (§8) and Google Play Readiness Gate (§9A.3). |
| `docs/flutter_project_engineering_standard_README.md` | Updated plain-English explainer items for Section 8.1, 8.5, and 8.6. |
| `plans/20260913_162500_flutter-l10n-parity-and-fixes.md` | Marked plan as completed. |

## 3. Needs native-reader review

> **Reviewed and confirmed on 2026-09-13** by the repository owner (a fluent reader); recorded in
> [20260913_191000_flutter-parity-test-and-fluent-reader-review.md](20260913_191000_flutter-parity-test-and-fluent-reader-review.md).

These Malayalam and Sanskrit terms and rules were added or changed. Under the review rule in §8.5.4,
a fluent reader must confirm them before apps use them. They are the same terms as in the Kotlin
guidelines, where the same owner review is recorded.

| Meaning | Malayalam | Sanskrit | Change |
|---|---|---|---|
| About (title, everywhere) | ആപ്പിനെക്കുറിച്ച് | विषयपरिचयः | Bad → Good table now matches the About glossary |
| Exit (button) | പുറത്തുകടക്കുക | निष्क्रम्यताम् | New Sanskrit imperative form |
| Exit (title, label) | പുറത്തുകടക്കൽ | निष्क्रमणम् | New Malayalam noun form |
| Rule: direction words on buttons | — | `प्रत्यागमनम्`, `अग्रिमम्`, `पूर्वम्`, `अधिकम्` stay nominal | New form rule |

## 4. Verification

- Sanskrit and Malayalam terms match the Kotlin guidelines word for word. **Reviewed and approved
  by a fluent reader** (see section 3 and `change_log/20260913_191000_flutter-parity-test-and-fluent-reader-review.md`).
- Markdown table structures, code blocks, and cross-references verified clean and properly closed.
  (Correction: a note had been placed inside the Gradle code block in
  `flutter_build_flavors_guide.md`; fixed in the follow-up change log.)
- Only relative paths used across all plans and change logs.
- Zero local machine paths, usernames, IP addresses, credentials, or sensitive data present.
