# Plan: Strong Security Baseline For Every App (Without Hurting Usability)

**Status:** completed

## The issue

The user wants security to be a top priority in these guidelines: **every app must be secure,
with no compromise**. At the same time, security must not make the app hard to use, and must not
slow down the app or the user.

Today the guidelines are strong only for apps under the `Sensitive Data Extension`. For every
other app, the security rules are thin:

- `flutter_project_engineering_standard.md` §15.1 (rules for all apps) has only five points:
  logging, permissions, and obfuscation. It says nothing about network security (HTTPS only,
  never turning off certificate checks), secrets in the app (keys in code, assets or
  `--dart-define` are public), sign-in tokens, deep links, WebViews, input from outside the app,
  Android component hardening, safe randomness, or third-party SDKs.
- The OWASP Mobile Top 10 checklist is only a `SHOULD` for normal production apps.
- There is no rule that stops a developer or AI agent from **weakening** a security control "to
  make it work" (for example turning on cleartext HTTP or trusting all certificates).
- There is no guidance on **balance**: how to keep security strong while keeping the app easy and
  fast (biometric unlock with fallback, lock grace period, crypto off the main isolate, no
  needless permission prompts, no blocking rooted devices without a reason).
- The "short" `security.md` for non-sensitive apps asks only for §10, §11 and §13, so network,
  secrets and entry points are never written down.
- The Definition of Done for every app has only one security line ("no secrets added to git").

## Approach

Add one clear, **mandatory Security Baseline for every app**, plus a short set of **usability and
performance rules** so the baseline does not add friction. Keep it version-agnostic (state the
behavior and how to check it; no Flutter version numbers). Do not renumber existing sections, so
all current cross-references (`security.md` §8.1, §10, §11, §12, §13; standard §15.1–§15.5) stay
valid.

## Files to change

1. `flutter_project_engineering_standard.md`
2. `security.md` (template)
3. `release_process.md`
4. `platform_store_readiness.md`
5. `PROJECT_PROFILE_TEMPLATE.md`
6. `AI_AGENT_START_HERE.md`
7. `CLAUDE_MD_GUIDELINE.md`
8. `AGENTS_MD_GUIDELINE.md`
9. `DOCS_FOLDER_GUIDELINE.md`
10. `guideline.md`
11. `README.md`
12. `GUIDELINES_MANIFEST.md`
13. `docs/security_README.md`
14. `docs/flutter_project_engineering_standard_README.md`
15. `change_log/<timestamp>_security-baseline-for-every-app.md` (new, after implementation)

No submodule files are involved.

## Planned changes

### 1. `flutter_project_engineering_standard.md`

**§2 Core Principles** — reword principle 7: "Every app is secure by default. Security is a
product requirement, never traded away for convenience, and designed so it does not burden the
user or the app."

**§15 intro (new, no new number)** — two short paragraphs:
- The §15.1 Security Baseline is a `MUST` for **every** app, in every profile. Profiles only
  *add* controls; they never remove baseline ones.
- If you are unsure whether data is sensitive, treat it as sensitive.

**§15.1 Core Security Rules → rename "Security Baseline (Every App)"** and expand into short
grouped rules. Keep the existing five points (logging, permissions, obfuscation text). Add:

- **Secrets in the app.** Anything shipped inside the app (Dart code, assets, `--dart-define`,
  `.env` files, native config) can be read by anyone, so it is public. Real secrets (server keys,
  signing keys, admin tokens) never ship in the app; they stay on a server. Keys that must ship
  (for example a maps or Firebase key) are restricted on the provider side (by app id /
  signature, API scope, quota).
- **Network.** HTTPS (TLS) only; no cleartext HTTP in release. Never turn off certificate
  checks (no `badCertificateCallback` that returns `true`, no global `HttpOverrides` that trusts
  all certificates in release). Keep platform defaults on (Android cleartext off, iOS/macOS ATS
  on). Set timeouts. Certificate pinning is **not** the default: use it only when the project
  controls certificate rotation and ships a backup pin, because a wrong pin locks every user out.
- **Tokens and credentials.** Any sign-in token, session cookie, refresh token or user
  credential goes in platform secure storage (`flutter_secure_storage` or equal), even in apps
  outside the Sensitive Data Extension. Never store a user's password. Sign-out removes tokens.
