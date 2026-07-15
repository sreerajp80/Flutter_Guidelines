## What does `release_process.md` say?

It is a **living release playbook** for your Flutter app. Unlike `architecture.md` (a blueprint of the system) or `security.md` (a threat model and control register), this document is a **runbook** — it is consulted at release time and walked through step by step. It has 15 sections covering every decision and action a Flutter release needs to make explicit:

- What is being released and to whom (Section 1)
- Who owns each part of the release process (Section 2)
- The version format and how build numbers increment (Section 3)
- Branch and merge policy, including hotfix strategy (Section 4)
- The flavor × mode matrix for `dev` and `prod` builds across platforms (Section 5)
- Mandatory release-build hardening — `--obfuscate`, `--split-debug-info`, R8/ProGuard rules, app size analysis, and the `android:debuggable=false` verification (Section 6)
- Signing material handling and keystore backup policy (Section 7)
- The full pre-release checklist split across Code & Quality, Performance, Security, Product & Documentation, and Artifact Validation (Section 8)
- Step-by-step Android, iOS, and Windows release procedures with exact build commands (Sections 9–11)
- Distribution channels per platform (Section 12)
- Rollback and hotfix process (Section 13)
- Release evidence — what to archive after each release (Section 14)
- Post-release checks (Section 15)

Right now it is a **blank template** — many fields say `<placeholder>` or `<owner>`. You fill it with your project's actual release decisions.

---

## How do you use it in a Flutter project?

Think of it as three things simultaneously.

**1. A release-readiness gate, not a coding reference**
Unlike the engineering standard or the architecture document, this one does not get consulted during daily development. It is opened at release time. Every item in Section 8 must be ticked, every command in Sections 9–11 must be run, and every artifact listed in Section 14 must be archived. A release that skips a Section 8 item is not a release — it is a risk that has not been classified yet.

**2. A reproducible build recipe**
Sections 9–11 contain the exact commands to produce a production artifact for each platform, including the obfuscation flags, the symbol path templating, and the size-analysis step. Anyone on the team — or your future self six months from now — can pull the repo at the release tag, follow these steps, and produce a byte-equivalent (or near-equivalent) build. No tribal knowledge, no "ask the lead developer".

**3. An evidence trail per release**
Section 14 is the audit record. Every release writes back: where the symbols are archived, what the size analysis showed, who signed off the OWASP checklist, where the artifact was uploaded. Six months later when a crash report arrives from production, Section 14 of the release notes for that version tells you exactly where to find the symbols to symbolicate it.

---

## What should you fill out BEFORE starting the project?

Most of `release_process.md` is filled out **before the first release**, not before the first line of code. But a small number of decisions must be made very early because they shape the build configuration, the `.gitignore`, and the CI workflow. Trying to retrofit them later is expensive.

| Priority | Section | Why Before Coding |
|----------|---------|-------------------|
| 🔴 Must | **§1 Release Scope** | Locks down which platforms ship and which engineering profiles apply — this gates the whole document |
| 🔴 Must | **§3 Versioning Policy** | The build-number increment rule must exist before the first `pubspec.yaml` version line is committed |
| 🔴 Must | **§5 Flavor Matrix** | Decides whether you need `dev`/`prod` flavors at all — feeds directly into `architecture.md §15` and the native flavor setup |
| 🔴 Must | **§6.1 Obfuscation & Symbols** | The symbol archive location must be decided before the first prod release build runs — without symbols, crash reports from that version are undecodable forever |
| 🔴 Must | **§7 Signing & Secrets** | Keystore ownership, backup locations, and `.gitignore` rules — a committed keystore is a security incident, so this must be settled before any signing config is written |
| 🟡 Soon | **§2 Roles & Responsibilities** | Even for a single-developer project, write down "I am the release owner, QA, and store uploader" — it forces the question of whether one person should hold all four roles |
| 🟡 Soon | **§4 Branch & Merge Policy** | Before the first PR is opened — protects `main` from accidental direct pushes |
| 🟡 Soon | **§6.2 ProGuard Rules** | Before the first `flutter build apk --release` — R8 silently strips reflection-accessed classes and the failure only shows up in release builds |
| 🟡 Soon | **§6.4 Debuggable Verification** | The `aapt2` check command should be ready before the first prod build — verifying it manually after release is already too late |
| 🟢 Later | **§8 Release Checklist** | Walk through before every release |
| 🟢 Later | **§9–§11 Platform Steps** | Refer to during each platform's release |
| 🟢 Later | **§12 Distribution Channels** | Fill when distribution channels are chosen (Play Store, TestFlight, sideload, MSIX, etc.) |
| 🟢 Later | **§13 Rollback & Hotfix** | Fill before first public release — you do not want to be designing this under pressure during an incident |
| 🟢 Later | **§14 Release Evidence** | Populate per release, never blank |
| 🟢 Later | **§15 Post-Release Checks** | Walk through after every release |

---

## How can AI use this document? What do you need to do?

### How AI uses it

