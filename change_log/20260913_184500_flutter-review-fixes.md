# Change Log: Fixes From Review of the Flutter L10n Parity Change

**Date:** 2026-09-13

**Plan:** [../plans/20260913_183500_flutter-review-fixes.md](../plans/20260913_183500_flutter-review-fixes.md)

## 1. What changed

| File | Change |
|---|---|
| `flutter_build_flavors_guide.md` | The "Disabling App Bundle language splitting" blockquote was inside the Gradle code block, which would break a copied build file. It is removed; the same explanation is now a Gradle comment next to `enableSplit = false`. |
| `flutter_project_engineering_standard.md` | §8.5.1 Sanskrit gate: word edges now also include `<`, `>` and the Markdown marks `*`, `_`, backtick, `#`, `|`, `:`, `;`, `~`, `-`. The self-test now checks five samples (ARB/JSON, HTML/XML tag, Markdown bold, code span, italic). The note below the script explains the edges. |
| `change_log/20260913_163000_flutter-l10n-parity-and-fixes.md` | Added a correction note. Profile is now `പ്രൊഫൈൽ` / `परिचयः` (was the wrong `പരിചയഃ`). The verification section no longer says the terms were reviewed; it says they still need a fluent reader. Added the "needs native-reader review" section that §8.5.4 requires. Noted the code-block fix. |
| `plans/20260913_162500_flutter-l10n-parity-and-fixes.md` | Profile rationale corrected to `परिचयः` (Sanskrit) / `വിവരണം` (Malayalam Description). |

## 2. Verification

- The new gate pattern was run with GNU grep on 20 sample lines before editing: 11 Hindi samples
  matched, 9 Sanskrit samples did not.
- The pattern is identical to the one in the Kotlin guidelines plan
  `plans/20260913_183600_sanskrit-gate-markdown-edges.md` in that repository, so both guideline sets
  behave the same once it is approved there.
- No local machine paths or personal details in these records.

## 3. Still open
 
- (Resolved) Both items below resolved on 2026-09-13 in
  [20260913_191000_flutter-parity-test-and-fluent-reader-review.md](20260913_191000_flutter-parity-test-and-fluent-reader-review.md):
  - §8.7 parity test extended to cover ARB keys & translations, badge heart marker, `app_config.json`, and help assets.
  - The four Malayalam and Sanskrit terms and rules reviewed and approved by a fluent reader.
