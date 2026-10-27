# Change Log: Strong Security Baseline For Every App

**Plan:** `plans/20261026_163823_security-baseline-for-every-app.md`

## Summary

Security is now mandatory for every app, not only for apps with sensitive data. A new Security
Baseline applies to all apps in every profile, and new rules keep security easy to use and fast.
No existing section was renumbered, so all old cross-references still work. No Flutter version
numbers were added.

## Changes

### `flutter_project_engineering_standard.md`
- §2 principle 7: every app is secure by default; security is never traded for convenience and
  must not burden the user or the app.
- §15 intro: the baseline is a MUST for every app; profiles only add controls; if unsure, treat
  data as sensitive.
- §15.1 renamed "Security Baseline (Every App)" and expanded: secrets in the app are public;
  HTTPS only and certificate checks never disabled; pinning only with controlled rotation and a
  backup pin; tokens in secure storage; OAuth/OIDC with PKCE via the system browser; server-side
  authorization; validation of all outside input (deep links, intents, imports, payloads,
  platform channels); no path traversal; deep links never do sensitive actions alone; WebView
  hardening; Android components, `PendingIntent`, `FileProvider`; private app folders;
  `Random.secure()` and standard crypto; no debug backdoors; third-party SDK review; data
  minimization; web rules.
- §15.2: points to §15.6 for usability and performance.
- §15.3: OWASP checklist is now a MUST for Production App Extension apps too (`n/a` allowed);
  MASVS named as the detailed reference; M1, M3, M4, M5, M8 controls updated.
- New §15.6 "Security With Good Usability And Performance": controls matched to risk, platform
  features, permission at the moment of need, lock grace period, warn-first on rooted devices,
  crypto off the main isolate and lazy init, clear localized messages, fail safe but
  recoverable, friction recorded in `security.md` §7.1.
- §19.1: a pass/fail secret-scanning check MAY be added to CI.
- §22.2: AI agents never weaken a security control without asking; plans get a "Security
  impact" line.
- §23.1: four new security items in the Core Baseline Definition of Done.
- §24: practical-guidance line reworded.

### `security.md` (template)
- Intro: baseline applies to every app; short form now needs §1.1, §3.1, §10, §11, §13.
- New §1.1 Security Baseline checklist, new §3.1 Entry Points table, Third-Party SDKs table in
  §4, expanded §5 In Transit, new §7.1 Usability Balance.
- §10: ATS on for every app; Android cleartext, exported components, `PendingIntent`,
  `FileProvider`.
- §12 OWASP wording and controls updated; §15 new test rows; §18 new checklist items.

### Other files
- `release_process.md`: four new Security checklist items; OWASP line now a MUST for production.
- `platform_store_readiness.md` §1: new "Security Baseline" rule for every store.
- `PROJECT_PROFILE_TEMPLATE.md` §0.1: baseline note, "if unsure, treat as sensitive", new
  third-party SDK row.
- `AI_AGENT_START_HERE.md`: new ground rule 6 "Security is never optional" (old rule 6 is now
  rule 7, reference updated); §3.3, §4 step 8 and §6 checklist mention the baseline.
- `CLAUDE_MD_GUIDELINE.md`, `AGENTS_MD_GUIDELINE.md`: three new lines in the Security rules
  template (kept identical).
- `DOCS_FOLDER_GUIDELINE.md`: short-form minimum for `security.md` updated.
- `guideline.md` §4 checklist: new Security Baseline item.
- `README.md`, `GUIDELINES_MANIFEST.md`: baseline added to the Core Baseline row; "secure by
  default" note.
- `docs/security_README.md`, `docs/flutter_project_engineering_standard_README.md`: explain the
  baseline, the new sections, the usability rules and the OWASP change.

## Deliberately not added
- No mandatory certificate pinning, root blocking, or app lock for low-risk apps.
- No new CI workflow or bot, and no new required document.

## Verification
- Searched for stale wording ("Core Security Rules", "SHOULD otherwise", old short-form minimum):
  none left.
- New section headings exist: standard §15.1, §15.6; `security.md` §1.1, §3.1, §7.1.
- References in §15.6 to §7, §10.5 and §10.6 point to the right sections.
