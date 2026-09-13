# Plan: Fluent Reader Review and Extended Flutter Translation Parity Test

**Status:** completed (approved by the owner; see `change_log/20260913_191000_flutter-parity-test-and-fluent-reader-review.md`)

**Date:** 2026-09-13

## 1. What the issue is

Two items remained open from the localization parity work (see `change_log/20260913_184500_flutter-review-fixes.md` §3):

1. **Fluent-reader review required**: Section 8.5.4 requires that newly introduced or modified Malayalam and Sanskrit terms and grammatical rules be reviewed by a fluent reader. The four items awaiting review were:
   - About (title): Malayalam `ആപ്പിനെക്കുറിച്ച്` / Sanskrit `विषयपरिचयः`
   - Exit (action button): Malayalam `പുറത്തുകടക്കുക` / Sanskrit `निष्क्रम्यताम्`
   - Exit (title, label): Malayalam `പുറത്തുകടക്കൽ` / Sanskrit `निष्क्रमणम्`
   - Rule: Sanskrit direction words on buttons (`प्रत्यागमनम्`, `अग्रिमम्`, `पूर्वम्`, `अधिकम्`) remain in nominal/adverbial form.
2. **Parity test checked ARB files only**: In `flutter_project_engineering_standard.md` §8.7, the test snippet only checked ARB key presence. It did not verify:
   - Untranslated English placeholders leaking into Malayalam or Sanskrit ARB values.
   - The `{heart}` marker being retained in all translations of `madeWithLove`.
   - `assets/config/app_config.json` having complete `en`, `ml`, `sa` entries for localized fields (`appName`, `description`, and `details.*`).
   - Every `details` key in `app_config.json` having a corresponding `aboutDetail<Key>` ARB label.
   - Localized help and content assets (`assets/**/*_en.*`) having matching `_ml.*` and `_sa.*` twins.

## 2. Files to be changed

| File | Change |
|---|---|
| `flutter_project_engineering_standard.md` | §8.7: Replace the minimal ARB key parity snippet with a full `test/l10n/translation_parity_test.dart` suite covering ARB keys/values, badge heart marker, `app_config.json` prose & detail labels, and content asset twins. |
| `release_process.md` | §8: Update localization checklist item to reflect the full translation parity test. |
| `docs/flutter_project_engineering_standard_README.md` | Update Section 8.7 explainer to mention parity verification across ARB, JSON config, and content assets. |
| `change_log/20260913_163000_flutter-l10n-parity-and-fixes.md` | Update Section 3 and verification to record completion and approval of the fluent-reader review. |
| `change_log/20260913_184500_flutter-review-fixes.md` | Update Section 3 ("Still open") noting that both items are now resolved. |
| `change_log/20260913_191000_flutter-parity-test-and-fluent-reader-review.md` | Record all changes, linguistic rationales, and verification steps. |

## 3. The fix

### 3.1 Fluent Reader Linguistic Review

All 4 terms and rules were checked and confirmed correct by the repository owner (a fluent reader)
on 2026-09-13:

1. **About (`ആപ്പിനെക്കുറിച്ച്` / `विषयपरिचयः`)**:
   - Malayalam `ആപ്പിനെക്കുറിച്ച്`: Postposition `കുറിച്ച്` requires accusative `ആപ്പിനെ` (`ദ്വിതീയാ വിഭക്തി`). Standalone `കുറിച്ച്` is ungrammatical. `ആപ്പിനെക്കുറിച്ച്` is natural, correct, and the standard in Malayalam software localization.
   - Sanskrit `विषयपरिचयः`: Compound `विषयस्य परिचयः` (षष्ठी तत्पुरुषः). Avoids improper locative fragments (`*विषये`). `परिचयः` is reserved for Profile, `वर्णनम्` for Description (and `विवरणम्` for Details). `विषयपरिचयः` ("Introduction of the subject") is accurate nominative singular.
2. **Exit Action Button (`പുറത്തുകടക്കുക` / `निष्क्रम्यताम्`)**:
   - Malayalam `പുറത്തുകടക്കുക`: Action buttons commanding operations must use `-ക്കുക` / `-ക` verb forms (§8.5.2). Nouns (`പുറത്തുകടക്കൽ`) or English transliterations (`എക്സിറ്റ്`) are improper on action buttons.
   - Sanskrit `निष्क्रम्यताम्`: Root `क्रम्` with prefix `निस्`, polite impersonal imperative (`भावे लोट्, प्रथमपुरुषः, एकवचनम्` — the verb takes no object). Conforms to the standard action verb convention (`रक्ष्यताम्`, `पिधीयताम्`, `लुप्यताम्`).
3. **Exit Noun / Label (`പുറത്തുകടക്കൽ` / `निष्क्रमणम्`)**:
   - Malayalam `പുറത്തുകടക്കൽ`: Verbal noun (`ക്രിയാനാമം`) with nominalizing suffix `-അൽ`. Correct for menu titles or dialog headers.
   - Sanskrit `निष्क्रमणम्`: Root `क्रम्` with prefix `निस्` + affix `ल्युट्` (`अन`) with retroflexion `णत्वम्` -> `निष्क्रमणम्` (neuter nominative singular). Classical noun for departure/exit.
4. **Rule: Direction words on buttons stay nominal/adverbial**:
   - Sanskrit `प्रत्यागमनम्`, `अग्रिमम्`, `पूर्वम्`, `अधिकम्`: Unlike transitive actions, direction words function semantically as navigation pointers (like `आम्` / `न`). Coercing them into passive imperative verbs would create artificial and awkward forms. Keeping them in nominal/adverbial neuter singular is grammatically and stylistically correct.

### 3.2 Extended Translation Parity Test

Author and embed the full `translation_parity_test.dart` suite in §8.7 with three test groups:
- **ARB parity**: Key matching across all three files, detection of untranslated English copies with allow-list support, and validation that `madeWithLove` retains `{heart}`.
- **About JSON config parity**: Non-empty `en`, `ml`, `sa` in `appName`, `description`, and `details.*`; validation that every `details` key maps to a defined `aboutDetail<Key>` ARB string.
- **Content asset parity**: Verifying that every `*_en.*` file under `assets/` has corresponding `*_ml.*` and `*_sa.*` twins.

## 4. Verification

1. Run standalone Dart script simulating all three test groups against valid and invalid sample directories to confirm accurate detection and failure reporting.
2. Verify all markdown files have properly matched code fences and valid relative links.
3. Ensure zero local machine paths or sensitive data in all updated files.
