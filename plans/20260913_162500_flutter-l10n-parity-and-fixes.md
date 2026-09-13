# Plan: Flutter Guidelines Parity with Kotlin — Localization, Play Readiness, and Badge Fixes

**Status:** completed

**Date:** 2026-09-13

## 1. What the issue is

The Flutter guidelines were recently updated for trilingual support (English, Malayalam, Sanskrit), but six gaps remain that were identified and fixed during the Kotlin guidelines work:

1. **Word conflicts in the standard:**
   - In `flutter_project_engineering_standard.md` §8.5.3, "About" is listed as `परिचयः` (Sanskrit) and `വിവരണം` (Malayalam). But §8.5.4 uses `परिचयः` for "Profile" and `വിവരണം` for "Description". §8.5.4 also lists `विषयपरिचयः` and `ആപ്പിനെക്കുറിച്ച്` for "About". This creates confusion.
   - "Exit" is currently listed only as a noun (`निष्क्रमणम्`). But the standard requires action buttons to use a command form (`निष्क्रम्यताम्`).
   - The rule allowing direction words (`प्रत्यागमनम्`, `अग्रिमम्`, `पूर्वम्`, `अधिकम्`) to stay nominal on buttons is missing.
   - The review rule requiring native reader approval for new words is missing.

2. **Play Store download breaks in-app language switching:**
   - Google Play splits Android App Bundles (.aab) by language by default. When an English user downloads the app, Play only installs English resources. If the user picks Malayalam or Sanskrit inside the app, the strings are missing.
   - The fix requires disabling language splitting in the Android Gradle build: `bundle { language { enableSplit = false } }`. This setting is not documented in the Flutter build guide, engineering standard, or release process.

3. **Heart badge emoji rendering:**
   - In `guideline.md` §1.7, the `MadeWithLove` widget draws the heart using the text character `_heart = '❤'`.
   - On many Android phones, the system emoji font overrides this character with a colorful emoji glyph, ignoring the red color. Using `WidgetSpan` with `Icon(Icons.favorite, color: Color(0xFFE53935))` guarantees an exact vector red heart on all devices.

4. **Character counting rules:**
   - Section 8.6 states a 22-character limit for short UI text, but notes that `प्रयुक्ता कृत्रिमबुद्धिः` is 24 characters because it counted UTF-16 code units.
   - In Malayalam and Sanskrit, a visible letter (grapheme cluster) includes attached vowel signs and viramas. Using Flutter's `characters.length` from `package:characters`, every glossary term easily fits within 22 characters.

5. **Asset and help file translation checks:**
   - The Sanskrit CI gate checks only `lib/l10n/app_sa.arb`. It does not check `assets/config/app_config.json` (where localized About content lives) or help files (`assets/help/*_sa.md`) for Hindi markers.
   - Feature parity rules must also ensure non-ARB assets exist in all three languages.

6. **About row label exemption:**
   - Section 8.6 does not explicitly state that `aboutDetail*` ARB keys for About screen detail rows are exempt from the 22-character limit so they can wrap to two lines cleanly.

---

## 2. Files to be changed

| File | Change |
|---|---|
| `guideline.md` | §1.7: Update `MadeWithLove` widget to use `WidgetSpan` with `Icon(Icons.favorite)` instead of a text emoji character. |
| `flutter_project_engineering_standard.md` | §8.1: Require `bundle.language.enableSplit = false` in `android/app/build.gradle.kts`. <br>§8.5.1: Add direction words button rule. Update CI gate pattern to scan ARB, JSON config, and markdown assets. <br>§8.5.2: Update Malayalam About rule. <br>§8.5.3: Fix "About" row in Bad → Good table to `विषयपरिचयः` and `ആപ്പിനെക്കുറിച്ച്`. <br>§8.5.4: Split "Exit" into button (`പുറത്തുകടക്കുക` / `निष्क्रम्यताम्`) and label (`പുറത്തുകടക്കൽ` / `निष्क्रमणम्`). Add review rule. <br>§8.6: Update character counting to use `package:characters` (grapheme clusters) and add `aboutDetail*` budget exemption. <br>§8.7: Extend parity checks to JSON and asset files. |
| `flutter_build_flavors_guide.md` | Add `bundle { language { enableSplit = false } }` to the Android Gradle configuration example and explain why it is needed. |
| `release_process.md` | Add language split check to the Google Play Store readiness checklist (§8 and §9A). |
| `docs/flutter_project_engineering_standard_README.md` | Update plain-English explainer for character counting, Play language split, and Sanskrit/Malayalam glossary parity. |

