# Plan: Trilingual Apps (EN/ML/SA), "Made with ❤️ from India" About Badge, Play Store Readiness, Tooltips

**Status:** completed

**Date:** 2026-09-13

## 1. What the issue is

The guidelines already make ARB string externalization mandatory (engineering standard §8.2), but they
stop there. Eight rules the owner wants enforced across every app are currently missing or only
implied:

1. **About-screen badge.** Every app must show the same "Made with ❤️ from India" footer on its
   About screen. Today §1.6 of `guideline.md` describes the dynamic `details` rows only, so each app
   invents its own footer (or has none).
2. **Three languages.** Every app must ship **English, Malayalam and Sanskrit**. The default is the
   system locale, and the user must be able to change the language inside the app. Today the standard
   only requires a single base ARB file, and there is no rule about an in-app language switcher or
   where the choice is persisted.
3. **Play Store readiness.** `release_process.md` covers hardening, signing and build commands but
   has no explicit "this app is publishable on Google Play" gate (target API level, App Signing,
   Data safety, privacy policy, store listing assets, testing track).
4. **Sanskrit must be Sanskrit.** Devanagari script makes Hindi text look "Sanskrit-ish". Without an
   explicit rule and a checkable list of Hindi markers, `app_sa.arb` will quietly drift into Hindi.
5. **Feature parity across languages.** Nothing forbids shipping a feature whose strings exist only
   in `app_en.arb`.
6. **Every text follows the language choice.** The About screen is the obvious hole: its content
   comes from `assets/config/app_config.json`, whose `details` keys and values are plain English
   strings today, so About stays English no matter what locale the user picks.
7. **Short menu/label text.** UI labels must be concise in all three languages; only descriptive
   text may be long. There is no length rule or ARB naming convention that separates the two.
