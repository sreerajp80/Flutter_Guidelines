# Change Log: Trilingual Apps (EN/ML/SA), "Made with ❤️ from India" About Badge, Play Store Readiness, Tooltips

**Plan Reference:** [../plans/20260913_125208_trilingual-about-badge-playstore-tooltips.md](../plans/20260913_125208_trilingual-about-badge-playstore-tooltips.md)

**Date:** 2026-09-13

## Summary

Eight rules are now enforced across every app in the guideline set:

1. Every About screen ends with the fixed **"Made with ❤️ from India"** badge — centered, red heart,
   localized words, not per-app configurable.
2. Every app ships **English, Malayalam and Sanskrit**. The app starts in the system language when
   it is one of the three and in English otherwise, and the user can change it inside the app.
3. Every app must pass an explicit **Google Play Store readiness gate** before its first upload and
   before every production release.
4. **Sanskrit must be real Sanskrit**, not Hindi written in Devanagari — with a grep-able
   forbidden-marker list, a bad → good table, and a shared cross-app UI glossary.
5. **Every feature** works in all three languages; a feature with English-only strings is unfinished.
6. **Every user-visible text** follows the language choice — including the About screen, whose JSON
   config now supports per-locale values and ARB-resolved row labels.
7. **Menu, button, label, tab and tooltip text is short** in all three languages; descriptive prose
   is exempt, and ARB key prefixes make the distinction checkable.
8. **Every icon-only control must have a localized tooltip.**

Two failure modes that would otherwise break rule 2 are documented with reference code: Flutter ships
no Sanskrit translation for its own Material/Cupertino widgets (needs fallback delegates, or date
pickers and dialogs throw under `sa`), and `intl` has no Sanskrit CLDR data (formatting must fall
back to English patterns — never Hindi).

## Files Changed

### 1. `guideline.md`

- **§1.2** — `app_config.json` schema extended: any text value may be a plain string or a
  `{"en","ml","sa"}` locale map; detail keys are now `lowerCamelCase` identifiers whose visible
  label comes from ARB key `aboutDetail<Key>`.
- **§1.4** — `AppConfig` reference implementation rewritten around a new `LocalizedText` value type
  (plain or per-locale, resolution order exact locale → `en` → any). Backward compatible: a plain
  string means the same text in every language.
- **§1.6** — About rows now render a localized label and a locale-resolved value; added the
  `aboutDetailLabel(...)` mapping helper; fixed top rows must take their labels from ARB too.
- **§1.7 (new)** — the mandatory "Made with ❤️ from India" badge: placement, colours, semantics,
  the three ARB entries, and a `MadeWithLove` reference widget that keeps the heart red in every
  language. The wording is fixed and copied verbatim into every app:
  `Made with ❤️ from India` / `സ്നേഹത്തോടെ ❤️ ഇന്ത്യയിൽ നിന്ന്` / `सस्नेहं निर्मितम् ❤️ भारततः`.
  The heart sits mid-string in all three, which the reference widget handles.
- **§3** — `l10n/` tree comment now lists all three required ARB files; new rules for the language
  triad, key parity, Sanskrit quality, the in-app language picker, short labels, tooltips, the
  About badge, and the Play readiness gate.
- **§4** — checklist gained ten boxes covering the above.

### 2. `flutter_project_engineering_standard.md`

- **§7.3** — the tooltip line is now a pointer to a hard requirement instead of a suggestion.
- **§7.8 (new)** — "Tooltips On Icon-Only Controls (Mandatory)": the widget-by-widget table of what
  needs a tooltip and how it is supplied, the rules (ARB-sourced text, names the action not the
  icon, short-label budget), code examples, and a widget-test helper that fails when a tooltip is
  missing.
- **§8 intro** — rewritten: three mandatory languages, plus a map of the sub-sections.
- **§8.1** — `supportedLocales` is fixed at `en`, `ml`, `sa`.
- **§8.2** — all three ARB files required; key parity required; `@key` descriptions must mark UI
  chrome so the length budget is enforceable. The "adding a second language later" note became
  "adding a fourth language later".
- **§8.3 (new)** — the three languages table; **§8.3.1** the Sanskrit Material/Cupertino/Widgets
  fallback delegates with full reference code and registration order, plus the required widget test;
  **§8.3.2** the `formattingLocale(...)` helper for `intl`; **§8.3.3** font and script coverage for
  Malayalam and Devanagari.
- **§8.4 (new)** — in-app language selection: resolution order, persistence key and pre-first-frame
  read, immediate app-wide application, the picker's contents in each script, and a
  `LocaleController` + `localeResolutionCallback` reference implementation.
