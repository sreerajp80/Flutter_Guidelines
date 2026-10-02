## What does `platform_store_readiness.md` say?

It is the **list of things each app store checks before it accepts your app**, written as
checklists ("gates"). It covers every place a Flutter app can be published:

- Rules that every store shares — permanent app id, ever-increasing build number, privacy policy,
  honest data disclosure, least-privilege permissions, in-app account deletion, real screenshots,
  testing the exact release build (Section 1)
- **Google Play** for Android — app id and versioning, target API level, App Bundle and Play App
  Signing, permissions, Data safety, listing assets, listing languages, internal testing and staged
  rollout (Section 2)
- **Apple App Store** for iOS — developer account and bundle id, required Xcode version, usage
  strings, privacy manifest, App Privacy answers, export compliance, screenshots, TestFlight
  (Section 3)
- **Windows** — Microsoft Store identity values and age rating, or a code-signed and timestamped
  package for direct download; the Windows App Certification Kit; clean-VM testing (Section 4)
- **macOS** — App Sandbox and entitlements, the Mac App Store path, and the Developer ID path with
  **notarization** and stapling so Gatekeeper lets users open the app (Section 5)
- **Linux** — desktop file, icons and AppStream metadata, then the Snap Store, Flathub, or direct
  packages such as AppImage and `.deb` (Section 6)
- How to record the result for each release (Section 7)

---

## How do you use it?

**1. Only read the parts you need.** Your app's `docs/PROJECT_PROFILE.md` lists the platforms and
stores you ship to. Apply Section 1, plus the section for each store you listed. An Android-only
app on Google Play never needs the macOS or Linux sections.

**2. Pass the gate before the first upload.** Items marked *(one-time)* — like the app id or
reserving the app name — are set once. Everything else is checked again before every
production release.

**3. Re-check the store's own rules.** Stores change their rules every year (target SDK levels,
required Xcode version, screenshot sizes). The numbers in the file were last checked on the date
shown at the top. If a store's current documentation says something different, the store wins —
update the file.

---

## Why does it matter?

A store rejection costs days. Most rejections come from the same small set of mistakes: a
missing permission explanation, a privacy form that does not match what the app does, a missing
privacy policy, a wrong screenshot size, or — on macOS — an app that was never notarized. This
file lists those mistakes per store, so you (or your AI agent) can catch them before you upload.
