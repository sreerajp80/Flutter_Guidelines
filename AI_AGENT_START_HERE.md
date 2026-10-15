# AI Agent — Start Here

You are an AI coding agent (Claude Code, Codex, Gemini, Cursor, or any other) and a user has asked
you to build or change a Flutter app **"following these guidelines"**. This page tells you what to
check first, what to ask the user, what to read, and which gates must pass before the app can ship.

Paths below are relative to this guideline set. In an app that uses it as a submodule, prefix
them with `docs/guidelines/` (see `GUIDELINES_MANIFEST.md`).

---

## 0. Before anything: check the folder

Run only read-only checks here. Change nothing yet.

1. **Is the folder a git repository, and is `docs/guidelines/` present and filled?** If not,
   stop. Ask the user to set it up themselves (only the user adds the submodule,
   `DOCS_FOLDER_GUIDELINE.md` §1):

   ```bash
   git init
   ```

   ```bash
   git submodule add <REPOSITORY_URL> docs/guidelines
   ```

2. **Is there a `pubspec.yaml` at the root?** No → new app: go to §3 (intake). Yes → existing
   app: go to §5 (adoption).
3. **Is there a `docs/PROJECT_PROFILE.md`?** If yes, does it still have `<...>` placeholders or
   `TODO` values? Those are questions for the user (§3).
4. **What is installed?** Run `flutter --version` and `flutter doctor`. Report problems to the
   user. Do not install or upgrade anything without asking.

---

## 1. Ground rules (always)

1. **Plan, get approval, then change.** Write a plan to `plans/`, ask the user to approve it, and
   only then edit files. Afterwards write a change log to `change_log/`. Both use relative paths
   only and contain no local system details or secrets (engineering standard §21.1.1).
2. **Never guess project choices.** Platforms, stores, languages, app name, author, contact,
   package id and the About badge come from `docs/PROJECT_PROFILE.md`. If it is missing or has
   `<...>` placeholders, ask the user (§3).
3. **Never type tool versions from memory.** Use the latest stable Flutter and the Android
   Gradle/Kotlin versions it generates; read versions from `flutter --version` and the project
   files (engineering standard §5.3).
4. **Only touch declared platforms.** Do not create or edit `ios/`, `macos/`, `linux/`, … unless
   the profile declares that platform.
5. **Use simple English** in plans, change logs, questions and replies.
6. **Never change the guidelines submodule.** In an app, `docs/guidelines/` is read-only: never
   edit, pull, check out or commit it or its version. If a guideline needs a change, write a
   note for the user instead (`DOCS_FOLDER_GUIDELINE.md` §1.1). If the folder is empty or
   missing, stop and ask the user.

---

## 2. Read in this order

| # | Read | Why |
|---|---|---|
| 1 | The app's `CLAUDE.md` / `AGENTS.md` (if they exist) | Project-specific rules override the defaults below |
| 2 | The app's `docs/PROJECT_PROFILE.md` (template: `PROJECT_PROFILE_TEMPLATE.md`) | Product brief, platforms, stores, languages, identity, tools |
| 3 | `guideline.md` | Fixed conventions: About config, Android keystore, `lib/` layout, checklist |
| 4 | `flutter_project_engineering_standard.md` | The full rulebook — skim §1–§5, then read the section for the task |
| 5 | `language_packs/<name>.md` for each declared language that has one | Mandatory language-specific rules |
| 6 | `flutter_build_flavors_guide.md` | Only if the app uses flavors or you are touching build files |
| 7 | `platform_store_readiness.md` | Before any release, and when touching permissions or store-listed behavior |
| 8 | `release_process.md`, `security.md`, `architecture.md` | Templates — every app has its own filled-in copy under `docs/`; read that copy, not the template |

The plain-English explainers in `docs/*_README.md` summarize the long documents.

---

## 3. New app — intake (collect the facts first)

The user usually gives you a short brief: what the app is for, the app name, the tools they use,
maybe the platforms. **You must collect everything else from the user. Do not assume anything.**
The result is a filled-in `docs/PROJECT_PROFILE.md` (copied from `PROJECT_PROFILE_TEMPLATE.md`).

