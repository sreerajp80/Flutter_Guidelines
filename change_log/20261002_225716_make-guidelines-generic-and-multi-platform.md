# Change log — Make the guidelines generic and ready for all store platforms

**Plan:** `plans/20261002_224009_make-guidelines-generic-and-multi-platform.md`

## Summary

The guideline set no longer hard-codes one owner's name, contact, country badge or languages, and
it now covers releasing to every major store: Google Play, Apple App Store, Microsoft Store, Mac
App Store / Developer ID (notarized) download, and Snap Store / Flathub / Linux packages. Per-project
choices now live in one file, `docs/PROJECT_PROFILE.md`. The previous owner-specific conventions
were kept as an optional language pack and a worked example profile.

## New files

| File | What it is |
|---|---|
| `AI_AGENT_START_HERE.md` | Entry point for any AI agent: ground rules, read order, new-app bootstrap, existing-app adoption, "ready for the stores" checklist. |
| `PROJECT_PROFILE_TEMPLATE.md` | Fill-in template for `docs/PROJECT_PROFILE.md`: identity, profiles, platforms + channels, languages, About options. |
| `platform_store_readiness.md` | Store gates. §1 every store; §2 Google Play (moved from `release_process.md` §9A, made generic); §3 App Store (new); §4 Windows / Microsoft Store (new); §5 macOS / Mac App Store + Developer ID notarization (new); §6 Linux / Snap / Flathub / packages (new); §7 recording. |
| `language_packs/sanskrit_malayalam.md` | Moved, unchanged in meaning: Sanskrit framework gaps, fonts, picker endonyms, Sanskrit & Malayalam quality rules, Hindi-marker CI gate, full glossary, label budget, store-listing notes, checklist. Old standard §8.5.1–§8.5.4 are now pack §4.1–§4.4. |
| `profiles/example_en_ml_sa_profile.md` | Worked example profile: English/Malayalam/Sanskrit, "Made with ❤️ from India" badge enabled, Google Play + Microsoft Store. Uses placeholders instead of a real name or email. |
| `docs/platform_store_readiness_README.md` | Plain-English explainer for the new store file. |

## Changed files

- `flutter_project_engineering_standard.md`
  - New §1.2.1 Project Profile (mandatory); rules depending on a platform, store or language apply
    only when declared.
  - §3.3 root layout: platform folders required only for declared platforms; `web/`, `AGENTS.md`,
    `docs/PROJECT_PROFILE.md` added; `flutter create --platforms=...` rule and one app id everywhere.
  - §5.2 commands include macOS and Linux. §5.5 is now "Desktop Build Setup (Windows, macOS,
    Linux)" with new §5.5.1 Windows (metadata, MSIX for Store vs signed direct download),
    §5.5.2 macOS (identity, entitlements, App Sandbox, Hardened Runtime, signing, flavors), and
    §5.5.3 Linux (build packages, oldest distro, `APPLICATION_ID`, desktop integration, packaging).
    §5.6 artifact table covers all five platforms.
  - §8 rewritten generic with the same sub-section numbers: declared languages, a generic
    fallback-delegate pattern for any language without a Flutter framework translation, generic
    `formattingLocale`, script-coverage fonts, language picker only for 2+ languages,
    `CFBundleLocalizations` for iOS/macOS, translation-quality rules and language packs (§8.5),
    generic label budget, parity test driven by a declared-locale list, real RTL rules when an
    RTL language is declared.
  - §10.7 size budget adds iOS, macOS, Linux. §17.4 script coverage generic. New §17.5 App Icons
    And Splash Screens. §19.2 CI is a per-platform matrix with correct runner OS and secrets rule.
    §20.4 `.gitignore` adds desktop packages and Apple/Windows signing files. §21 adds
    `docs/PROJECT_PROFILE.md` and a privacy policy. §22 and §23 use declared languages/platforms,
    the optional badge, per-platform permissions, and every declared store gate.
- `guideline.md` — retitled "Common App Conventions"; sample `app_config.json` uses placeholders
  and declared languages; `aiUsed` / `ideUsed` are optional rows; `website` / `privacyPolicy` rows
  added; §1.7 badge is now optional, set in the project profile, and allows `{heart}` anywhere;
  §2 is "Android release keystore"; new §2.5 signing on other platforms; §3 rules and §4 checklist
  use declared languages, platforms and store gates.
- `release_process.md` — §1 lists all five platforms with channels; §5 and §6.3 cover all desktops;
  §7 covers all signing material; §8 localization checklist generic and a new "Store Readiness
  (Every Declared Channel)" checklist; §9A now points to `platform_store_readiness.md`; §10 iOS
  steps expanded; §11 Windows adds Store vs signing and WACK; new §11A macOS and §11B Linux release
  steps; §12 distribution table pre-filled per channel.
- `flutter_build_flavors_guide.md` — intro, toolchain table (Linux packages), new macOS and Linux
  flavor sections, release matrix rows for macOS and Linux, new-project notes for macOS and Linux,
  generic language-split comment.
- `security.md` — platforms in scope include macOS and Linux; new macOS and Linux platform
  controls; Windows capability/signing lines; permissions and uninstall-purge notes for all platforms.
- `architecture.md` — platforms from the profile; new "Target Platforms And Distribution" and
  "Platform Differences" tables; screen-reader line covers all platforms.
- `CLAUDE_MD_GUIDELINE.md`, `AGENTS_MD_GUIDELINE.md` — removed private repository names and
  "learned from the nine files"; "from global rules" → "from this guideline set"; identity table
  adds platforms, stores, languages; docs table adds the profile and store gates; build commands
  for every platform; signing note for other platforms; generic localization section; updated
  self-checks.
- `DOCS_FOLDER_GUIDELINE.md` — `PROJECT_PROFILE.md` is baseline doc #1 (9 docs now); catalog adds
  `PROJECT_PROFILE.md` and `glossary.md`.
- `README.md`, `GUIDELINES_MANIFEST.md` — written as a reusable set; AI entry point; new files listed.
- `docs/*_README.md` explainers — updated to match.

## Notes

- Section numbers in the engineering standard (§1–§24 and §8.1–§8.9) were kept, so existing links
  still point to the right topic. Only §1.2.1, §5.5.1–§5.5.3 and §17.5 are new.
- Links that pointed to standard §8.5.x (Sanskrit/Malayalam rules) should now point to
  `language_packs/sanskrit_malayalam.md` §4.x. Links to `release_process.md` §9A.x should point to
  `platform_store_readiness.md` §2.x.
- Fixed in passing: the old sample Sanskrit `aiUsed` value started with a Malayalam letter; the
  example profile uses the Devanagari spelling.
- Store rules (sizes, SDK levels, Xcode versions) are marked "last checked 2026-10" and must be
  re-checked against each store's current documentation before release.
