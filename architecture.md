# Architecture

Use this document to describe the current system design of the Flutter app.

> **How to fill this in.** This is a template. Fill in only the sections that apply to your app
> and mark the rest `N/A` (with a one-line reason) rather than inventing content. A small
> single-screen tool will legitimately mark many sections `N/A`; a shipped sensitive-data app
> will fill in most of them. The goal is an accurate picture, not a fully-populated form.

## 1. Scope

- Product: `<app name>`
- Repository type: `application` or `package/plugin`
- Engineering standard profiles in force:
  - `Core Baseline`
  - `Production App Extension` if applicable
  - `Sensitive Data Extension` if applicable
- Platforms: `Android`, `iOS`, `Web`, `Desktop` as applicable

---

## 2. Goals And Non-Goals

### Goals

- `<goal 1>`
- `<goal 2>`
- `<goal 3>`

### Non-Goals

- `<non-goal 1>`
- `<non-goal 2>`

---

## 3. Architecture Summary

Describe the current architecture in one short paragraph.

Example:

> The app uses a Tier 2 feature-first Flutter structure with Riverpod for state management.
> Screens delegate business logic to use-case services, local persistence is isolated behind
> repository abstractions over a sqflite database, and app-wide configuration is injected at
> startup through Riverpod providers at the root widget tree. The app is fully offline; no
> network access is used or permitted.

---

## 4. Repository Structure

### Current Structure Tier

- `Tier 1` or `Tier 2`
- Why this tier is appropriate now:
  - `<reason 1>`
  - `<reason 2>`

### Top-Level Source Layout

```text
lib/
|-- <folder>
|-- <folder>
`-- main.dart
```

### Ownership Rules

| Path | Responsibility |
|------|----------------|
| `lib/...` | `<responsibility>` |
| `lib/...` | `<responsibility>` |

---

## 5. App Initialization Sequence

Document the exact order of initialization steps that run in `main()` before `runApp`. Getting
this order wrong causes platform crashes that only appear in release builds.

| Step | Code / Call | Notes |
|------|-------------|-------|
| 1 | `WidgetsFlutterBinding.ensureInitialized()` | Always first |
| 2 | FFI / platform binding init | e.g. `sqfliteFfiInit()` on Windows/Linux |
| 3 | Secure storage / key bootstrap | Before any encrypted read |
| 4 | Database open + migrate | Schema version N applied |
| 5 | Flavor config load | `AppFlavorConfig.instance` |
| 6 | Logging init | Logger configured before any log calls |
| 7 | Lifecycle observer registration | `WidgetsBinding.instance.addObserver(...)` |
| 8 | `runApp(...)` | |

Document any initialization that is deferred (lazy) and explain why.

---

## 6. App Lifecycle Behavior

Document what the app does in response to each lifecycle state change.

| Lifecycle State | App Behavior |
|----------------|--------------|
| `resumed` | `<e.g. resume timers, re-validate lock state>` |
| `inactive` | `<e.g. obscure screen content for sensitive views>` |
| `paused` | `<e.g. flush pending DB writes, trigger app lock>` |
| `detached` | `<e.g. finalize writes, close file handles>` |
| Memory pressure | `<e.g. clear image cache>` |

---

## 7. Offline Behavior

State whether the app is online, offline-first, or cache-assisted, and document the implications.

- **Connectivity requirement**: `fully offline` / `offline-first` / `online required`
- **Network permission**: `INTERNET permission absent` / `present but optional`
- **Offline data source**: `<sqflite / hive / files>`

If the app is **fully offline**:
- The merged Android release manifest MUST NOT contain `<uses-permission android:name="android.permission.INTERNET" />`.
  Verify this by inspecting the actual permissions in the built artifact, not via
  `output-metadata.json` (which only enumerates the APK files produced):
  - `aapt2 dump permissions build/app/outputs/apk/prod/release/<apk>` — lists every permission baked into the APK.
  - Or open the merged manifest XML at
    `build/app/intermediates/merged_manifests/prodRelease/AndroidManifest.xml` and search for `INTERNET`.
  - Or in Android Studio: Build → Analyze APK → select the APK → open `AndroidManifest.xml`.
- All dependencies have been audited for transitive network activity (see `docs/dependency_audit.md`
  or the audit log in the release checklist).
- The integration test suite includes an offline scenario test that runs with airplane mode
  simulated to verify no accidental network calls are attempted.

---

## 8. State Management

- Primary pattern: `Provider`, `Riverpod`, `Bloc`, etc.
- Why this pattern was chosen:
  - `<reason 1>`
  - `<reason 2>`
- State boundaries:
  - Widgets own: `<ui-only concerns>`
  - State layer owns: `<screen/app state concerns>`
  - Services own: `<business logic concerns>`

---

## 9. Data Flow

Describe the expected request and update path.

```text
Widget -> State Layer -> Service or Use Case -> Repository -> Datasource
```

If the app intentionally omits a layer, document that here.

### Rules

- Widgets must not know: `<sql/http/crypto/etc.>`
- Services must not know: `<navigation/copy/etc.>`
- Repositories abstract: `<api/db/cache/etc.>`

---

## 10. Error Handling Architecture

Document how errors are classified and propagated from the datasource layer to the UI.

- **Global error handler**: `FlutterError.onError` and `PlatformDispatcher.instance.onError`
  configured in `main()`. See `lib/core/errors/` for implementation.
- **Domain exception hierarchy**: sealed class in `lib/core/errors/app_exception.dart`.

| Exception Class | Thrown By | Meaning |
|----------------|-----------|---------|
| `StorageException` | Repository | DB read/write failure |
| `ValidationException` | Service | Input failed business rules |
| `<add more>` | `<layer>` | `<meaning>` |

- **Error escalation policy**: `<document when a recoverable error becomes a session error, etc.>`
- **Fatal error screen**: `<path to the widget shown on unrecoverable errors>`

---

## 11. Domain Model

### Current Schema Version

SQLite schema version: `<N>` (increment this whenever a migration is added)

Migration history:

| Version | Change Summary |
|---------|---------------|
| 1 | Initial schema: `<table names>` |
| 2 | `<change>` |

### Core Models Or Entities

| Type | Purpose | Mutable? | Notes |
|------|---------|----------|-------|
| `<ModelName>` | `<purpose>` | `No` | `<notes>` |
| `<ModelName>` | `<purpose>` | `No` | `<notes>` |

### Serialization Strategy

- JSON models: `<yes/no>`
- Database models: `<yes/no>`
- Separate domain entities from transport models: `<yes/no and why>`

### Database Indexes

| Table | Indexed Columns | Reason |
|-------|----------------|--------|
| `<table>` | `<column>` | `<query that needs it>` |

---

## 12. Dependency Management And Injection

- DI approach: `<provider tree / get_it / riverpod / manual wiring>`
- App-root dependencies:
  - `<dependency>`
  - `<dependency>`
- Test replacement strategy:
  - `<mock/fake/override approach>`

---

## 13. Navigation

- Navigation approach: `<Navigator 1.0 / go_router / auto_route / custom>`
- Route definition location: `<path>`
- Protected-route strategy: `<auth/app-lock gating pattern>`
- Deep-link support: `<yes/no>`

---

## 14. Persistence And External Systems

### Local Storage

- Database: `<sqflite/isar/hive/etc.>`
- WAL mode: `<enabled/disabled>`
- Key-value storage: `<shared_preferences/etc.>`
- Secure storage: `<flutter_secure_storage/etc.>`

### Network

- Network client: `<dio/http/none>`
- Offline behavior: `<online-only/offline-first/cache-assisted/fully offline>`

### Platform Channels Or Native Integrations

- `<integration>`: `<purpose>`
- `<integration>`: `<purpose>`

---

## 15. Environment And Build Model

- Flavors used: `<dev/prod/staging/none>`
- Runtime config mechanism: `<dart-define/config file/native flavor>`
- Build outputs supported:
  - `<debug apk>`
  - `<release apk split-per-abi>`
  - `<app bundle>`
  - `<windows MSIX>`
- Obfuscation: `<enabled for prod release — symbols stored at <location>>`

---

## 16. UI System

- Theme source of truth: `<path>`
- Design tokens location: `<path>`
- Shared widget strategy: `<where shared UI lives>`
- Accessibility expectations:
  - Minimum touch target: 48 × 48 dp on mobile
  - Color contrast: WCAG AA minimum (4.5:1 normal text, 3:1 large text)
  - Screen reader: TalkBack (Android), Narrator (Windows) tested before each release
  - Text scale: layouts verified at 1.0×, 1.5×, 2.0× text scale

---

## 17. Logging

- Logger implementation: `<logger package / custom>`
- Log file location (if applicable): `<path e.g. app cache dir>`
- Log rotation policy: `<max file size, max rotated files>`
- Verbose logging gate: `<flavor config flag>`
- Sensitive data policy: `<never logged / explicitly listed exceptions>`

---

## 18. Testing Strategy

| Test Type | Scope | Notes |
|-----------|-------|-------|
| Unit | `<scope>` | `<notes>` |
| Widget | `<scope>` | `<notes>` |
| Integration | `<scope>` | `<notes>` |
| Performance | `<scope>` | `<notes>` |

### Test Layout

```text
test/
|-- <mirrored folders>
|-- helpers/
`-- fixtures/
```