### 3.1 Intake rules

1. **Write down what the user already said.** Put each fact from the brief into the profile. Do
   not ask again for something the user already gave.
2. **Ask for everything else in one organised list.** Group the questions as in §3.2 and number
   them, so the user can answer "3: yes, 5: Google Play only". Ask only what is still open.
3. **Never fill a gap silently.** Every value is either *given by the user* or *proposed by you
   and confirmed by the user*. When you propose, say it is a suggestion and give the reason —
   for example: "Suggested app id: `com.<publisher>.notes`. It can never change after the first
   store release. Please confirm or change it."
4. **"You decide" is not enough for permanent or public values.** You may propose, but you need
   a clear "yes" for: app id, Dart package name, app name, publisher, public contact, platforms,
   stores, languages, licence and privacy policy URL. For technical choices (state management,
   structure tier, flavors) you may offer the documented default and record
   `default (proposed, confirmed by user)` once the user agrees.
5. **Unknown is allowed, but mark it.** If the user does not know yet, write
   `TODO — needed before <gate>` (for example `TODO — needed before first Google Play release`)
   instead of inventing a value. Treat every `TODO` as a blocker for the gate it names.
6. **Ask in rounds when needed.** The first round covers §3.2. Later rounds ask only what earlier
   answers opened up — for example, the user chose iOS → ask whether they have a Mac and an Apple
   developer account; the app stores health data → ask the Sensitive Data questions.
7. **Do not store private details.** Record the developer's OS only as "can build iOS/macOS
   locally: yes/no". Never record machine names, user names, local paths or personal emails
   unless the user wants that address published (engineering standard §21.1.1).
8. **Show the filled profile and get approval** before `flutter create` or any other file.

### 3.2 What to ask

Ask only the items the brief did not answer.

| Group | Ask about |
|---|---|
| **Product** | What the app does, in one line; the main features of the first version; who will use it; what is out of scope for now |
| **Identity** | App display name; publisher / organisation (as shown in stores); reverse-DNS app id; Dart package name; public support contact; website; copyright owner; licence of the app's own code |
| **Platforms and stores** | Which platforms ship; the store or channel for each; a minimum OS only if the user has a reason; whether iOS / macOS can be built locally (needs a Mac) |
| **Languages** | The template language; every other UI language; store-listing languages; who checks each translation (a role, not a personal email) |
| **Data and privacy** | Offline only, or online; backend or third-party API; user accounts / sign-in; what data is stored and where (device only, cloud); any personal, health, finance or secret data; analytics; crash reporting; ads; in-app purchases or subscriptions; device permissions (camera, location, files, notifications, …) and why each is needed; privacy policy URL |
| **Look and feel** | Brand colours, or the default theme; app icon (user supplies it, or a placeholder for now); light / dark mode; About screen rows and the optional signature badge (`guideline.md` §1.7) |
| **Tools and workflow** | Which AI agents will work on the project (Claude Code, Gemini, Codex, other); IDE; git hosting (GitHub or other); whether CI checks are wanted; who owns releases (a role) |
| **Technical defaults** | State management, structure tier, build flavors — offer the documented default and ask for a yes |

### 3.3 What the answers decide

After the intake, tell the user in a short summary what follows from their answers:

- the applicability profiles (engineering standard §1.2) — Production App Extension if the app
  ships to real users or stores; Sensitive Data Extension if it handles secrets, PII, health or
  finance data or local encrypted data;
- which store gates in `platform_store_readiness.md` apply (one per declared channel);
- which language packs apply;
- which agent instruction files to write (§4 step 3).

Then put the profile in a plan and ask the user to approve it.

---

## 4. New app — bootstrap sequence

Start only when the intake (§3) is done and the profile is approved. Do these as separate,
approved plans; do not jump ahead.

1. **Profile.** Save the approved profile as `docs/PROJECT_PROFILE.md`.
2. **Create the project** in the project root — the folder that already holds `docs/guidelines/`
   — with only the declared platforms. The final `.` means "this folder"; without it Flutter
   makes a new subfolder:

   ```bash
   flutter create --org <reverse-dns-prefix> --project-name <dart_package_name> --platforms=<declared platforms> .
   ```

