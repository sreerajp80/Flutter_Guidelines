# Flutter Guidelines

A reusable guideline set for building Flutter apps — by hand or with an AI coding agent — that are
consistent, maintainable, and ready to publish on **Google Play, the Apple App Store, the Microsoft
Store, the Mac App Store (or notarized macOS download), and the Snap Store / Flathub (or Linux
packages)**.

Nothing here is tied to one person, company, country or language. Every project records its own
choices — platforms, stores, languages, author, About options — in one file,
`docs/PROJECT_PROFILE.md`, and the rules apply only to what that file declares.

## AI agents: start here

Tell your AI agent: *"Follow the Flutter guidelines in `docs/guidelines/`, starting with
`AI_AGENT_START_HERE.md`."* That page gives the read order, the bootstrap steps for a new app, and
the release gates for each store.

## These are templates

This repository is a **source collection of templates**, not a deployed app. When you adopt
these guidelines in a real project, add this repository as a Git submodule at
`docs/guidelines/` (see [GUIDELINES_MANIFEST.md](GUIDELINES_MANIFEST.md)), copy the blueprint
templates you need into the app's `docs/` folder, and reference them from its `CLAUDE.md` and
`AGENTS.md`.

That is why cross-references inside the documents use a `docs/` prefix — for example
`docs/architecture.md`. The prefix means "once the file lives in your app's `docs/` folder",
**not** a path inside this repository (here the files are flat, side by side).

## The documents

| Document | What it is |
|---|---|
| [AI_AGENT_START_HERE.md](AI_AGENT_START_HERE.md) | Entry point for AI agents: read order, new-app bootstrap, existing-app adoption, "ready for the stores" checklist. |
| [PROJECT_PROFILE_TEMPLATE.md](PROJECT_PROFILE_TEMPLATE.md) | Fill-in template for the app's `docs/PROJECT_PROFILE.md`: identity, platforms, stores, languages, About options. |
| [guideline.md](guideline.md) | Common app conventions: About-screen JSON config, optional signature badge, Android release keystore rules (**source of truth for keystore rules**), signing on other platforms, baseline `lib/` layout, new-app checklist. |
| [flutter_project_engineering_standard.md](flutter_project_engineering_standard.md) | The master, project-agnostic rulebook — structure, UI, accessibility, localization, performance, database, logging, security, desktop setup, icons, CI, git, Definition of Done. |
| [platform_store_readiness.md](platform_store_readiness.md) | Release gates for every store: Google Play, App Store, Microsoft Store, Mac App Store / Developer ID, Snap Store / Flathub / Linux packages. |
| [release_process.md](release_process.md) | A step-by-step release runbook — versioning, hardening, signing, per-platform build commands (Android, iOS, Windows, macOS, Linux), distribution, rollback. |
| [flutter_build_flavors_guide.md](flutter_build_flavors_guide.md) | A platform-by-platform technical reference for build flavors on Android, iOS, Windows, macOS and Linux. |
| [architecture.md](architecture.md) | A per-project architecture blueprint template. You fill it in with one app's actual decisions. |
| [security.md](security.md) | A per-project security blueprint template — threat model, sensitive-data inventory, crypto design, platform controls, OWASP checklist. |
| [language_packs/](language_packs/) | Optional language-specific rules (grammar, glossary, CI gates). Mandatory only for projects that declare that language. Currently: [Sanskrit & Malayalam](language_packs/sanskrit_malayalam.md). |
| [profiles/](profiles/) | Worked examples of filled-in project profiles, e.g. [an English/Malayalam/Sanskrit app with a signature badge](profiles/example_en_ml_sa_profile.md). |
| [CLAUDE_MD_GUIDELINE.md](CLAUDE_MD_GUIDELINE.md) | Mandatory guideline for creating and maintaining the project-root `CLAUDE.md` (**MUST**). |
| [AGENTS_MD_GUIDELINE.md](AGENTS_MD_GUIDELINE.md) | Mandatory guideline for creating and maintaining the project-root `AGENTS.md` for other LLMs and AI agents (**MUST**). |
| [DOCS_FOLDER_GUIDELINE.md](DOCS_FOLDER_GUIDELINE.md) | How to create files in a project's `docs/` folder (local vs submodule, naming rules, file anatomy, catalog of recognized doc types). |
| [GUIDELINES_MANIFEST.md](GUIDELINES_MANIFEST.md) | Single, portable manifest file copied into a project's `docs/` folder to index all shared guidelines. |

