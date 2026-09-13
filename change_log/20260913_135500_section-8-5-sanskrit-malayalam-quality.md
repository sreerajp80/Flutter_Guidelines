# Change Log: Section 8.5 Quality Assurance (Sanskrit & Malayalam) and Glossary Refinement

**Plan Reference:** [../plans/20260913_134700_section-8-5-sanskrit-malayalam-quality.md](../plans/20260913_134700_section-8-5-sanskrit-malayalam-quality.md)

**Date:** 2026-09-13

## Summary

Section 8.5 of `flutter_project_engineering_standard.md` has been upgraded, audited, and debugged to ensure linguistic correctness, natural phrasing, and structural integrity:

1. **Section 8.5 Structure & Scope**:
   - Expanded title to **Sanskrit & Malayalam Quality — Standard UI Glossary** and updated Table of Contents line 928.
   - Divided into dedicated subsections for Sanskrit Quality (§8.5.1), Malayalam Quality (§8.5.2), Bad → Good Translations with linguistic rationale (§8.5.3), and the tri-lingual Standard UI Glossary (§8.5.4).
   - Removed broken unclosed code block at lines 1271–1272 that broke markdown rendering.

2. **Sanskrit Quality Corrections**:
   - Fixed subject-participle gender agreement in "No data found": corrected `न किमपि दत्तांशं प्राप्तम्` to `न कोऽपि दत्तांशः प्राप्तः` (since `दत्तांशः` is masculine nominative).
   - Replaced pseudo-verb derivations coined from nouns: Copy verb is `प्रतिलिख्यताम्` (from root `लिख्`), and Copied participle is `प्रतिलिखितम्` (not `प्रतिलिप्यताम्` / `प्रतिलिपितम्`).
   - Corrected Export causative passive to `निर्याप्यताम्` (not `निर्यात्यताम्`).
   - Corrected Confirm action to `स्थिरीक्रियताम्` / `दृढीक्रियताम्` (not `संपुष्यताम्`, which means to nourish).
   - Rejected Hindi loans for Account: use authentic Sanskrit `उपयोक्तृविवरणम्` (rejecting both Hindi `खाता` and `लेखा`).
   - Corrected About screen title: use nominal `परिचयः` (rejecting locative postposition `विषये`).
   - Used `ध्वनिः` for audio/sound to avoid ambiguity with `शब्दः` (Word).
   - Disambiguated Reset (`पुनःसज्जीक्रियताम्`) and Restore (`पुनःस्थाप्यताम्`).
   - Disambiguated Update (`अद्यतनीक्रियताम्`) and Refresh (`नवीक्रियताम्`).
   - Enshrined the foundational linguistic principle: Sanskrit is fully generative and mathematically rich in dhātus, upasargas, pratyayas, and samāsa rules; it lacks nothing and can derive precise vocabulary for any technical or future concept. Hindi must never be used anywhere as a substitute, crutch, or fallback.

3. **Malayalam Quality Rules & Corrections**:
   - Loosened transliteration rule to permit widely established digital loanwords (`ഹോം`, `മെനു`, `പ്രൊഫൈൽ`, `അക്കൗണ്ട്`, `ഡൗൺലോഡ്`, `ഓഫ്‌ലൈൻ`, `തീം`, `ഫയൽ`, `ഫോൾഡർ`, `ലിങ്ക്`, `ലൈസൻസ്`), while strictly prohibiting lazy transliterations where authentic native terms exist (`സൂക്ഷിക്കുക`, `അച്ചടിക്കുക`, `കമ്പനം`, `ഐച്ഛികം`, `സംഖ്യ`, `താൾ`).
   - Enforced action verb endings in `-ക്കുക` / `-ക` for button commands (including `ലോഗൗട്ട് ചെയ്യുക`).
   - Clarified negation distinction: `ഇല്ല` for action confirmation/refusal vs. `അല്ല` for non-identity.
   - Prohibited standalone postposition `കുറിച്ച്` for screen titles; established `വിവരണം`.
   - Corrected false friend `മുൻഗണനകൾ` (Priorities) to `താൽപ്പര്യങ്ങൾ` / `ഇഷ്ടങ്ങൾ` (Preferences).
   - Standardized Unicode atomic chillu characters and corrected corrupted spellings: `അറിയിപ്പുകൾ`, `നാൾവഴി`, `വിശദാംശങ്ങൾ`, `താൾ`, `ആരംഭിക്കുക`, and `അടയ്ക്കുക`.
   - Disambiguated distinct actions: Update (`നവീകരിക്കുക`) vs Refresh (`പുതുക്കുക`), Resume (`പുനരാരംഭിക്കുക`) vs Continue (`തുടരുക`), Exit (`പുറത്തുകടക്കുക`) vs Logout (`ലോഗൗട്ട് ചെയ്യുക`).
   - Harmonized System Default to `സിസ്റ്റം സ്വതവേ` / `तन्त्रसिद्धम्` across §8.4 and §8.5.

4. **UI Glossary Audit**:
   - Standardized all 8 tables (Navigation, Actions, Settings, Status/Feedback, Time/Date, Content/Fields, Confirmations, About) to contain exactly one value per cell, eliminating confusing parentheticals and slashes.
   - Corrected citation in §8.5.4 About table header to `guideline.md §1.6`.

## Files Changed

| File | Changes |
|---|---|
| `flutter_project_engineering_standard.md` | Cleaned all 8 lines of residual mojibake (`→`, `×`, `€`); removed duplicate code block close; updated Section 8.4 and 8.5 with single-value audited glossary, action disambiguation, corrected Malayalam spellings, fixed CI grep gate, and proper cross-references. |
| `docs/flutter_project_engineering_standard_README.md` | Updated Section 8 summary and Section 8.5 explainer to reflect dual-language quality mandates. |
| `plans/20260913_134700_section-8-5-sanskrit-malayalam-quality.md` | Created plan covering the scope and quality criteria. |
| `change_log/20260913_135500_section-8-5-sanskrit-malayalam-quality.md` | Accurately documented implemented changes, linguistic rationale, and audited verification results. |

## Verification

- Automated regex scan confirmed zero mojibake sequences (`â†’`, `Ã—`, `â‚¬`) and zero replacement characters (`U+FFFD`) across `flutter_project_engineering_standard.md`.
- Markdown code block fence parity verified (even count of 118 fences, zero unclosed blocks).
- Sanskrit CI grep gate updated to PCRE (`grep -nP`) with boundary lookarounds for standalone copulas/words and Unicode codepoint matching for nukta (`\x{093C}` / `[\x{0958}-\x{095F}]`); verified across all 141 glossary terms (0 false positives) and verified against Hindi smoke tests.
- All short UI label entries verified strictly within the 22-character limit of Section 8.6 (only one About-screen exception `प्रयुक्ता कृत्रिमबुद्धिः` at 24 characters noted in Section 8.6 exceeds it).
- Zero multi-value / parenthetical entries remain in the UI glossary tables.
- All plan and change log files verified to contain relative paths only and zero sensitive data.
