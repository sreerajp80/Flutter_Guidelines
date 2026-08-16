# Plan: Mandatory String Externalization (ARB) + No Local System Details In Plans/Change Logs

**Status:** completed

**Date:** 2026-08-16

## 1. What the issue is

### Issue A — String externalization is optional today

The engineering standard §8.2 says user-visible strings must be externalized **"When the app
supports or plans to support more than one language"**. `guideline.md` §3 also marks the
`lib/l10n/` folder as *"(if the app is translated)"*.

So a single-language app can legally hard-code every string. That is the opposite of what we
want. On Android, `res/values/strings.xml` is created and used even for a one-language app, so
translation later is only a matter of adding a new `values-xx/strings.xml`. Flutter's equivalent
of `strings.xml` is the **ARB file** (`lib/l10n/app_en.arb`) plus `l10n.yaml` and `flutter gen-l10n`.

The reference app (`SreerajP_TextApp`) already does this correctly: `l10n.yaml` at the root,
`lib/l10n/app_en.arb` + `app_ml.arb`, and generated `app_localizations.dart`. We want that shape
to be the mandatory baseline for **every** app, even one that ships only English.

### Issue B — "No sensitive data" rule does not clearly cover local system details

A policy already exists (see [change_log/20260808_204000_relative-paths-and-sensitive-data-policy.md](../change_log/20260808_204000_relative-paths-and-sensitive-data-policy.md)):
plans and change logs must use relative repository paths and must not contain secrets.

But it is written mainly around *paths* and *secrets*. It does not clearly forbid the other
local machine details that leak when a repository is pushed to a public host: the OS user name,
the computer / host name, home-directory paths, drive letters, LAN IP addresses, network share
names, local server URLs with ports, device serial numbers, and personal email addresses.

Plans and change logs are committed files. They can end up on the public internet. They must
read as if written for a stranger, with nothing that identifies the machine they were written on.

## 2. Files to be changed

| File | Change |
|---|---|
| `flutter_project_engineering_standard.md` | §8.2 rewritten to make ARB externalization mandatory for all apps (incl. single-language); §21.1 / §22.2 / §22.3 / §23.1 extended with the "no local system details" rule |
| `guideline.md` | §3 `l10n/` folder note becomes mandatory; §3 rules + §4 checklist updated for both issues |
| `CLAUDE_MD_GUIDELINE.md` | §4 workflow-rules template, §5 verbatim section, §8 self-check — add "no local system details"; add localization to the project-rules template |
| `AGENTS_MD_GUIDELINE.md` | Same edits as above, in its §4 / §6 / §9 |
| `DOCS_FOLDER_GUIDELINE.md` | §3 path/privacy rule extended with the local-system-details list |
| `docs/flutter_project_engineering_standard_README.md` | Plain-English explainer updated to say ARB files are required for every app |
| `GUIDELINES_MANIFEST.md` | No content change expected; re-checked only |

## 3. The plan for the fix

### 3.1 Localization becomes mandatory (Core Baseline)

In `flutter_project_engineering_standard.md` §8:

1. Change the §8 intro so localization is a Core Baseline requirement, not a "translated apps only"
   topic.
2. Rewrite §8.2 as **"String Externalization (Mandatory, All Apps)"**:
   - Every app MUST have `l10n.yaml` at the project root and at least one ARB file,
     `lib/l10n/app_en.arb` (or the app's own base locale).
   - Every user-visible string MUST live in the ARB file and be read through
     `AppLocalizations.of(context)`. Raw string literals in widgets are not allowed — this
     applies even when the app ships a single language.
   - Add one short line explaining the parallel: *ARB is the Flutter equivalent of Android's
     `res/values/strings.xml`; we create it from day one so adding a language later is only a
     new ARB file, not a rewrite.*
   - Add the narrow exceptions, so the rule is enforceable: debug/log messages, exception
     messages not shown to the user, asset keys/route names/map keys, and developer-only
     screens. Anything a real user reads goes in the ARB file.
   - Keep the existing ARB example, directory tree, and `flutter gen-l10n` command; change the
     tree comment so the second locale is shown as optional rather than the trigger for the rule.
3. Add a short **"Adding a second language later"** note: add `app_<code>.arb`, add the locale to
   `supportedLocales`, re-run `flutter gen-l10n`. No screen code changes.
4. Add the ARB rule to the Definition of Done (§23.1) and to §21/§22 checklists where the other
   Core Baseline musts are listed.

In `guideline.md`:

5. §3 tree: change `l10n/  # localization (if the app is translated)` to
   `l10n/  # ARB string files — REQUIRED for every app, even single-language`.
6. §3 Rules: add a rule that `l10n.yaml` + `lib/l10n/app_en.arb` MUST exist and all user-visible
   text MUST come from `AppLocalizations`.
7. §4 checklist: add two boxes — `l10n.yaml` exists; no hard-coded user-visible strings.

In `docs/flutter_project_engineering_standard_README.md`:

8. Update the localization bullet so the explainer matches the new hard rule, in simple English.

In `CLAUDE_MD_GUIDELINE.md` and `AGENTS_MD_GUIDELINE.md`:

9. Add a localization line to the project-rules template block, e.g.
   *"All user-visible text comes from `lib/l10n/*.arb` via `AppLocalizations` — never a raw
   string literal in a widget."*, and a matching self-check box.

### 3.2 No local system details in plans and change logs

10. Write one canonical rule sentence and reuse it in all five documents so the wording does not
    drift:

    > Files in `plans/` and `change_log/` are committed and may become public. They MUST use
    > relative repository paths only and MUST NOT contain any local system details — OS user
    > name, computer/host name, home or drive-letter paths (`C:\Users\...`, `l:\...`,
    > `file:///...`), network share names, LAN/internal IP addresses, local server URLs with
    > ports, device serial numbers, personal email addresses — or any secret (API key, token,
    > password, keystore passphrase, credential, PII).

11. Add a short **"How to write it instead"** table (bad → good), e.g.
    `l:\Android\MyApp\lib\main.dart` → `lib/main.dart`; `C:\Users\<name>\.gradle` →
    "the local Gradle home"; `192.168.1.42:8080` → "the local dev server".
12. Update the existing self-check boxes in all five documents from "zero sensitive data" to
    "zero sensitive data and zero local system details".
13. Re-check this repository's own `plans/` and `change_log/` files for any leaked local details
    and clean up anything found (the earlier pass covered paths; this pass covers user/host names
    and IPs too).

### 3.3 Consistency pass

14. Grep the whole repository for `if the app is translated`, `localized app`, and for
    `C:\`, `l:\`, `file:///`, `192.168.`, `10.0.`, `Users\` to confirm nothing contradicts the new
    rules.
15. This plan file and its change log will themselves follow the new rule (no local paths).

## 4. Out of scope

- No change to the reference app `SreerajP_TextApp`.
- No new guideline document is created; only existing ones are edited.
- RTL (§8.3) and formatting (§8.4) rules stay as they are.

## 5. After implementation

Write the change log to `change_log/20260816_hhMMss_mandatory-l10n-and-no-local-system-details.md`
referencing this plan with a relative path, and set this plan's `Status:` to `completed`.