When `release_process.md` is populated and placed in your project, an AI assistant reads it and can:

- Know the exact build command for each platform — including `--obfuscate`, `--split-debug-info=<path>`, `--split-per-abi`, and the size-analysis step — and never suggest a release build that omits any of them
- Know the symbol archive policy and remind you to copy `build/symbols/` to the secure archive location after every prod build
- Know the version-bump rule and update `pubspec.yaml` correctly when asked to prepare a release
- Know the flavor matrix and use the right `--flavor` argument on Android/iOS or the right `--dart-define=APP_FLAVOR=<value>` on Windows (cross-referencing the build flavors guide)
- Know the Section 8 checklist and walk through it item by item when you say "prepare release v1.2.3", rather than skipping ahead to the build command
- Know the rollback policy and produce a hotfix branch with the correct naming pattern when an incident happens
- Know that the full release checklist applies to hotfixes too — and not let "it's just a small fix" skip the obfuscation, symbol archival, or `android:debuggable=false` verification
- Know the `git tag v<version>` step and produce the exact commands to tag and push at the end of the release

Without this document, the AI has to guess your release procedure — and the most common release incidents (lost symbols, debuggable production build shipped, R8-stripped class crashing in release, keystore committed to git) are exactly the cases the AI cannot recover from after the fact.

### What you need to do

**Step 1 — Place it in the right location**

```
<project_root>/docs/release_process.md
```

**Step 2 — Add a rule to your `CLAUDE.md`**

Open your existing `CLAUDE.md` and add this rule:

```
Rule N: Read docs/release_process.md before suggesting any release build command,
        any version bump, any tag-and-push, or any hotfix workflow.

  Build command rules:
  - Every prod release build MUST include --obfuscate and
    --split-debug-info=build/symbols/<platform>-<version>/.
  - Every prod release build MUST be followed by a size-analysis step
    and a symbol archive step.
  - Never suggest a release command that omits these flags.

  Checklist rules:
  - When asked to "prepare release vX.Y.Z", walk through Section 8 in order.
    Do not skip directly to the build command.
  - Hotfixes follow the same checklist as full releases. No exceptions.

  Verification rules:
  - For Android prod builds, suggest the aapt2 debuggable verification step.
  - For Windows prod builds, suggest installing on a clean VM (not the dev machine).
```

**Step 3 — Fill out the 🔴 Must sections before the first release**

The 🔴 Must items shape your `.gitignore`, your `build.gradle.kts`, and your CI workflow. They cannot be retrofitted cleanly. The 🟡 Soon items can wait until just before the first release, but not later.

**Step 4 — Reference it explicitly when asking for help**

Instead of:

> *"Build a release APK"*

Say:

> *"Following docs/release_process.md §9 Android Release Steps, give me the full prod release command including obfuscation, split-debug-info path templated to the current pubspec.yaml version, and the post-build size-analysis step."*

Instead of:

> *"Help me ship version 1.2.0"*

Say:

> *"Following docs/release_process.md §8 Release Checklist, walk me through preparing v1.2.0 from a clean main branch. Do not skip any item. After the checklist, generate the platform-specific build commands per §9–§11."*

Instead of:

> *"There's a crash, ship a hotfix"*

Say:

> *"Following docs/release_process.md §13 Rollback And Hotfix Process, set up a hotfix branch for v1.2.1. Confirm the full §8 checklist still applies. Confirm the symbols for v1.2.1 will be archived per §6.1."*

**Step 5 — Use Section 14 as a per-release record, never blank**

After each release, populate Section 14 in a release notes file (or a dated entry inside `release_process.md` itself, depending on your team's preference). Tell the AI:

> *"Update release_process.md §14 with the v1.2.0 evidence: CI run #847, symbols archived at releases/v1.2.0/symbols/, APK at releases/v1.2.0/, OWASP checklist signed off by [name] on 2026-04-29."*

This keeps the audit trail current, so when a crash report arrives six months later you know exactly where to find the symbols.

---

## How the five documents work together

| Document | Answers |
|----------|---------|
| `flutter_project_engineering_standard.md` | *How* should all Flutter code be written? Universal rules for every project. |
| `flutter_build_flavors_guide.md` | *How* exactly do flavors wire into each platform's native build system? |
| `architecture.md` | *What* did this specific project decide? Tier, packages, schema, routes, signing strategy. |
| `security.md` | *What* does this specific project protect? What is sensitive, what is never logged, how is data encrypted? |
| `release_process.md` | *How* does this specific project ship? The exact commands, the checklist, the evidence trail. |

The release process document sits at the **end of the chain**: it consumes decisions recorded in `architecture.md §15` (signing strategy, build outputs supported), pre-release controls from `security.md §18` (the security review checklist), and command patterns from `flutter_build_flavors_guide.md` (per-platform build syntax). It is the operational layer that turns those decisions into a reproducible, auditable release. If `architecture.md` and `security.md` are the *what* and the *why*, `release_process.md` is the *how-to-ship-it-without-breaking-anything*.
