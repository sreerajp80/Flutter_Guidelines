# Plan: Section 8.5 Quality Assurance (Sanskrit & Malayalam) and Glossary Refinement

**Status:** completed

**Date:** 2026-09-13

## 1. What the issue is

Section 8.5 of `flutter_project_engineering_standard.md` previously focused primarily on Sanskrit vs. Hindi in Devanagari, but contained several grammatical inaccuracies in Sanskrit, missed explicit quality rules for Malayalam, and exhibited problematic translations in both languages across the shared UI glossary:

1. **Sanskrit grammatical inconsistencies**:
   - `न किमपि दत्तांशं प्राप्तम्` had incorrect gender agreement (`दत्तांशः` is masculine nominative; participle must be `प्राप्तः`, and indefinite pronoun must be `कोऽपि`).
   - `प्रतिलिप्यताम्` and `प्रतिलिपितम्` were pseudo-verbs coined from the noun `प्रतिलिपि`; proper Sanskrit uses the verb root `लिख्` with prefix `प्रति` (`प्रतिलिख्यताम्` / `प्रतिलिखितम्`).
   - `निर्यात्यताम्` was ungrammatical; the causative passive of `या` is `निर्याप्यताम्`.
   - `संपुष्यताम्` means "let it be nourished"; "Confirm" is `स्थिरीक्रियताम्` or `दृढीक्रियताम्`.
   - `लेखा` strictly means a line/streak in Sanskrit (borrowed from Hindi for account); `खाता` or `उपयोक्तृविवरणम्` is appropriate.
   - `ध्वनिः` should be used for audio/sound to avoid confusion with `शब्दः` (Word).
2. **Missing Malayalam quality principles**:
   - Malayalam translations in the glossary frequently fell back to lazy English phonetic transliterations (`സേവ്`, `പ്രിന്റ്`, `വൈബ്രേഷൿ`, `ലോഗൗട്ട്`, `ഓപ്ഷണൽ`, `നമ്ബൿ`).
   - Action buttons must use verbs ending in `-ക്കുക` / `-ക` (`സൂക്ഷിക്കുക`, `അച്ചടിക്കുക`, `പുറത്തുകടക്കുക`).
   - Dialog confirmation requires distinguishing `ഇല്ല` (negative response/non-existence) from `അല്ല` (negation of identity).
   - `കുറിച്ച്` is a bound postposition that cannot stand alone as an About screen title; `വിവരണം` or `ആപ്പിനെക്കുറിച്ച്` is required.
   - False friends need correction: `താൽപ്പര്യങ്ങർ` / `ഇഷ്ടങ്ങർ` for Preferences (`മുൿഗണനകർ` means Priorities); `പ്രയോഗിക്കുക` for Apply (`ബാധകമാക്കുക` is legalistic).
3. **Tri-lingual Glossary alignment**:
   - All 8 subsections of the standard UI glossary must be updated with verified, natural, and length-budget compliant terms for both Malayalam and Sanskrit.

## 2. Files to be changed

| File | Change |
|---|---|
| `flutter_project_engineering_standard.md` | Update Section 8 TOC line 928, rewrite §8.5 with Malayalam & Sanskrit quality guidelines, enhanced Bad → Good table, and audited UI glossary tables |
| `docs/flutter_project_engineering_standard_README.md` | Update §8.5 summary to reflect both Sanskrit and Malayalam quality and glossary standards |

## 3. Plan for the fix

1. Update TOC at line 928 of `flutter_project_engineering_standard.md`.
2. Rewrite §8.5 to:
   - Establish dual-language quality mandates: Sanskrit Quality (§8.5.1) and Malayalam Quality (§8.5.2).
   - Keep and refine forbidden Hindi markers and CI check script.
   - Expand Bad → Good table covering both languages with explanations.
   - Re-audit and clean all 8 glossary tables.
3. Verify formatting, check character limits against §8.6, and ensure zero Hindi markers in Sanskrit ARB terms.
4. Record change log in `change_log/`.