### Critical Test Areas

- `<critical logic area>`
- `<critical flow>`
- `<migration or parsing path>`
- Database upgrade path from version 1 to current
- App lock trigger and re-authentication flow (if applicable)
- Error boundary behavior for unhandled exceptions

---

## 19. Operational Constraints

Document constraints that shape implementation choices.

- Minimum supported OS versions: `<versions>`
- Performance constraints:
  - Cold startup target: under 2 seconds to first meaningful frame (release build)
  - Frame budget: 16 ms at 60 Hz; sustained jank above 5% is release-blocking
  - APK size budget: `<see engineering standard section 10.7>`
- Regulatory or store constraints: `<if any>`
- Team constraints: `<single developer / multi-developer / release cadence>`
- Offline constraints: `<no internet permission / no network packages>`

---

## 20. Decisions And Tradeoffs

Record the decisions that are likely to be questioned later.

| Decision | Chosen Option | Why | Tradeoff |
|----------|---------------|-----|----------|
| `<topic>` | `<choice>` | `<reason>` | `<tradeoff>` |
| `<topic>` | `<choice>` | `<reason>` | `<tradeoff>` |

---

## 21. Known Risks And Follow-Ups

- Risk: `<risk>`
  Mitigation: `<mitigation>`
- Risk: `<risk>`
  Mitigation: `<mitigation>`

---

## 22. Related Documents

- `README.md`
- `docs/flutter_project_engineering_standard.md`
- `docs/flutter_build_flavors_guide.md`
- `docs/release_process.md` (required for shipped apps)
- `docs/security.md` (required for sensitive-data apps)