3. **Instruction files.** Write `CLAUDE.md` and `AGENTS.md` from `CLAUDE_MD_GUIDELINE.md` and
   `AGENTS_MD_GUIDELINE.md` (both are mandatory, whatever agents are used). If the profile lists
   an agent that reads a different file by default (for example Gemini), make it read the same
   rules: either a short file in that tool's expected name that only points to `AGENTS.md`, or
   the tool's own setting that makes it read `AGENTS.md`. Check the tool's current docs for the
   file name; do not assume it. Never keep two different copies of the rules. Add
   `docs/GUIDELINES_MANIFEST.md` and the baseline `docs/` set (`DOCS_FOLDER_GUIDELINE.md` §6).
   Create `plans/` and `change_log/`.
4. **Foundations** (engineering standard): analysis options and formatting (§16), `material_ui` /
   `cupertino_ui` (§6.1), theme tokens (§6.1), thin `main.dart` and init order (§4.5), logging
   (§14), global error handling (§11), localization with every declared language and the parity
   test (§8), About config + screen (`guideline.md` §1), language picker if 2+ languages (§8.4).
5. **Platform identity.** Same app id on every platform; app name; icons and splash for every
   declared platform (§17.5); desktop setup per platform (§5.5); flavors if needed (§5.1).
6. **CI.** GitHub CI runs only the analyze/test checks (§19.1) and Dependabot (§19.2). No release
   builds, signing, or secrets in CI (§19.3).
7. **Features.** One plan per feature, in the order of the product brief (profile §0). Each
   feature is done only when the Definition of Done holds (§23) — tests, analyze clean, every
   declared language, tooltips, accessibility. If a feature needs a fact the profile does not
   have (a new permission, a backend, a new language), ask the user and update the profile first.
8. **Release.** Complete `docs/release_process.md` in full, and `docs/security.md` in full if the
   Sensitive Data Extension applies (both files exist from step 3). Pass the gate for **each declared channel** in
   `platform_store_readiness.md`, then follow the platform's release steps in
   `release_process.md` (§9 Android, §10 iOS, §11 Windows, §11A macOS, §11B Linux).

## 5. Existing app — adoption sequence

1. Read the code before proposing anything. Identify structure tier, state management, flavors,
   and which platforms already exist.
2. Create `docs/PROJECT_PROFILE.md` from what the code shows. Then run the intake (§3) for the
   gaps: ask the user to confirm what you found and to answer what the code cannot tell you.
3. Add `CLAUDE.md`, `AGENTS.md` (and a pointer file for other agents, §4 step 3),
   `docs/GUIDELINES_MANIFEST.md`.
4. Migrate toward the guidelines in small, separately approved plans — localization, About
   config, keystore layout, CI, then store gates. Do not mix migration with feature work.

---

## 6. What "ready for the stores" means

An app built with these guidelines is ready to publish on a declared channel when **all** of
these are true:

- [ ] `docs/PROJECT_PROFILE.md` has no `<...>` placeholders and no `TODO` that names this release
      or this channel.
- [ ] `guideline.md` §4 checklist passes.
- [ ] Engineering standard §23 Definition of Done holds for every merged change.
- [ ] A hardened release (`--obfuscate --split-debug-info`) is built locally for every declared
      platform; GitHub CI runs only the checks and Dependabot (engineering standard §19).
- [ ] Signing is set up for every declared platform, with no signing material in git
      (`guideline.md` §2, §2.5).
- [ ] `platform_store_readiness.md` §1 passes, plus the section for each declared channel:
      §2 Google Play · §3 App Store · §4 Microsoft Store / Windows download ·
      §5 Mac App Store / Developer ID · §6 Snap Store / Flathub / Linux packages.
- [ ] `release_process.md` §8 checklist passes for this release.

Store rules change every year. Before each release, check the store's current requirements. If
something changed, tell the user so they can update `platform_store_readiness.md` in the
guidelines repository (§1 rule 6).
