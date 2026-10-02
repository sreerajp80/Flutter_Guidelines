# Project Profile — `<App Name>`

> **How to use this template.** Copy this file to your app as `docs/PROJECT_PROFILE.md` and fill in
> every `<...>` value. Delete rows that do not apply. This file is the single place that records
> the choices the guidelines leave open — platforms, stores, languages, identity and About
> options. Every rule that depends on a platform, store or language applies **only when this file
> declares it** (engineering standard §1.2.1).
>
> **AI agents:** read this file before writing code. If a value is still `<...>`, ask the user —
> never guess an author, company, email, package id, platform, store or language.
>
> **Privacy:** this file is committed. Put only public information here — a public support
> address, not a personal one if you do not want it published. Never put passwords, keys or
> keystore details here.

---

## 1. Identity

| Item | Value |
|---|---|
| App display name | `<Your App>` |
| Short description (one line) | `<What the app does>` |
| Organization / publisher | `<Company or person name, as shown in stores>` |
| Reverse-DNS app id | `<com.example.yourapp>` — permanent once published; used on every platform where allowed |
| Dart package name (`pubspec.yaml` `name`) | `<your_app>` |
| Public support contact | `<support address or URL>` |
| Website | `<https://...>` or `none` |
| Privacy policy URL | `<https://...>` — required before any public store release |
| Copyright line | `© <year> <owner>` |
| License of the app's own code | `<proprietary / MIT / Apache-2.0 / ...>` |

## 2. Applicability profiles (engineering standard §1.2)

- [ ] `Core Baseline` (always)
- [ ] `Production App Extension` — shipped to real users, external QA, or stores
- [ ] `Sensitive Data Extension` — secrets, PII, health, finance, or local encrypted data

## 3. Target platforms and distribution

Tick only the platforms the app ships on. Create only these platform folders
(`flutter create --platforms=...`). For every ticked channel, the matching gate in
`platform_store_readiness.md` MUST pass before the first release to it.

| Platform | Ship? | Minimum OS | Distribution channels |
|---|---|---|---|
| Android | `<yes/no>` | `minSdk <NN>` (documented reason) | `<Google Play / direct APK / other store>` |
| iOS | `<yes/no>` | `iOS <NN>` (≥ Flutter minimum) | `<App Store / TestFlight only>` |
| Windows | `<yes/no>` | `Windows 10 <build>` | `<Microsoft Store / signed MSIX download / signed installer>` |
| macOS | `<yes/no>` | `macOS <NN>` (≥ Flutter minimum) | `<Mac App Store / Developer ID notarized download>` |
| Linux | `<yes/no>` | `<oldest distro, e.g. Ubuntu 22.04>` | `<Snap Store / Flathub / AppImage / .deb / .rpm>` |
| Web | `<yes/no>` | `<browsers>` | `<hosting>` |

Build flavors: `<none / dev + prod / dev + staging + prod>` (engineering standard §5.1).

## 4. Languages (engineering standard §8)

| Locale | Language | Script | Role | Language pack |
|---|---|---|---|---|
| `en` | English | Latin | Template ARB, ultimate fallback | — |
| `<code>` | `<language>` | `<script>` | Full UI translation | `<language_packs/... or none>` |

- In-app language picker: `<required — 2+ languages / not needed — 1 language>`.
- Languages without a Flutter framework translation (need the fallback delegate, §8.3.1):
  `<codes or none>`.
- Store-listing languages per store: `<e.g. Google Play: en-US, es-ES; App Store: English, Spanish>`.
- Native-reader reviewer for each non-template language: `<role or team, not a personal email>`.

## 5. About screen (`guideline.md` §1)

| Item | Value |
|---|---|
| `details` rows to show | `<author, email, website, privacyPolicy, license, ...>` |
| Signature badge (§1.7) | `<off>` or `<on>` |
| Badge text — template language | `<e.g. "Made with {heart} by <Team>">` |
| Badge text — each other language | `<translated, with {heart}>` |

## 6. Optional project-specific values

| Item | Value |
|---|---|
| App size budget override (§10.7) | `<default or custom>` |
| State management | `<Riverpod / Bloc / Provider / ...>` |
| Structure tier (§3.1) | `<Tier 1 layer-first / Tier 2 feature-first>` |
| Release owner (role) | `<role>` |

A complete, worked example is in `profiles/example_en_ml_sa_profile.md`.
