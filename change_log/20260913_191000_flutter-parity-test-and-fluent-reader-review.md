# Change Log: Fluent Reader Review and Extended Flutter Translation Parity Test

**Date:** 2026-09-13

**Plan:** [../plans/20260913_190000_flutter-parity-test-and-fluent-reader-review.md](../plans/20260913_190000_flutter-parity-test-and-fluent-reader-review.md)

## 1. What changed

| File | Change |
|---|---|
| `flutter_project_engineering_standard.md` | §8.7: Replaced the minimal ARB key parity test snippet with a full `test/l10n/translation_parity_test.dart` suite covering ARB key parity, untranslated English copy detection, heart badge `{heart}` marker retention, `assets/config/app_config.json` multilingual prose & detail label keys, and localized content asset twins (`assets/**/*_en.*`). |
| `release_process.md` | §8: Updated localization checklist item to reflect the full translation parity test covering ARB files, `app_config.json`, and help assets. |
| `docs/flutter_project_engineering_standard_README.md` | §8.7 explainer updated to note translation parity test coverage across ARB files, About JSON, and content assets. |
| `change_log/20260913_163000_flutter-l10n-parity-and-fixes.md` | §3 and §4 updated to record that the fluent reader review is completed and all four items are approved. |
| `change_log/20260913_184500_flutter-review-fixes.md` | §3 ("Still open") updated to note resolution of both open items. |

## 2. Fluent-Reader Review Details & Approval

Under §8.5.4, the 4 Malayalam and Sanskrit terms and rules from the parity change were checked and confirmed correct by the repository owner (a fluent reader) on 2026-09-13. The notes below record the reasoning for each item:

| Meaning | Malayalam | Sanskrit | Linguistic Analysis & Approval |
|---|---|---|---|
| About (title, everywhere) | `ആപ്പിനെക്കുറിച്ച്` | `विषयपरिचयः` | **Approved.** In Malayalam, postpositions like `കുറിച്ച്` require an accusative noun base (`ആപ്പിനെ`, ദ്വിതീയാ വിഭക്തി); a bare `കുറിച്ച്` is ungrammatical. `ആപ്പിനെക്കുറിച്ച്` is natural, standard Malayalam software localization. In Sanskrit, `विषयपरिचयः` (षष्ठी तत्पुरुषः) provides an authentic nominative singular title ("Introduction/Overview of the subject"), avoiding clumsy locative fragments (`*विषये`) or collisions with `परिचयः` (Profile), `वर्णनम्` (Description) and `विवरणम्` (Details). |
| Exit (button action) | `പുറത്തുകടക്കുക` | `निष्क्रम्यताम्` | **Approved.** Action buttons commanding an operation must use `-ക്കുക` / `-ക` verb forms in Malayalam (§8.5.2); `പുറത്തുകടക്കുക` is standard and natural. In Sanskrit, root `क्रम्` with prefix `निस्` in polite impersonal imperative (`भावे लोट्, प्रथमपुरुषः, एकवचनम्` — the verb takes no object) `निष्क्रम्यताम्` strictly conforms to the action verb standard (`रक्ष्यताम्`, `पिधीयताम्`, `लुप्यताम्`). |
| Exit (title, label, noun) | `പുറത്തുകടക്കൽ` | `निष्क्रमणम्` | **Approved.** Malayalam verbal noun (`ക്രിയാനാമം`) with `-അൽ` nominalizer is appropriate for menu titles and dialog headers. Sanskrit `निस् + क्रम् + ल्युट्` (`अन`) with retroflexion `णत्वम्` -> `निष्क्रमणम्` (neuter nominative singular) is the classical noun for exit/departure. |
| Rule: direction words on buttons | — | `प्रत्यागमनम्`, `अग्रिमम्`, `पूर्वम्`, `अधिकम्` stay nominal | **Approved.** Direction words on buttons function semantically as navigation pointers (like `आम्` / `न`) rather than transitive operations on data objects. Forcing them into passive imperative verbs would create artificial, unnatural coinages. Keeping them in nominal/adverbial neuter singular form is grammatically sound and idiomatic. |

## 3. Extended Translation Parity Test Implementation

The upgraded `translation_parity_test.dart` added in §8.7 provides three distinct test groups:

1. **ARB Parity Tests**:
   - `every ARB file has the same keys as the template`: Asserts no missing and no extraneous keys across `app_ml.arb` and `app_sa.arb` relative to `app_en.arb`.
   - `no translation is a copy of the English value`: Compares localized values against English and flags untranslated placeholders, while providing a configurable `sameAsEnglishAllowed` set for brand names or symbols.
   - `aboutMadeWithLove keeps the {heart} marker in all three languages`: Asserts that `madeWithLove` retains the `{heart}` marker so vector heart rendering remains intact.
2. **About JSON Config Parity Tests**:
   - `app_config.json has all three languages and valid detail labels`:
     - Checks that localized prose objects (`appName`, `description`, and `details.*`) contain non-empty strings for `en`, `ml`, and `sa`.
     - Validates that every key in `details` (e.g. `author`, `license`) has a matching ARB label key (`aboutDetailAuthor`, `aboutDetailLicense`) in the template ARB.
3. **Content Asset Parity Tests**:
   - `content and help assets exist in all three languages`: Recursively scans `assets/` for any `*_en.*` file (e.g. `help_en.md`) and asserts that matching `*_ml.*` and `*_sa.*` twin files exist on disk.

## 4. Verification

- Test logic validated against a simulation runner in Dart 3.12.2 covering all test groups and assertions.
- Code blocks in markdown files verified clean with matching opening and closing fences.
- All internal markdown references use relative paths only.
- Zero local machine paths, usernames, IP addresses, credentials, or sensitive data present.
