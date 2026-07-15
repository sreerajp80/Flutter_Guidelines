# Security

Use this document when the repository handles secrets, protected personal data, health data,
financial data, private files, or any local encrypted store.

If the app is not security-sensitive, keep this file short and document that decision explicitly.

---

## 1. Security Scope

- App: `<app name>`
- Data sensitivity level: `low`, `moderate`, or `high`
- Engineering standard profiles in force:
  - `Core Baseline`
  - `Sensitive Data Extension` if applicable
- Platforms in scope:
  - `Android`
  - `iOS`
  - `Windows`
  - `<other>`

---

## 2. Security Objectives

- `<objective 1>`
- `<objective 2>`
- `<objective 3>`

Example objectives:
- Protect locally stored data from casual extraction on a lost or stolen device.
- Prevent accidental disclosure through logs, backups, screenshots, or exports.
- Preserve recoverability and migration without weakening encryption.
- Comply with OWASP Mobile Top 10 controls for all applicable risk categories.

---

## 3. Threat Model Summary

Document the threats the product is designed to address and those it explicitly does not address.

### In Scope Threats

- `<lost or stolen device>`
- `<casual local access by another user>`
- `<accidental plaintext export>`
- `<log leakage of sensitive data>`
- `<reverse engineering of application logic from release binary>`

### Out Of Scope Threats

- `<fully compromised/rooted device>`
- `<physical hardware attacks>`
- `<attacks requiring OS-level compromise>`
- `<nation-state adversaries>`

---

## 4. Sensitive Data Inventory

| Data Type | Example | Where It Exists | Protection Required |
|-----------|---------|-----------------|---------------------|
| `<secret>` | `<example>` | `<memory/db/export>` | `<control>` |
| `<token>` | `<example>` | `<memory/storage>` | `<control>` |
| `<user data>` | `<example>` | `<storage/logs?>` | `<control>` |

---

## 5. Storage Model

### At Rest

- Primary local storage: `<sqflite/files/etc.>`
- Secure key storage: `<flutter_secure_storage/keychain/keystore/etc.>`
- Backup behavior: `<disabled/restricted/encrypted/plaintext not allowed>`

### In Memory

- Sensitive values are kept in memory: `<briefly / cached / long-lived>`
- Memory clearing strategy: `<lock/background/manual clear>`

### In Transit

- Network use: `<none / https api / internal network>`
- Transport protections: `<tls/pinning/none>`

---

## 6. Cryptography Design

Document only the design, not the secrets.

- Encryption algorithm: `<AES-256-GCM/etc.>`
- Key derivation: `<PBKDF2/Argon2/etc.>`
- Nonce or IV strategy: `<random per record>`
- Format versioning: `<how versioning works>`
- Legacy format support: `<yes/no and why>`

### Rules

- Keys, IVs, salts, and passwords must never be hardcoded.
- Randomness must use cryptographically secure generation.
- Encrypted formats must be versioned.

---

## 7. Authentication And Access Control

- App-lock strategy: `<none / biometric / pin / password / device credential>`
- Fallback behavior: `<behavior>`
- Session-expiry rule: `<rule>`
- Background lock rule: `<triggered on AppLifecycleState.paused>`
- Protected-route strategy: `<where enforced — go_router redirect or state guard>`
- Lock screen implementation: `<path to the lock widget/screen>`

---

## 8. Binary Protections

### 8.1 Obfuscation

All production release builds MUST be compiled with:

```bash
--obfuscate --split-debug-info=build/symbols/<platform>-<version>/
```

Obfuscation is a useful hardening step, not only a build optimization. It renames Dart class and
method names in the compiled binary to meaningless identifiers, raising the cost of casual
inspection and automated analysis. It is **not a strong security boundary on its own**: a
determined analyst with the binary and enough time can still reconstruct logic from the compiled
machine code. Do not treat it as a substitute for sound data security, proper secret management,
or server-side enforcement of sensitive operations.

Without obfuscation, the low-effort attack surface is larger — an attacker can:
- Read exact class names, method names, and string literals from the binary.
- Identify security-sensitive paths (authentication, encryption, validation logic) quickly.
- Reconstruct a partial view of the app's architecture from the symbol table alone.

The debug symbol files produced by `--split-debug-info` MUST be:
- Stored securely and retained for the lifetime of the released version.
- Never committed to source control.
- Accessible only to the engineering team for crash symbolication.

### 8.2 R8 / ProGuard

Android release builds run R8 code shrinking. Verify `proguard-rules.pro` keeps classes accessed
via reflection. See `docs/flutter_build_flavors_guide.md` for the required rules.

### 8.3 Debuggable Flag

Verify `android:debuggable=false` in the merged release manifest before every production release.
A debuggable release build allows an attacker to attach a debugger to the process, read memory,
and inject code.

---

## 9. Logging And Telemetry Policy

### Never Log

- Secrets
- Tokens
- Recovery codes
- Decrypted payloads
- Full database row content that may contain user data
- Sensitive personal data unless explicitly approved

### Allowed Diagnostic Context