8. **Tooltips on icon buttons.** §7.3 mentions tooltips as a suggestion ("use them for icon
   buttons"). It must be a hard requirement for every icon-only control, with localized text.

Two technical gotchas must be documented, or rule 2 will produce broken apps:

- Flutter's `GlobalMaterialLocalizations` / `GlobalCupertinoLocalizations` do **not** ship a
  Sanskrit (`sa`) translation. Adding `Locale('sa')` to `supportedLocales` without a fallback
  delegate throws at runtime for Material widgets.
- `intl` has no `sa` date/number symbols, so `DateFormat(..., 'sa')` throws. Formatting must fall
  back to a supported locale while the UI text stays Sanskrit.
- Malayalam and Devanagari glyphs are not guaranteed on every Android device; fonts must be bundled
  or an explicit fallback documented.

## 2. Files to be changed

| File | Change |
|---|---|
| `guideline.md` | New §1.7 "Made with ❤️ from India" badge; localized About config schema (§1.2/§1.4/§1.6); new §3 rules for the EN/ML/SA triad, language switcher, tooltips; §4 checklist boxes |
| `flutter_project_engineering_standard.md` | §7.3 tooltip requirement (hard rule) + new §7.8 tooltip rules; §8 rewritten for the three mandatory locales, in-app language switcher, Sanskrit quality rules with a Hindi-marker list, per-language feature parity, label conciseness budget + ARB key naming, font/script requirements; §17.4 font licensing note; §22 AI rules; §23.1 Definition of Done |
| `release_process.md` | New section "Google Play Store Readiness" + checklist block in §8; localization line items |
| `CLAUDE_MD_GUIDELINE.md` | Localization rules template block extended (triad, switcher, Sanskrit, tooltips, short labels); self-check boxes |
| `AGENTS_MD_GUIDELINE.md` | Same edits, kept word-for-word aligned |
| `docs/flutter_project_engineering_standard_README.md` | Plain-English explainer items for the new rules |
| `docs/release_process_README.md` | Plain-English explainer item for Play Store readiness |
| `README.md`, `GUIDELINES_MANIFEST.md` | Re-checked only; no contradiction expected |

## 3. The plan for the fix

### 3.1 About-screen badge (rule 1)

- Add `guideline.md` §1.7: the About screen MUST end with a centered badge reading
  **"Made with ❤️ from India"**, the heart rendered in red, the rest in the theme's muted
  foreground colour, below every other About content.
- The badge text comes from ARB (`madeWithLove`), so it renders in the active language; the ❤️ glyph
  stays in all three translations.
- Give a reference widget and the three ARB entries.
- The badge is fixed: not driven by `app_config.json`, not per-app editable, present in every app.

### 3.2 Localized About config (rules 1, 6)

- Extend the `app_config.json` schema so About content is localizable:
  - top-level `appName`, `description` may be either a plain string or a `{ "en": …, "ml": …,
    "sa": … }` map;
  - each `details` entry keeps a free key, and its value may be a plain string (for
    locale-independent values such as an email) or the same locale map;
  - the row **label** is resolved from ARB when a key matching `aboutDetail<Key>` exists, else the
    raw key is shown.
- Update `AppConfig` to store `LocalizedText` values, and give the About screen reference snippet a
  `resolve(...)` call. Keep backward compatibility: a plain string is treated as the same text for
  every locale.

### 3.3 Three mandatory locales + in-app switcher (rules 2, 5, 6)

Rewrite engineering standard §8 into:

- §8.1 minimum setup — `supportedLocales` MUST be `en`, `ml`, `sa`; `en` is the template ARB.
- §8.2 string externalization (existing rules kept) — now with three required ARB files and a key
  parity requirement.
- New §8.3 **Supported Languages** — the table of the three locales, the Sanskrit Material/Cupertino
  fallback delegate (with reference code), the `intl` formatting fallback, and the font/script
  requirement.
- New §8.4 **In-App Language Selection** — resolution order (saved choice → system locale →
  English), persistence key, a `LocaleController` reference implementation, `localeResolutionCallback`,
  the requirement that the switcher lives in Settings, lists each language endonym
  (English / മലയാളം / संस्कृतम्), applies immediately without restart, and that "System default"
  is an explicit option.
- New §8.5 **Sanskrit Quality** — proper Sanskrit, not Hindi in Devanagari: rules, a Hindi-marker
  token list that MUST NOT appear in `app_sa.arb`, a bad → good table, and a standard UI glossary
  (EN / ML / SA) for common terms so terminology is identical across apps.
- New §8.6 **Label Conciseness** — length budgets per language for menus/buttons/labels/tabs,
  the ARB key-prefix convention that separates short UI text from descriptive text, and the
  exemption for descriptive text.
- New §8.7 **Per-Feature Language Completeness** — a feature is not done until its strings exist in
  all three ARB files; add the parity test/CI snippet.
- Existing RTL and formatting sub-sections renumbered to §8.8 / §8.9.

### 3.4 Tooltips (rule 8)

- §7.3: turn the tooltip sentence into a MUST.
- New §7.8 **Tooltips On Icon-Only Controls**: every `IconButton`, `FloatingActionButton`,
  `PopupMenuButton`, icon-only `InkWell`/`GestureDetector`, `BottomNavigationBarItem` and
  `NavigationRail` destination that shows an icon without a persistent text label MUST supply a
  tooltip whose text comes from ARB; tooltip text is short (same budget as §8.6) and names the
  action, not the icon; a widget-test snippet that fails when any icon button lacks a tooltip.

### 3.5 Play Store readiness (rule 3)

- New `release_process.md` section "Google Play Store Readiness (Android)" covering: package id,
  target/min SDK policy, App Bundle + Play App Signing, monotonic `versionCode`, 64-bit and
  16 KB page size, permissions justification, Data safety form, privacy policy URL, content rating,
  store listing assets and their exact sizes, localized listings (English and Malayalam; Sanskrit
  is not an available Play listing language — ship it in-app), pre-launch report, closed/internal
  testing track before production, and phased rollout.
- Add a matching checklist block to §8 of the same document.

### 3.6 Consistency pass

- Mirror the new hard rules into `CLAUDE_MD_GUIDELINE.md` / `AGENTS_MD_GUIDELINE.md` templates and
  their self-check lists, word-for-word aligned.
- Update the two plain-English explainers.
- Update `guideline.md` §3 rules and §4 checklist.
- Re-grep the repository for statements that contradict the new rules (e.g. "even single-language").

## 4. Out of scope

- No reference application is created or modified; all changes are documentation.
- Translation of any real app's ARB files.
- iOS App Store / Microsoft Store readiness gates (only Google Play was requested).

## 5. After implementation

Write the change log to `change_log/20260913_hhMMss_trilingual-about-badge-playstore-tooltips.md`
referencing this plan with a relative path, and set this plan's `Status:` to `completed`.
