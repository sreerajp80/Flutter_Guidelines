# Plan: Fixes From Review of the Flutter L10n Parity Change

**Status:** completed (approved by the owner in review; see
`change_log/20260913_184500_flutter-review-fixes.md`)

**Date:** 2026-09-13

## 1. What the issue is

A review of [20260913_162500_flutter-l10n-parity-and-fixes.md](20260913_162500_flutter-l10n-parity-and-fixes.md)
found four problems:

1. **Note inside a code block.** In `flutter_build_flavors_guide.md`, the note about disabling App
   Bundle language splitting was placed inside the Gradle Kotlin code block. Anyone copying the
   file gets lines starting with `>`, and the Gradle build fails. The note also renders as code.
2. **Untrue claim in the change log.** The change log said the Sanskrit terms were "verified against
   the reviewed and approved glossary in Kotlin guidelines". Those terms have not been reviewed by a
   fluent reader. The change log also had no "needs native-reader review" list, which the new
   §8.5.4 review rule requires.
3. **Wrong Profile term in the records.** The plan and change log gave Profile as `പരിചയഃ` (the
   Sanskrit word written in Malayalam letters). The standard correctly uses `പ്രൊഫൈൽ` (Malayalam) /
   `परिचयः` (Sanskrit).
4. **Hindi check misses Markdown text.** The gate's word edges do not include Markdown marks, so
   `यह **है**`, ``यह `है` `` and `वह *था*` in a `*_sa.md` help file pass the check.

## 2. Files to be changed

| File | Change |
|---|---|
| `flutter_build_flavors_guide.md` | Remove the blockquote from inside the code block; put the explanation as a Gradle comment inside `bundle { language { } }` |
| `flutter_project_engineering_standard.md` | §8.5.1: widen the gate's word edges (`<`, `>`, `*`, `_`, backtick, `#`, `|`, `:`, `;`, `~`, `-`); self-test covers ARB/JSON, XML/HTML and Markdown samples; update the note |
| `change_log/20260913_163000_flutter-l10n-parity-and-fixes.md` | Fix the Profile term and the verification claim; add the "needs native-reader review" section and a correction note |
| `plans/20260913_162500_flutter-l10n-parity-and-fixes.md` | Fix the Profile term |

## 3. The fix

1. Replace the misplaced note with a three-line Gradle comment next to `enableSplit = false`.
2. New pattern (identical to the one proposed for the Kotlin guidelines, so both stacks behave the
   same):
   - Lookbehind edges: `[\s"'([{<>।,*_`#|:;~-]` or start of line.
   - Lookahead edges: `[\s"')\]}<>।,.?!*_`#|:;~-]` or end of line.
   - Tested before the change on 20 sample lines: all 11 Hindi samples caught (ARB/JSON, XML,
     Markdown bold, code span, italic, heading, table, list, colon, daṇḍa, `सेटिंग्स`); all 9
     Sanskrit samples pass (`स्थाप्यताम्`, `यथा तथा कथा`, `**पुनःस्थाप्यताम्**`, `` `स्थानम्` ``,
     `_अस्ति_`, `न कोऽपि दत्तांशः प्राप्तः`, the badge line, and the Hindi-looking prefixes `होम`,
     `थाली`).
3. Correct the records without rewriting their history: add a correction note pointing to the new
   change log.

## 4. Out of scope

- Adding test code for the JSON and asset parity rules in §8.7 (noted as a follow-up in the change
  log).
- Reviewing the Malayalam and Sanskrit wording itself (needs a fluent reader).