- Operation name
- Screen or flow name
- Error category (not raw exception message if it may contain user data)
- Non-sensitive identifiers where justified

### Logging Controls

- Logger implementation: `<logger package — see engineering standard section 14>`
- Verbose logging gate: `<flavor/config flag — AppFlavorConfig.enableVerboseLogging>`
- Log level in production: `info` and above only
- Redaction strategy: `<e.g. user-provided field values replaced with [REDACTED]>`

---

## 10. Platform Security Controls

### Android

- `android:allowBackup`: `<true/false and why>`
  - For sensitive-data apps: set to `false` or use `android:fullBackupContent` to explicitly
    exclude sensitive directories.
- `android:fullBackupContent`: `<value>`
- Screenshot protection:
  - Android: set `FLAG_SECURE` on the window to prevent screenshots and screen recording.
  - Implementation: use `FlutterWindowManager` or a method channel to call
    `getWindow().addFlags(WindowManager.LayoutParams.FLAG_SECURE)`.
  - Apply on all screens showing sensitive data; remove on non-sensitive screens if UX requires.
- `android:debuggable`: MUST be `false` in release builds (verify per section 8.3).
- Root detection: `<if any — e.g. SafetyNet / Play Integrity API>`

### iOS

- Sensitive-screen capture policy:
  - Add an opaque overlay on the appropriate "will resign active" hook to prevent the OS
    capturing screen content for the app switcher.
  - Post-UIScene migration (Flutter ≥ 3.38, iOS 26 SDK builds): apply the overlay in
    `sceneWillResignActive(_:)` on your `UIWindowSceneDelegate` (or via
    `FlutterSceneLifeCycleDelegate` in a plugin).
  - Legacy `AppDelegate`-only projects: apply the overlay in `applicationWillResignActive`.
  - See `docs/flutter_build_flavors_guide.md` for the UIScene migration mechanics.
- Keychain usage: `<usage — e.g. flutter_secure_storage uses Keychain automatically>`
- Required privacy descriptions in `Info.plist`:
  - `<NSCameraUsageDescription if applicable>`
  - `<NSPhotoLibraryUsageDescription if applicable>`
  - `<Other permissions used>`
- ATS (App Transport Security): if offline app, verify no `NSAllowsArbitraryLoads` is set.

### Windows

- Windows Credential Manager or DPAPI via `flutter_secure_storage` for secret storage.
- Application data directory access: store sensitive files in
  `getApplicationSupportDirectory()`, not a shared or user-accessible directory.
- Verify no sensitive data is written to Windows Event Log.

---

## 11. Permissions

| Permission | Why It Is Needed | Requested When | Denial Handling |
|------------|------------------|----------------|-----------------|
| `<permission>` | `<reason>` | `<point of use>` | `<behavior>` |
| `<permission>` | `<reason>` | `<point of use>` | `<behavior>` |

Permission review rules:
- Request only permissions the app currently uses. Remove unused permissions promptly.
- For offline apps: verify `INTERNET` permission is absent from the merged release manifest.
- Dangerous permissions MUST be requested at the point of use with a rationale, not at startup.
- The app MUST function in a degraded but safe state if a non-critical permission is denied.

---

## 12. OWASP Mobile Top 10 Compliance

Review and sign off each item before every production release.

| ID | Risk | Control | Status |
|----|------|---------|--------|
| M1 | Improper Credential Usage | No hardcoded secrets; platform secure storage used | `verified / n/a / risk-accepted` |
| M2 | Inadequate Supply Chain Security | `pubspec.lock` committed; dependency audit performed; licenses verified | `verified / n/a / risk-accepted` |
| M3 | Insecure Authentication | App lock enforced; background lock on `paused` state | `verified / n/a / risk-accepted` |
| M4 | Insufficient Input/Output Validation | All user input validated; DB writes sanitized via parameterized queries | `verified / n/a / risk-accepted` |
| M5 | Insecure Communication | No network traffic (offline) OR TLS-only with valid certificates | `verified / n/a / risk-accepted` |
| M6 | Inadequate Privacy Controls | Data inventory reviewed; no PII in logs; sensitive fields excluded from backup | `verified / n/a / risk-accepted` |
| M7 | Insufficient Binary Protections | `--obfuscate` applied; `android:debuggable=false` verified; ProGuard applied | `verified / n/a / risk-accepted` |
| M8 | Security Misconfiguration | Permissions minimal; backup config explicit; debug features disabled in prod | `verified / n/a / risk-accepted` |
| M9 | Insecure Data Storage | No sensitive data in `SharedPreferences`; no sensitive data in unencrypted files | `verified / n/a / risk-accepted` |
| M10 | Insufficient Cryptography | Versioned encrypted formats; secure key derivation; no hardcoded keys | `verified / n/a / risk-accepted` |

For each `risk-accepted` item, document the justification and owner below the table.

---

## 13. Data Retention And Purge Policy

Define what data is stored, how long it lives, and what triggers deletion.

### Retention Schedule