- **Sign-in (if the app has accounts).** Use OAuth 2.0 / OpenID Connect with PKCE through the
  system browser (or the platform's sign-in sheet / passkeys) — never an embedded WebView login
  form. Authorization is enforced on the server; the app never trusts its own checks for access
  to server data.
- **Input from outside the app.** Treat as untrusted and validate: deep links and app links,
  Android intents, push payloads, files the user imports (check size, type, and paths — block
  `../` and "zip slip" when extracting), clipboard, platform-channel arguments, QR codes, and
  server responses. Use parameterized SQL queries only. A deep link never performs a sensitive
  action (delete, pay, change account) without the user confirming in the app.
- **Deep links for sign-in callbacks** use verified links (Android App Links / iOS Universal
  Links) where the platform supports them, not a custom scheme alone.
- **WebView.** Avoid it when a native screen or the system browser works. If used: load only
  allowlisted HTTPS origins, enable JavaScript only when needed, expose no JavaScript bridge to
  content you do not control, and open other links in the system browser.
- **Android components.** `android:exported="false"` unless the component must be reachable
  from outside, and then it validates its input; `PendingIntent` uses `FLAG_IMMUTABLE` unless
  mutability is required; `FileProvider` paths are as narrow as possible; no world-readable or
  world-writable files.
- **Local storage.** App data lives in app-private folders
  (`getApplicationSupportDirectory()` / documents of the app), never in shared or public storage
  unless the user chose the location.
- **Randomness and crypto.** Use `Random.secure()` for anything security-related. Never invent
  your own crypto; use well-known, maintained libraries.
- **Release builds.** No debug menus, test accounts, backdoors, or "skip login" switches in
  production; developer tools are behind a non-production flavor.
- **Third-party SDKs.** Every analytics, ads, crash or other SDK is reviewed for what data it
  collects and sends, and listed in `docs/security.md` §4. Crash reports and analytics never carry
  secrets or personal data.
- **Data minimization.** Collect and keep only the data a feature needs.
- **Web (if declared).** No secrets in the web bundle; set a Content Security Policy where the
  host allows it.

**New §15.6 Security With Good Usability And Performance** (new section number at the end of
§15, so nothing is renumbered). Rules:
- Controls are proportionate to the data (`docs/security.md` §4). Low-risk apps do not get
  high-friction controls (forced app lock, re-login on every resume).
- Prefer platform features that are both safer and easier: biometrics with device-credential
  fallback, passkeys, system photo/file pickers (no broad storage permission), platform secure
  storage.
- Ask for a permission only at the moment it is needed, with a short reason; the app still works
  (in a reduced way) if the user says no.
- App lock (when used): a short, documented grace period after backgrounding instead of locking
  on every brief switch; the user can choose stricter settings.
- Rooted / jailbroken / emulator detection: by default warn or reduce features; block only when
  the threat model in `docs/security.md` §3 justifies it.
- Performance: crypto, key derivation, hashing and big file encryption run off the main isolate
  (`compute()` / `Isolate`). Tune key-derivation cost so unlock stays quick on a low-end target
  device, and derive once per unlock, keeping the key in memory only while unlocked. Initialize
  secure storage lazily so startup time is not hurt (§10.6).
- Security messages are clear and localized, tell the user what to do next, and every security
  screen (lock, biometric prompt fallback) is accessible (§7).
- Fail safe but recoverable: on a security error, deny the action and offer a clear way forward;
  never fail open (for example never skip the lock because biometrics failed to load).
- Every control that adds user friction is recorded in `docs/security.md` §7.1 with its reason.

**§15.2 Sensitive Data Extension** — add one line pointing to §15.6 for the usability and
performance rules (biometrics, grace period, key derivation in an isolate).

**§15.3 OWASP checklist** — make it a `MUST` for every app under the `Production App Extension`
too (items that do not apply are marked `n/a`, so small apps are not burdened). Name OWASP MASVS
as the detailed reference. Update the control text of M1, M4, M5, M8 to match the new baseline
(secrets public, external input, no disabled certificate checks, exported components).

**§19.1 CI** — under "other read-only check scripts", add a `MAY` example: a secret-scanning
check (pass/fail only, no secrets used).

**§22.2 AI agent rules** — add:
- Never weaken a security control to make something work (cleartext HTTP, trusting all
  certificates, `allowBackup`/`exported` loosened, extra permissions, debug flags, disabled
  checks). If a task seems to need it, stop and ask in the plan.
- Every plan that touches network, storage, auth, permissions, deep links, WebViews, platform
  files or dependencies has a short **"Security impact"** line.

**§23.1 Definition of Done (Core Baseline)** — add security items:
- The change follows the §15.1 Security Baseline; no security control was weakened.
- New network calls use HTTPS with normal certificate checks.
- New outside input (links, files, intents, payloads) is validated.
- `docs/security.md` was updated if the change touched permissions, storage, network, sign-in,
  SDKs or entry points.

**§24 Practical Guidance** — reword "Security requirements should be attached to product risk,
not copied blindly" to: "The baseline is for every app; extra controls are matched to product
risk, not copied blindly."

### 2. `security.md` (template)

- **Intro:** say the Security Baseline applies to every app. In the short form (non-sensitive
  apps), fill in at least the new §1.1, the new §3.1, §10, §11 and §13.
- **New §1.1 Security Baseline (every app):** a checklist mirroring standard §15.1 (secrets,
  network, tokens, sign-in, outside input, deep links, WebView, Android components, storage,
  release build, SDKs, data minimization), each `verified / n/a`.
- **New §3.1 Entry Points And Attack Surface:** table of every way data enters the app (deep
  links, intents, imports, push, WebView, platform channels, network APIs) with how each is
  validated.
- **§4 Sensitive Data Inventory:** add a note to list third-party SDKs and what they collect.
- **§5 In Transit:** expand: HTTPS only, certificate checks never disabled, pinning decision with
  reason and backup pin.
- **New §7.1 Usability Balance:** record grace period, fallback paths, root-detection response,
  and every friction-adding control with its reason (standard §15.6).
- **§10 Android:** add exported components, `PendingIntent` immutability, cleartext/network
  security config, `FileProvider` paths.
- **§12 OWASP table:** update the "MUST" wording (Sensitive Data and Production App) and the
  M1/M4/M5/M8 control text to match the standard.
- **§15 Testing table:** add rows for deep-link/intent validation, import path validation, and
  "no cleartext / no disabled certificate checks".
- **§18 Review Checklist:** add Security Baseline (§1.1) and entry points (§3.1) items.

### 3. `release_process.md` — Security checklist

- Add: "Security Baseline (`docs/security.md` §1.1) re-verified."
- Add: "No secrets in the shipped app (code, assets, `--dart-define`, native config)."
- Add: "No code path disables certificate checks; iOS/macOS ATS exceptions absent or documented."
- Change the OWASP line to "MUST under the Sensitive Data and Production App Extensions".

### 4. `platform_store_readiness.md`

- In the shared/common gate (or Google Play §2.4 plus a matching line for the other stores),
  add one line: the Security Baseline in `docs/security.md` §1.1 is verified before the first
  release to each channel. (Exact spot chosen after reading the file's common section.)

### 5. `PROJECT_PROFILE_TEMPLATE.md`

- In §0.1, add a note: "If unsure whether data is sensitive, treat it as sensitive." Add a row
  "Third-party SDKs that send data" so they reach `security.md` §4.

### 6. `AI_AGENT_START_HERE.md`

- §3.3: add "if unsure, choose the Sensitive Data Extension" and "the Security Baseline applies
  to every app".
- In the "ready for the stores" checklist / release step, add the Security Baseline check.

### 7–8. `CLAUDE_MD_GUIDELINE.md` and `AGENTS_MD_GUIDELINE.md`

- In the `## Security rules` template block, add three lines:
  - HTTPS only; never disable certificate checks.
  - No secrets in code, assets or `--dart-define` — anything in the app is public.
  - Never weaken a security control to make something work; ask first.
- Keep both files in sync.

### 9. `DOCS_FOLDER_GUIDELINE.md`

- Update the short-form minimum for `security.md` to §1.1, §3.1, §10, §11 and §13.

### 10. `guideline.md`

- New-app checklist: add "Security Baseline (engineering standard §15.1) followed;
  `docs/security.md` §1.1 filled in."

### 11–12. `README.md` and `GUIDELINES_MANIFEST.md`

- In the profile table, `Core Baseline` row: add "the Security Baseline (engineering standard
  §15.1, `security.md` §1.1)". Add one sentence: every app is secure by default; profiles only
  add controls.

### 13–14. Explainers `docs/security_README.md` and `docs/flutter_project_engineering_standard_README.md`

- Explain in simple English the new baseline, the new `security.md` sections, the usability
  balance rules, and that OWASP is now a MUST for production apps.

## What is deliberately NOT added (to avoid burden)

- No mandatory certificate pinning (it causes outages if done wrong).
- No mandatory root/jailbreak blocking (warn by default).
- No mandatory app lock for low-risk apps.
- No new CI workflow or bot; a secret scan is only a `MAY` check step.
- No new required document; everything goes into the existing `security.md`.

## Verification

- Grep for every changed cross-reference (`§15.1`, `§15.6`, `security.md` §1.1, §3.1, §7.1) and
  check they point to real sections.
- Check no Flutter version numbers were added (guidelines stay version-agnostic).
- Check `CLAUDE_MD_GUIDELINE.md` and `AGENTS_MD_GUIDELINE.md` security blocks match.
- Write the change log in `change_log/`.