- **§8.5 (new)** — Sanskrit quality rules, the forbidden Hindi-marker token list with a CI grep
  gate (explicitly labelled a smoke test, not a proof), a bad → good table, and the standard
  EN/ML/SA UI glossary that all apps must share. The glossary is categorized — navigation,
  actions, settings, status/empty states, time and date, content fields, confirmation words, and
  About-screen labels matching the `aboutDetail<Key>` keys — and is preceded by a **Sanskrit form
  convention**: polite imperative passive (`रक्ष्यताम्`) for buttons and menu items, nominal form
  (`अन्वेषणम्`, `विन्यासः`) for titles, tabs, labels and statuses, indeclinables for yes/no words.
  So one English word legitimately has two Sanskrit entries when it is both an action and a
  heading. "Theme" is `रूपविन्यासः`.
- **§8.6 (new)** — label conciseness: per-language budgets for short text, rules, the exemption for
  descriptive prose, the ARB key-prefix convention, and a length test sketch.
- **§8.7 (new)** — per-feature language completeness, including localized content assets, tests in
  all three locales, and the ARB key-parity test.
- **§8.8 / §8.9** — former §8.3 RTL and §8.4 formatting renumbered; RTL now says to test via
  `Directionality` rather than adding Arabic or Hebrew to the fixed `supportedLocales`; formatting
  points at `formattingLocale(...)`.
- **§17.4** — font licensing now requires Malayalam and Devanagari coverage, bundled rather than
  runtime-fetched.
- **§22.2** — AI-assistant rules gained: add every key to all three ARB files, write Sanskrit not
  Hindi and flag uncertain strings, respect the length budget, tooltip every icon-only control,
  keep the About badge.
- **§23.1 / §23.2** — Definition of Done gained the ARB triad, parity, Sanskrit gate, length
  budget, tooltip, three-language screen check, About badge, and Play readiness items.

### 3. `release_process.md`

- **§9 step 12 (new)** — complete the Play readiness gate before uploading; steps renumbered.
- **§9A (new)** — "Google Play Store Readiness (Mandatory Gate)": application identity and
  versioning, API level / ABI / compatibility, signing and upload (AAB + Play App Signing + symbol
  upload), manifest and permission declarations, store account declarations (privacy policy, Data
  safety, content rating, account deletion), listing assets with exact sizes, listing localization
  (English and Malayalam; Sanskrit ships in-app because Play has no Sanskrit listing locale), and
  pre-launch verification with internal testing, pre-launch report, and staged rollout.
- **§8** — two new checklist blocks: "Localization" (11 items) and "Google Play Store Readiness"
  (10 items).

### 4. `CLAUDE_MD_GUIDELINE.md` and `AGENTS_MD_GUIDELINE.md`

- The "Localization rules" template block rewritten, word-for-word aligned between both files:
  three languages, key parity, Sanskrit-not-Hindi, the Sanskrit delegates and formatting helper,
  the in-app picker, short labels, tooltips, and the About badge.
- Self-check lists: the single localization box became five boxes covering the triad, Sanskrit,
  the picker, tooltips + short labels, and the About badge.

### 5. `docs/flutter_project_engineering_standard_README.md`

- Section 5–9 overview line now mentions the mandatory tooltip rule and the three-language
  localization model.
- Setup items rewritten and expanded into six plain-English entries: the three ARB files, the three
  mandatory languages with the Sanskrit delegate and font traps, the in-app language picker,
  Sanskrit-not-Hindi, the short-label budget, and tooltips on icon-only buttons.

### 6. `docs/release_process_README.md`

- Added the Section 9A summary line to the list of what the runbook covers, including the note that
  Sanskrit cannot be a Play listing language and that this is expected.

### 7. `README.md` and `GUIDELINES_MANIFEST.md`

- The `guideline.md` description row in both indexes now mentions the About badge and the three
  mandatory app languages.

## Not Changed

- `architecture.md`, `security.md`, `flutter_build_flavors_guide.md`, `DOCS_FOLDER_GUIDELINE.md` —
  checked; nothing in them contradicts the new rules.
- iOS App Store and Microsoft Store readiness — only Google Play was in scope.
- No reference application was created or modified; every change is documentation.

## Verification

- Searched all repository Markdown files for `single-language`, `one-language`,
  `if the app is translated`, and `app_<base>`; the only remaining hits are inside the new rule
  text that states there is no single-language app.
- Confirmed §7 sub-sections run 7.1–7.8 and §8 sub-sections run 8.1–8.9 with no duplicate or
  skipped numbers, and that the §8 intro table matches the sub-sections that follow.
- Confirmed the Android release steps renumber cleanly (1–14) after inserting the Play gate step.
- Scanned every Markdown file for stray control characters introduced while editing; none remain.
- Re-read this change log and its plan for local system details; both contain relative repository
  paths only.

## Known Follow-Ups (not blocking)

- The Sanskrit glossary in §8.5 covers common UI terms. It is meant to grow: when an app needs a
  term that is not listed, the term is added to the glossary in the standard rather than invented
  per app.
- The badge wording in §1.7 of `guideline.md` is supplied by the repository owner and is final.
  The other Malayalam and Sanskrit strings — the glossary and the inline examples — should still be
  reviewed by a reader of each language before an app adopts them verbatim.