---

## 3. Plan for the fix

### 3.1 Badge Widget Fix (`guideline.md`)
- Update `MadeWithLove` reference code in §1.7:
  - Replace `TextSpan(text: _heart, ...)` with:
    ```dart
    WidgetSpan(
      alignment: PlaceholderAlignment.middle,
      child: Icon(
        Icons.favorite,
        size: (base.fontSize ?? 12) * 1.1,
        color: _heartColor,
      ),
    )
    ```
  - Explain why: `WidgetSpan` guarantees the heart icon is drawn with the exact vector asset and exact `#E53935` red color on every device, without being altered by device emoji fonts.

### 3.2 Terminology & Glossary Parity (`flutter_project_engineering_standard.md`)
- In §8.5.1, add the rule: Direction words (`प्रत्यागमनम्` Back, `अग्रिमम्` Next, `पूर्वम्` Previous, `अधिकम्` More) stay in nominal/adverbial form on buttons, like Yes/No. Close (`पिधीयताम्`) and Exit (`निष्क्रम्यताम्`) on buttons use the polite imperative verb form.
- In §8.5.2, clarify that `ആപ്പിനെക്കുറിച്ച്` is the About title; `വിവരണം` is reserved for Description.
- In §8.5.3, update the "About" row:
  - Good (Sanskrit): `विषयपरिचयः`
  - Good (Malayalam): `ആപ്പിനെക്കുറിച്ച്`
  - Rationale: Explain that `परिचयः` (Sanskrit) is Profile and `വിവരണം` (Malayalam) is Description.
- In §8.5.4, split Exit:
  - `Exit (button)`: `പുറത്തുകടക്കുക` | `निष्क्रम्यताम्`
  - `Exit (title, label)`: `പുറത്തുകടക്കൽ` | `निष्क्रमणम्`
- Add the review rule in §8.5.4: Any new or modified Malayalam or Sanskrit terms must be reviewed by a fluent reader before release.

### 3.3 Character Counting & Label Exemption (`flutter_project_engineering_standard.md`)
- In §8.6, explain that characters are visible grapheme clusters, counted using `string.characters.length` from `package:characters`.
- Update the note in §8.5.4 and §8.6: Remove the old statement that `प्रयुक्ता कृत्रिमबुद्धिः` is 24 characters. Clarify that `aboutDetail*` keys are exempt from the 22-character budget so they can wrap to two lines.

### 3.4 Sanskrit CI Gate & Asset Parity (`flutter_project_engineering_standard.md`)
- Update the Sanskrit check script to scan `lib/l10n/app_sa.arb`, `assets/config/app_config.json`, and all `assets/**/*_sa.*` files.
- In §8.7, require that translation completeness tests check JSON config files and localized help assets in addition to ARB files.

### 3.5 Play Store Language Splitting (`flutter_build_flavors_guide.md`, `release_process.md`, `flutter_project_engineering_standard.md`)
- In `flutter_build_flavors_guide.md`, add `bundle { language { enableSplit = false } }` inside `android { ... }` in the Gradle reference code.
- Explain the reason: Google Play splits App Bundles by language by default. Disabling language splitting ensures users can change languages inside the app at any time without missing strings.
- In `release_process.md`, add an explicit verification checkbox under Google Play Store readiness.

### 3.6 Documentation Explainers
- Update `docs/flutter_project_engineering_standard_README.md` to reflect these updates in simple English.

---

## 4. Verification Plan

1. Verify all markdown cross-references and internal links.
2. Ensure all text conforms to simple English rules.
3. Check all Sanskrit and Malayalam terms against the verified glossary from the Kotlin guidelines.
4. Verify that only relative paths are used and no machine-specific or sensitive details are included.
