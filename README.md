# Flutter Guidelines

This repository is the shared guideline set for my Flutter apps. It keeps every app
consistent — same conventions, same architecture language, same release and security
practices.

## These are templates

This repository is a **source collection of templates**, not a deployed app. When you adopt
these guidelines in a real project, most of the documents are copied into that app's `docs/`
folder and referenced from its `CLAUDE.md`; the index/README sits at the app's project root.

That is why cross-references inside the documents use a `docs/` prefix — for example
`docs/architecture.md`. The prefix means "once the file lives in your app's `docs/` folder",
**not** a path inside this repository (here the files are flat, side by side).

## The documents

| Document | What it is |
|---|---|
| [guideline.md](guideline.md) | My personal cross-app conventions: About-screen JSON config, the release keystore rules, and the baseline `lib/` folder layout. **This is the source of truth for keystore rules.** |
| [flutter_project_engineering_standard.md](flutter_project_engineering_standard.md) | The master, project-agnostic rulebook — rules that apply to *every* app (structure, UI, accessibility, performance, database, logging, security, CI, git, Definition of Done). |
| [architecture.md](architecture.md) | A per-project architecture blueprint template. You fill it in with one app's actual decisions. |
| [flutter_build_flavors_guide.md](flutter_build_flavors_guide.md) | A platform-by-platform technical reference for setting up build flavors on Android, iOS, and Windows. |
| [release_process.md](release_process.md) | A step-by-step release runbook — versioning, hardening, signing, per-platform build commands, distribution, rollback. |
| [security.md](security.md) | A per-project security blueprint template — threat model, sensitive-data inventory, crypto design, OWASP checklist. |

## Where do I start?

- **Starting a new app** — read [guideline.md](guideline.md) (the conventions to follow
  from day one) and [flutter_project_engineering_standard.md](flutter_project_engineering_standard.md)
  (the rules to build to).
- **Designing one app's structure** — fill in [architecture.md](architecture.md) for that app.
- **Setting up build flavors** — see [flutter_build_flavors_guide.md](flutter_build_flavors_guide.md).
- **Shipping a release** — follow [release_process.md](release_process.md).
- **Handling sensitive data** — fill in [security.md](security.md) for that app.

## What applies where (by profile)

The engineering standard defines three applicability profiles. A document (or a marked section
of one) switches on only when its profile applies, so a small app is never forced into
release-process or high-security rules that do not fit it. Pick your app's profiles, then read
across the row.

| Profile | Applies to | Documents / sections in force |
|---|---|---|
| `Core Baseline` | Every app | [guideline.md](guideline.md); the Core Baseline rules of [flutter_project_engineering_standard.md](flutter_project_engineering_standard.md); [architecture.md](architecture.md) (fill in what applies) |
| `Production App Extension` | Apps shipped to real users / QA / stores | The above **plus** [release_process.md](release_process.md), [flutter_build_flavors_guide.md](flutter_build_flavors_guide.md) (if using flavors), and the `Production App Extension` sections of the engineering standard |
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
- [docs/release_process_README.md](docs/release_process_README.md)
- [docs/security_README.md](docs/security_README.md)

`guideline.md` is short enough to read directly and have no separate explainer.