## Where do I start?

- **Using an AI agent** — point it at [AI_AGENT_START_HERE.md](AI_AGENT_START_HERE.md).
- **Starting a new app** — fill in [PROJECT_PROFILE_TEMPLATE.md](PROJECT_PROFILE_TEMPLATE.md), then
  read [guideline.md](guideline.md) and [flutter_project_engineering_standard.md](flutter_project_engineering_standard.md).
- **Writing / maintaining project root `CLAUDE.md` & `AGENTS.md` (MUST)** — follow [CLAUDE_MD_GUIDELINE.md](CLAUDE_MD_GUIDELINE.md) and [AGENTS_MD_GUIDELINE.md](AGENTS_MD_GUIDELINE.md).
- **Structuring project `docs/` files** — follow [DOCS_FOLDER_GUIDELINE.md](DOCS_FOLDER_GUIDELINE.md).
- **Adding guidelines to an existing app** — copy [GUIDELINES_MANIFEST.md](GUIDELINES_MANIFEST.md) to your app's `docs/` folder.
- **Designing one app's structure** — fill in [architecture.md](architecture.md) for that app.
- **Setting up build flavors** — see [flutter_build_flavors_guide.md](flutter_build_flavors_guide.md).
- **Publishing to a store** — pass the gates in [platform_store_readiness.md](platform_store_readiness.md),
  then follow [release_process.md](release_process.md).
- **Handling sensitive data** — fill in [security.md](security.md) for that app.

## What applies where (by profile)

The engineering standard defines three applicability profiles. A document (or a marked section
of one) switches on only when its profile applies, so a small app is never forced into
release-process or high-security rules that do not fit it. Platform-, store- and language-specific
rules switch on only for what `docs/PROJECT_PROFILE.md` declares. Pick your app's profiles, then
read across the row.

| Profile | Applies to | Documents / sections in force |
|---|---|---|
| `Core Baseline` | Every app | Root `CLAUDE.md` (via [CLAUDE_MD_GUIDELINE.md](CLAUDE_MD_GUIDELINE.md), **MUST**); Root `AGENTS.md` (via [AGENTS_MD_GUIDELINE.md](AGENTS_MD_GUIDELINE.md), **MUST**); `docs/PROJECT_PROFILE.md` (**MUST**); [guideline.md](guideline.md); the Core Baseline rules of [flutter_project_engineering_standard.md](flutter_project_engineering_standard.md); language packs for declared languages; [architecture.md](architecture.md) (fill in what applies); [DOCS_FOLDER_GUIDELINE.md](DOCS_FOLDER_GUIDELINE.md) |
| `Production App Extension` | Apps shipped to real users / QA / stores | The above **plus** [platform_store_readiness.md](platform_store_readiness.md) (sections for declared channels), [release_process.md](release_process.md), [flutter_build_flavors_guide.md](flutter_build_flavors_guide.md) (if using flavors), and the `Production App Extension` sections of the engineering standard |
| `Sensitive Data Extension` | Apps handling secrets, PII, health, finance, or local encrypted stores | The above **plus** [security.md](security.md) and the `Sensitive Data Extension` sections of the engineering standard |

Profiles stack: a shipped password manager is in all three; a small internal tool is in
`Core Baseline` only.

## Plain-English explainers

Several of the technical documents have a matching `<name>_README.md` explainer in the
[docs/](docs/) folder that describes, in simple English, what the document says and how to use
it. Open the explainer first if a document looks dense. The available explainers are:

- [docs/architecture_README.md](docs/architecture_README.md)
- [docs/flutter_project_engineering_standard_README.md](docs/flutter_project_engineering_standard_README.md)
- [docs/flutter_build_flavors_guide_README.md](docs/flutter_build_flavors_guide_README.md)
- [docs/platform_store_readiness_README.md](docs/platform_store_readiness_README.md)
- [docs/release_process_README.md](docs/release_process_README.md)
- [docs/security_README.md](docs/security_README.md)

`guideline.md`, `AI_AGENT_START_HERE.md` and `PROJECT_PROFILE_TEMPLATE.md` are short enough to
read directly and have no separate explainer.
