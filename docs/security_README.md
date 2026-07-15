## What does `security.md` say?

It is a **living security blueprint template** for your Flutter app. It has 18 sections that force every security decision to be written down explicitly rather than left as assumptions:

- What data the app handles and how sensitive it is (Sections 1–2)
- What threats are in scope versus intentionally out of scope (Section 3)
- A complete inventory of every piece of sensitive data and where it lives (Section 4)
- How data is stored at rest, in memory, and in transit (Section 5)
- Your cryptography design — algorithm, key derivation, nonce strategy, format versioning (Section 6)
- App-lock, session-expiry, background-lock, and protected-route enforcement (Section 7)
- Binary protections — obfuscation, R8/ProGuard, debuggable flag (Section 8)
- What is allowed and prohibited in logs (Section 9)
- Platform-specific controls — Android FLAG_SECURE, iOS app-switcher overlay, Windows Credential Manager (Section 10)
- Every permission the app requests, why, when, and what happens on denial (Section 11)
- OWASP Mobile Top 10 compliance sign-off table (Section 12)
- Data retention schedule and the purge implementation (Section 13)
- Backup, import, export, and recovery rules (Section 14)
- Security testing strategy per area (Section 15)
- Incident response notes (Section 16)
- Open risks and future hardening items (Section 17)
- Pre-release security review checklist (Section 18)

Right now it is a **blank template** — every field says `<placeholder>`. You fill it with your project's actual security decisions.

---

## How do you use it in a Flutter project?

Think of it as three things simultaneously.

**1. A threat model before you write a single line of code**
Going through Sections 1–7 forces you to decide upfront: what data is sensitive, where it lives, how it is protected, and what the lock/unlock rules are. These decisions shape your database schema, your state model, your repository layer, and your `main()` sequence. Getting them wrong early is expensive to undo.

**2. A pre-release gate on every production build**
Section 18 is a checklist that must be completed before every release. Items include verifying `--obfuscate` is in the build command, confirming `android:debuggable=false` in the merged manifest, auditing log statements for PII exposure, and signing off the OWASP Top 10 table. It is not optional documentation — it is a release blocker.

**3. An audit trail for sensitive-data decisions**
Every time a decision is made — "we allow plaintext export with confirmation", "we use AES-256-GCM with per-record random nonces", "we do not support iOS Keychain purge on reinstall yet" — you record it here. Future you, a second developer, or an AI assistant can read it and understand the security posture without reading the code.

---

## What should you fill out BEFORE starting the project?

These sections must be decided and written **before writing any code**, because they directly shape your database schema, your `AppException` hierarchy, your state model, and your `main()` sequence.

| Priority | Section | Why Before Coding |
|----------|---------|-------------------|
| 🔴 Must | **§1 Security Scope** | Locks sensitivity level and which profiles apply |
| 🔴 Must | **§2 Security Objectives** | Your north star — shapes every security trade-off |
| 🔴 Must | **§3 Threat Model** | Defines what you are and are not protecting against — prevents over-engineering and under-engineering simultaneously |
| 🔴 Must | **§4 Sensitive Data Inventory** | Tells you what needs encrypting, what must never be logged, and what must never go into `SharedPreferences` |
| 🔴 Must | **§5 Storage Model** | Decides: sqflite vs secure storage vs plain files — shapes your entire persistence layer |
| 🔴 Must | **§7 Authentication & Access Control** | App-lock strategy and background-lock rule — shapes your `AppLifecycleService` and your go_router redirect guard |
| 🔴 Must | **§11 Permissions** | For your app: verify `INTERNET` is absent — this is your offline hard constraint |
| 🟡 Soon | **§6 Cryptography Design** | Fill before writing any encryption utility or secure storage wrapper |
| 🟡 Soon | **§8 Binary Protections** | Confirm `--obfuscate` is in all your release build commands before the first prod build |
| 🟡 Soon | **§9 Logging Policy** | Before writing `AppLogger` — the policy shapes what the logger is allowed to emit |
| 🟡 Soon | **§10 Platform Controls** | Android FLAG_SECURE, iOS app-switcher overlay — decide before building the first sensitive screen |
| 🟡 Soon | **§13 Data Retention & Purge** | Before writing your Settings screen — the "Delete all data" action must be implemented, not bolted on later |
| 🟢 Later | **§12 OWASP Top 10** | Fill in and verify before every production release |
| 🟢 Later | **§14 Backup / Import / Export** | Fill when those flows are designed |
| 🟢 Later | **§15 Security Testing** | Fill as the test suite grows |
| 🟢 Later | **§16 Incident Response** | Fill before first public release |
| 🟢 Later | **§17 Open Risks** | Populate from your project's risk register |
| 🟢 Later | **§18 Checklist** | Walk through before every production build |