| Data Type | Retention / Rotation Policy | Deletion Trigger |
|-----------|-----------------------------|------------------|
| `<user content>` | `<indefinite / N days>` | `<user delete action / account purge>` |
| `<log files>` | `<max 5 MB per file, 3 rotated files>` | `<size limit reached / app uninstall>` |
| `<temp export files>` | `<session only>` | `<export complete or app close>` |
| `<cache>` | `<OS discretion>` | `<stored in cache dir — cleared by OS under pressure>` |

### Purge Implementation

- Provide a user-accessible **"Delete all data"** action in Settings or the danger zone of the
  app that removes:
  - All database files
  - All log files
  - All cache files
  - All secure storage entries
  - All temporary files in the app's documents and cache directories

- Verify completeness of the purge in an integration test: after purge, reinstalling or reopening
  the app should result in a first-run state with no residual data.

- Temporary files (export staging, image resizing, import parsing buffers) MUST be created in
  `getTemporaryDirectory()` and deleted within the same session. If the session ends abnormally,
  clean up on next `resumed` or app startup.

### Data Purge On Uninstall

- Android: app data is deleted on uninstall by default unless `android:allowBackup=true`
  and a cloud backup exists. Verify backup config explicitly.
- iOS: Keychain items persist across uninstall on iOS unless explicitly deleted or unless
  `kSecAttrAccessibleAfterFirstUnlockThisDeviceOnly` access group is used. For sensitive keys,
  delete Keychain items on first launch after reinstall detection (compare stored install UUID
  against a newly generated one).
- Windows: `flutter_secure_storage` writes to Windows Credential Manager, which persists across
  app reinstall. Implement a first-run detection and credential purge if a clean uninstall
  + reinstall should produce a fresh state.

---

## 14. Backup, Import, Export, And Recovery

- Backup supported: `<yes/no>`
- Backup format: `<encrypted/plaintext/both>`
- Import supported: `<yes/no>`
- Recovery flow: `<description>`
- Plaintext export policy: `<disallowed / allowed with confirmation>`

### Validation Requirements

- Import parsing MUST reject malformed or unexpected data formats safely (no panic, no partial
  state corruption).
- Recovery flows MUST be tested like authentication flows — they are a high-value attack target.
- Export UX MUST clearly communicate the sensitivity of the exported file before the user saves
  or shares it.
- Encrypted exports MUST use a versioned format to support future decryption after algorithm
  or key-derivation changes.

---

## 15. Security Testing Strategy

| Area | Test Type | Notes |
|------|-----------|-------|
| Crypto format | Unit | Deterministic test vectors; encrypt then decrypt round-trip |
| Secret storage | Unit or integration | Verify no write to `SharedPreferences` or plaintext file |
| Lock / auth flow | Widget or integration | Lock triggers on `paused`; re-auth required on `resumed` |
| Backup and recovery | Integration | Full round-trip: export → purge → import → verify data intact |
| Data purge | Integration | After purge: verify DB, logs, cache, and secure storage are empty |
| Obfuscation | Release build verification | Confirm `--obfuscate` flag present in release commands |
| Debuggable | Release build verification | `android:debuggable=false` confirmed in merged manifest |
| Permission audit | Release build verification | Merged manifest contains only declared, needed permissions |

### Required Test Vectors Or Regression Areas

- `<deterministic crypto vector set — e.g. known plaintext → known ciphertext with fixed key/IV>`
- `<migration path — e.g. db v1 → v2 → v3 round-trip>`
- `<failure mode — e.g. corrupt DB file handled gracefully>`
- `<first-run after reinstall — Keychain / Credential Manager purge behavior>`

---

## 16. Incident Response Notes

- Triage owner: `<owner>`
- Severity model: `<brief model>`
- Immediate containment actions:
  - `<action 1: e.g. halt distribution of affected version>`
  - `<action 2: e.g. notify affected users>`
- User communication trigger: `<when users must be notified>`
- Patch release process reference: `docs/release_process.md`

---

## 17. Open Risks And Future Hardening

- Risk: `<risk>`
  Hardening option: `<option>`
- Risk: `<risk>`
  Hardening option: `<option>`

---

## 18. Security Review Checklist

Complete before every production release.

- [ ] Threat model reviewed and current.
- [ ] Sensitive data inventory updated.
- [ ] Logging policy reviewed — no new log statements introduce PII exposure.
- [ ] Storage and backup behavior reviewed.
- [ ] Permission usage reviewed — no unnecessary permissions.
- [ ] `--obfuscate` confirmed in all release build commands.
- [ ] Debug symbols archived for this release version.
- [ ] `android:debuggable=false` verified in merged release manifest.
- [ ] ProGuard rules verified (Android).
- [ ] OWASP Mobile Top 10 checklist (section 12) completed and signed off.
- [ ] Data retention policy reviewed — purge paths tested.
- [ ] Recovery, import, export, and migration paths tested.
- [ ] Platform-specific controls verified (FLAG_SECURE, Keychain behavior, Credential Manager).
- [ ] Tests cover the highest-risk failure modes.
