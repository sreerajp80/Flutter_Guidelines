# Change Log: Mandatory ARB String Externalization + No Local System Details In Plans/Change Logs

**Plan Reference:** [../plans/20260816_215540_mandatory-l10n-and-no-local-system-details.md](../plans/20260816_215540_mandatory-l10n-and-no-local-system-details.md)

**Date:** 2026-08-16

## Summary

Two guideline changes:

1. **Localization is now mandatory for every app**, even one that ships a single language. ARB files
   (`lib/l10n/app_en.arb`) plus `l10n.yaml` are Flutter's equivalent of Android's
   `res/values/strings.xml`. They must exist from day one, and every user-visible string must come
   from `AppLocalizations` — no raw string literals in widgets. Before this change, the rule only
   applied "when the app supports or plans to support more than one language".
2. **Plans and change logs must not contain local system details.** The earlier privacy rule
   covered absolute paths and secrets. It now also names OS user name, computer/host name, home and
   drive-letter paths, network share names, LAN/internal IP addresses, local server URLs with ports,
   device serial numbers, and personal email addresses — because these files are committed and may
   become public on the internet.

## Files Changed

### 1. `flutter_project_engineering_standard.md`

- **§8 intro** — localization is now stated as `Core Baseline`; single-language apps must complete
  both 8.1 (delegates) and 8.2 (string externalization).
- **§8.2** — retitled to "String Externalization (Mandatory, All Apps)" and rewritten:
  - `l10n.yaml`, `lib/l10n/app_<base>.arb`, and `@key` descriptions are required for every app.
  - Added the `strings.xml` parallel and the reason (adding a language later is a new file, not a
    rewrite).
  - Added a table of narrow exceptions that may stay plain literals: logs, non-UI exception
    messages, technical identifiers (asset paths, route names, map/JSON keys), developer-only
    screens.
  - Directory tree comments now mark `app_en.arb` REQUIRED and extra locales OPTIONAL.
  - Added an "Adding a second language later" three-step note.
  - Noted the `nullable-getter: false` call form.
- **§21.1** — the `plans/` and `change_log/` rows now point to a new sub-section.
- **§21.1.1 (new)** — the canonical privacy rule, plus a bad → good examples table.
- **§22.2** — AI-assistant rule updated to reference §21.1.1; added a rule requiring ARB-based
  strings.
- **§22.3** — change-log rule updated; added a final re-read check for leaked local details.
- **§23.1 (Definition of Done)** — privacy item now covers local system details; added an item for
  `l10n.yaml` / ARB existence and localized strings.

### 2. `guideline.md`

- **§3 tree** — `l10n/` comment changed from "(if the app is translated)" to
  "REQUIRED for every app, even single-language".
- **§3 rules** — expanded the plans/change-log privacy rule with the local-system-details list and a
  link to §21.1.1; added a new rule requiring `l10n.yaml`, the base ARB file, and `AppLocalizations`.
- **§4 checklist** — privacy box reworded; added two boxes for `l10n.yaml` / base ARB and for
  "no hard-coded user-visible strings".

### 3. `CLAUDE_MD_GUIDELINE.md`

- Template gained a **"Localization rules"** section (ARB mandatory, `flutter gen-l10n`, `@key`
  descriptions, allowed literal exceptions).
- Workflow rule 3 rewritten with the local-system-details list and the "write it for a stranger" line.
- §5 (verbatim sections) updated to mention local system details.
- Final self-check: privacy box reworded; added a localization box.

### 4. `AGENTS_MD_GUIDELINE.md`

- The same four edits as `CLAUDE_MD_GUIDELINE.md`, kept word-for-word aligned so both root files stay
  in sync.

### 5. `DOCS_FOLDER_GUIDELINE.md`

- §3 naming/path rule expanded with the local-system-details list and a pointer to §21.1.1 of the
  engineering standard.

### 6. `docs/flutter_project_engineering_standard_README.md`

- Added a plain-English setup item: create `l10n.yaml` and `lib/l10n/app_en.arb` even for a
  one-language app, with the `strings.xml` comparison and the allowed exceptions.

## Not Changed

- `GUIDELINES_MANIFEST.md` and `README.md` — checked; no wording contradicted the new rules.
- §8.3 (RTL) and §8.4 (formatting) — unchanged.
- No reference application was modified.

## Verification

- Searched all repository Markdown files for `if the app is translated`, `localized app`, absolute
  path patterns (`C:\`, `l:\`, `file:///`, `Users\`) and IP patterns (`192.168.`, `10.0.0.`).
  The only remaining hits are inside the new rule text and its bad → good example table, which is
  intentional. No existing plan or change log leaks a user name, host name, or IP address.
