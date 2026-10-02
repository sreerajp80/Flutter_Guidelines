# AI Agent — Start Here

You are an AI coding agent (Claude Code, Codex, Gemini, Cursor, or any other) and a user has asked
you to build or change a Flutter app **"following these guidelines"**. This page tells you what to
read, in what order, and which gates must pass before the app can ship.

Paths below are relative to this guideline set. In an app that uses it as a submodule, prefix
them with `docs/guidelines/` (see `GUIDELINES_MANIFEST.md`).

---

## 1. Ground rules (always)

1. **Plan, get approval, then change.** Write a plan to `plans/`, ask the user to approve it, and
   only then edit files. Afterwards write a change log to `change_log/`. Both use relative paths
   only and contain no local system details or secrets (engineering standard §21.1.1).
2. **Never guess project choices.** Platforms, stores, languages, app name, author, contact,
   package id and the About badge come from `docs/PROJECT_PROFILE.md`. If it is missing or has
   `<...>` placeholders, ask the user.
3. **Never type tool versions from memory.** Use the latest stable Flutter and the Android
   Gradle/Kotlin versions it generates; read versions from `flutter --version` and the project
   files (engineering standard §5.3).
4. **Only touch declared platforms.** Do not create or edit `ios/`, `macos/`, `linux/`, … unless
   the profile declares that platform.
5. **Use simple English** in plans, change logs and replies.

---

## 2. Read in this order

| # | Read | Why |
|---|---|---|
| 1 | The app's `CLAUDE.md` / `AGENTS.md` (if they exist) | Project-specific rules override the defaults below |
| 2 | The app's `docs/PROJECT_PROFILE.md` (template: `PROJECT_PROFILE_TEMPLATE.md`) | Platforms, stores, languages, identity |
| 3 | `guideline.md` | Fixed conventions: About config, Android keystore, `lib/` layout, checklist |
| 4 | `flutter_project_engineering_standard.md` | The full rulebook — skim §1–§5, then read the section for the task |
| 5 | `language_packs/<name>.md` for each declared language that has one | Mandatory language-specific rules |
| 6 | `flutter_build_flavors_guide.md` | Only if the app uses flavors or you are touching build files |
| 7 | `platform_store_readiness.md` | Before any release, and when touching permissions or store-listed behavior |
| 8 | `release_process.md`, `security.md`, `architecture.md` | Templates — fill in the app's own copies under `docs/` |

The plain-English explainers in `docs/*_README.md` summarize the long documents.

---

## 3. New app — bootstrap sequence

Do these as separate, approved plans; do not jump ahead.

1. **Profile.** Copy `PROJECT_PROFILE_TEMPLATE.md` to `docs/PROJECT_PROFILE.md` and fill it in
   **with the user**: name, org, reverse-DNS id, platforms, stores, languages, About options,
   applicability profiles.
2. **Create the project** with only the declared platforms:

   ```bash
   flutter create --org <reverse-dns-prefix> --platforms=<android,ios,windows,macos,linux> <app_name>
   ```

3. **Instruction files.** Write `CLAUDE.md` and `AGENTS.md` from `CLAUDE_MD_GUIDELINE.md` and
   `AGENTS_MD_GUIDELINE.md`. Add `docs/GUIDELINES_MANIFEST.md` and the baseline `docs/` set
   (`DOCS_FOLDER_GUIDELINE.md` §6). Create `plans/` and `change_log/`.
4. **Foundations** (engineering standard): analysis options and formatting (§16), `material_ui` /
   `cupertino_ui` (§6.1), theme tokens (§6.1), thin `main.dart` and init order (§4.5), logging
   (§14), global error handling (§11), localization with every declared language and the parity
   test (§8), About config + screen (`guideline.md` §1), language picker if 2+ languages (§8.4).
5. **Platform identity.** Same app id on every platform; app name; icons and splash for every
   declared platform (§17.5); desktop setup per platform (§5.5); flavors if needed (§5.1).
6. **CI.** Minimum checks (§19.1), then a release build job for every declared platform (§19.2).
7. **Features.** One plan per feature. Each feature is done only when the Definition of Done
   holds (§23) — tests, analyze clean, every declared language, tooltips, accessibility.
8. **Release.** Fill in `docs/release_process.md` and `docs/security.md` (if the Sensitive Data
   Extension applies). Pass the gate for **each declared channel** in
   `platform_store_readiness.md`, then follow the platform's release steps in
   `release_process.md` (§9 Android, §10 iOS, §11 Windows, §11A macOS, §11B Linux).

## 4. Existing app — adoption sequence

1. Read the code before proposing anything. Identify structure tier, state management, flavors,
   and which platforms already exist.
2. Create `docs/PROJECT_PROFILE.md` from what the code shows; ask the user to confirm the gaps.
3. Add `CLAUDE.md`, `AGENTS.md`, `docs/GUIDELINES_MANIFEST.md`.
4. Migrate toward the guidelines in small, separately approved plans — localization, About
   config, keystore layout, CI, then store gates. Do not mix migration with feature work.

---

## 5. What "ready for the stores" means

An app built with these guidelines is ready to publish on a declared channel when **all** of
these are true:

- [ ] `guideline.md` §4 checklist passes.
- [ ] Engineering standard §23 Definition of Done holds for every merged change.
- [ ] CI builds a hardened release (`--obfuscate --split-debug-info`) for every declared platform.
- [ ] Signing is set up for every declared platform, with no signing material in git
      (`guideline.md` §2, §2.5).
- [ ] `platform_store_readiness.md` §1 passes, plus the section for each declared channel:
      §2 Google Play · §3 App Store · §4 Microsoft Store / Windows download ·
      §5 Mac App Store / Developer ID · §6 Snap Store / Flathub / Linux packages.
- [ ] `release_process.md` §8 checklist passes for this release.

Store rules change every year. Before each release, check the store's current requirements and
update `platform_store_readiness.md` if something changed.