---

## How can AI use this document? What do you need to do?

### How AI uses it

When `security.md` is populated and placed in your project, an AI assistant reads it and can:

- Know the sensitivity level and never suggest `SharedPreferences` for anything in the sensitive data inventory
- Know the logging policy and refuse to emit PII or sensitive field values in any log call it writes
- Know the cryptography design and use the correct algorithm, nonce strategy, and format version in any encryption utility it writes
- Know the app-lock rules and correctly implement the `AppLifecycleState.paused` → lock trigger in `AppLifecycleService`
- Know the offline constraint is in force and confirm `INTERNET` permission is absent before suggesting any package
- Know the threat model scope and not over-engineer controls for out-of-scope threats (e.g. rooted device protection when that is explicitly out of scope)
- Know the OWASP compliance status and flag any code it writes that would flip a verified item back to unverified
- Know the purge requirements and include the "Delete all data" path in any Settings screen it generates

Without this document, the AI has to guess your security posture — and a wrong guess in security is significantly more costly than a wrong guess in architecture.

### What you need to do

**Step 1 — Place it in the right location**

```
<project_root>/docs/security.md
```

**Step 2 — Add a rule to your `CLAUDE.md`**

Open your existing `CLAUDE.md` and add this rule alongside Rule 1:

```
Rule 3: Read docs/security.md before writing any repository method, any logger call,
        any encryption utility, any Settings screen, or any lifecycle handler.
        Never log values from the sensitive data inventory (Section 4).
        Never write INTERNET permission into any manifest.
        Never store sensitive values in SharedPreferences.
```

**Step 3 — Fill out the 🔴 Must sections before writing any code**

**Step 4 — Reference it explicitly when asking for code**

Instead of:

> *"Write the AppLogger implementation"*

Say:

> *"Following docs/security.md §9 logging policy, write the AppLogger implementation. Use the offline-mode, sensitivity-level, and never-log list from §1 and §4 of security.md."*

Instead of:

> *"Write the AppLifecycleService"*

Say:

> *"Following docs/security.md §7 authentication rules and docs/architecture.md §6 lifecycle table, write the AppLifecycleService. App lock is not required in v1 but the paused handler must flush pending DB writes."*

**Step 5 — Keep it current as the project evolves**

Every time a new sensitive screen is added, a new permission is required, or a security decision changes, update the relevant section. Then tell the AI:

> *"Updated docs/security.md §11 — camera permission added for future QR import feature, not yet in use. Verify no camera permission appears in the current merged manifest."*

---

## How the five documents work together

| Document | Answers |
|----------|---------|
| `flutter_project_engineering_standard.md` | *How* should all Flutter code be written? Universal rules for every project. |
| `flutter_build_flavors_guide.md` | *How* exactly do flavors wire into each platform's native build system? |
| `architecture.md` | *What* did this specific project decide? Tier, packages, schema, routes, signing strategy. |
| `security.md` | *What* does this specific project protect? What is sensitive, what is never logged, how is data encrypted? |
| `release_process.md` | *How* does this specific project ship? The exact commands, the checklist, the evidence trail. |

`security.md` sits at the **constraint layer**: it does not describe the system, it describes what the system is forbidden from doing. The engineering standard says *use a logger*; `security.md` says *and never log these fields*. `architecture.md` says *use sqflite*; `security.md` says *and never put sensitive data in `SharedPreferences`*. `release_process.md` says *build with `--obfuscate`*; `security.md` says *and verify `android:debuggable=false` in the merged manifest before shipping*. The other four documents tell you what to build and how to ship it; `security.md` tells you what the result is not allowed to expose.
