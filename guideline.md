# Common Folder-Structure Guideline for Flutter Apps

This guideline makes all of my Flutter apps follow the **same conventions** for the common
things they all share: About-screen constants, the release keystore, and the overall `lib/`
folder layout.

New apps MUST follow it from the start. Existing apps SHOULD be migrated toward it over time (migration is a separate task, not part of adopting this document).

Conformance words: **MUST** = required, **SHOULD** = expected default, **MAY** = optional.

---

## 1. About-screen constants (single source of truth)

All values shown on the **About** screen (app name, version, author, AI used, IDE used,
etc.) MUST come from **one JSON asset file**, not from hard-coded Dart strings scattered in
the UI. Changing About content is then a config edit, not a code change.

This is the standard pattern for all apps.

### 1.1 The three fixed paths

| Purpose                | Path                                  | What lives here                                                        |
| ---------------------- | ------------------------------------- | ---------------------------------------------------------------------- |
| Data (source of truth) | `assets/config/app_config.json`       | The actual About values, human-editable                                |
| Typed model            | `lib/core/config/app_config.dart`     | `AppConfig` class: `fromJson` + a safe `fallback`                      |
| Loader                 | `lib/core/config/config_service.dart` | `ConfigService`: loads the asset, degrades to fallback, checks version |

Every app MUST use exactly these paths and these class names (`AppConfig`, `ConfigService`).

### 1.2 The JSON file

`assets/config/app_config.json`:

```json
{
  "appName": "My App Name",
  "description": "One-line description of what the app does.",
  "version": "1.0.0",
  "build": "1",
  "details": {
    "Author": "Your Name",
    "Email": "<Email>",
    "License": "All libraries used are open source.",
    "AI used": "<AI Name>",
    "IDE used": "<IDE Name>"
  }
}
```

- `appName`, `description`, `version`, `build` are required top-level string fields.
- `details` is a free `string → string` map. Add or remove rows as needed; the About
  screen renders each key/value as a labelled row.
- Keep `version` and `build` in sync with `pubspec.yaml`. `ConfigService` will log a
  non-fatal debug note if they drift (see §1.5).

### 1.3 Register the asset in `pubspec.yaml`

```yaml
flutter:
  assets:
    - assets/config/
```

Without this line the file is not bundled and the app falls back to defaults.

### 1.4 The `AppConfig` model — `lib/core/config/app_config.dart`

Rules the model MUST follow:

- Immutable class with `appName`, `description`, `version`, `build`, and
  `Map<String, String> details`.
- A `static const AppConfig fallback` with safe built-in values, so a missing or
  malformed config never crashes the app.
- A `factory AppConfig.fromJson(Map<String, dynamic> json)` that reads field by field and
  falls back per field on a missing value or wrong type (never throws).

Reference implementation:

```dart
/// Typed values for the About screen, loaded from `assets/config/app_config.json`.
/// Changing About content is a config edit, not a code change.
class AppConfig {
  final String appName;
  final String description;
  final String version;
  final String build;
  final Map<String, String> details;

  const AppConfig({
    required this.appName,
    required this.description,
    required this.version,
    required this.build,
    this.details = const {},
  });

  /// Safe built-in value used when the config file is missing or malformed,
  /// so the app never crashes on a bad config.
  static const AppConfig fallback = AppConfig(
    appName: 'My App',
    description: 'A Flutter application.',
    version: '0.0.0',
    build: '0',
    details: {'License': 'All libraries used are open source.'},
  );

  factory AppConfig.fromJson(Map<String, dynamic> json) {
    String str(String key, String fallbackValue) {
      final value = json[key];
      return value is String ? value : fallbackValue;
    }

    Map<String, String> parseStringMap(String key) {
      final raw = json[key];
      if (raw is! Map) return const {};
      final out = <String, String>{};
      raw.forEach((k, v) {
        if (k is String && v is String) out[k] = v;
      });
      return out;
    }

    return AppConfig(
      appName: str('appName', fallback.appName),
      description: str('description', fallback.description),
      version: str('version', fallback.version),
      build: str('build', fallback.build),
      details: parseStringMap('details'),
    );
  }
}
```

### 1.5 The `ConfigService` loader — `lib/core/config/config_service.dart`

Rules the loader MUST follow:

- Constant `assetPath = 'assets/config/app_config.json'`.
- `load()` reads the asset, decodes JSON, and returns `AppConfig.fallback` on **any**
  error (missing asset, bad JSON, wrong shape).
- `loadAndVerify()` additionally compares the config's `version`/`build` with
  `package_info_plus` and logs a non-fatal debug note on mismatch.
- The asset loader SHOULD be injectable so tests can supply text without the real bundle.

Reference implementation:

```dart
import 'dart:convert';
import 'package:flutter/foundation.dart';
import 'package:flutter/services.dart' show rootBundle;
import 'package:package_info_plus/package_info_plus.dart';
import 'app_config.dart';

class ConfigService {
  static const String assetPath = 'assets/config/app_config.json';

  final Future<String> Function(String path) _loadAsset;

  ConfigService({Future<String> Function(String path)? loadAsset})
      : _loadAsset = loadAsset ?? rootBundle.loadString;

  Future<AppConfig> load() async {
    try {
      final text = await _loadAsset(assetPath);
      final decoded = jsonDecode(text);
      if (decoded is! Map<String, dynamic>) return AppConfig.fallback;
      return AppConfig.fromJson(decoded);
    } catch (_) {
      return AppConfig.fallback;
    }
  }

  Future<AppConfig> loadAndVerify({PackageInfo? packageInfo}) async {
    final config = await load();
    try {
      final info = packageInfo ?? await PackageInfo.fromPlatform();
      final mismatch =
          info.version != config.version || info.buildNumber != config.build;
      if (mismatch && kDebugMode) {
        debugPrint(
          'ConfigService: version/build in app_config.json '
          '(${config.version}+${config.build}) does not match the build '
          '(${info.version}+${info.buildNumber}).',
        );
      }
    } catch (_) {
      // Package info unavailable (e.g. plain unit test) — ignore.
    }
    return config;
  }
}
```

### 1.6 About screen — render `details` dynamically

The About screen MUST be **data-driven**: it iterates over `AppConfig.details` and renders
one row per entry, in order. Whatever key/value fields you add to `details` in
`app_config.json` MUST appear on the About screen automatically — adding or removing a key
in the JSON is the **only** change needed. The screen MUST NOT hard-code field names like
`Author` or `Email`.

Rules:

- Loop over `config.details.entries`; render one row (`ListTile` or equivalent) per entry.
- Skip any entry whose key or value is empty after trimming.
- Optional nicety: if a key equals `email` (case-insensitive), make its row tappable to
  open `mailto:<value>`.

Reference snippet (from the reference About screen):

```dart
for (final entry in config.details.entries)
  if (entry.key.trim().isNotEmpty && entry.value.trim().isNotEmpty)
    ListTile(
      title: Text(entry.key),
      subtitle: Text(entry.value),
      onTap: entry.key.trim().toLowerCase() == 'email'
          ? () => _openMail(entry.value)
          : null,
    ),
```

The fixed top rows (`appName` + `description`, and `version`/`build`) MAY be rendered
explicitly; everything else comes from the `details` loop.

> **Note on other constants.** This JSON pattern is only for **About-screen** metadata.
> Technical constants (database names, preference keys, thresholds, channel IDs) do NOT
> go in the JSON. Keep those in a plain Dart file such as
> `lib/core/constants/app_constants.dart` (class `AppConstants`, values only, no logic).

---

## 2. Release keystore + `key.properties`

Every app that ships a signed release MUST follow this.

### 2.1 Locations and names

| Item               | Location   | Name                                                         |
| ------------------ | ---------- | ------------------------------------------------------------ |
| Keystore           | `android/` | `android/<name>.jks` — **name is the user's choice per app** |
| Signing properties | `android/` | `android/key.properties` — **fixed name**                    |

- The keystore MUST live directly in `android/`. Its filename is free per app (e.g.
  `keystore.jks`, `release-keystore.jks`, `my_app_keystore.jks`) — whatever you choose,
  `key.properties` points to it.
- The properties file MUST be named `key.properties` (not `keystore.properties`).

### 2.2 `key.properties` format

`android/key.properties`:

```properties
storePassword=<store password>
keyPassword=<key password>
keyAlias=<key alias>
storeFile=<name>.jks
```

`storeFile` is the keystore filename you chose in §2.1 (a path relative to `android/app`
if Gradle resolves it that way in your project — keep it consistent with your
`build.gradle`).

### 2.3 Never commit secrets

Both files MUST be git-ignored. Add to the app's `.gitignore`:

```gitignore
# Signing — never commit
android/key.properties
android/*.jks
android/*.keystore
```

Keep a secure, offline backup of each app's keystore. Losing it means you can no longer
publish updates under the same signature.

---

## 3. Standard `lib/` folder structure

Use this baseline layout so every app feels the same. Small apps use a subset; larger apps
MAY add more folders — but the **About pattern (§1) and keystore rules (§2) are fixed and
MUST NOT change**.

```
lib/
  main.dart
  core/
    config/          # AppConfig + ConfigService (About). REQUIRED, fixed path.
    constants/       # app_constants.dart — technical constants, values only
    errors/          # error/exception types (optional)
    utils/           # small helpers, extensions
  models/            # data models / entities
  services/          # platform + business services
  repositories/      # data access (db, prefs, network)
  providers/         # or state/ — app state (Riverpod/Provider/etc.)
  screens/           # full-page screens (incl. the About screen)
  widgets/           # reusable UI widgets
  theme/             # colors, text styles, ThemeData
  l10n/              # localization (if the app is translated)
```

Rules:

- `core/config/` MUST exist and hold `AppConfig` + `ConfigService` exactly as in §1.
- The About screen lives under `screens/` (e.g. `screens/about_screen.dart`) and reads its
  values from `ConfigService` / `AppConfig` — it MUST NOT hard-code About text.
- Pick **one** state-management folder name per app (`providers/` **or** `state/`) and use
  it consistently.
- Larger apps MAY introduce a layered or feature-first structure (e.g. `domain/`, `data/`,
  `application/`, `presentation/`, or feature folders). When they do, the `core/config/`
  About pattern and the `android/` keystore rules still apply unchanged.

---

## 4. Quick checklist for a new (or migrated) app

- [ ] `assets/config/app_config.json` exists with `appName`, `description`, `version`,
      `build`, `details`.
- [ ] `assets/config/` registered under `flutter: assets:` in `pubspec.yaml`.
- [ ] `lib/core/config/app_config.dart` defines `AppConfig` with `fromJson` + `fallback`.
- [ ] `lib/core/config/config_service.dart` defines `ConfigService` with `load()` +
      `loadAndVerify()`.
- [ ] About screen reads from `ConfigService`, not hard-coded strings.
- [ ] About screen renders `details` dynamically (loops the map, no hard-coded field names).
- [ ] Release keystore is at `android/<name>.jks`; `android/key.properties` points to it.
- [ ] `android/key.properties`, `android/*.jks`, `android/*.keystore` are git-ignored.
- [ ] `lib/` follows the baseline layout in §3 (subset is fine for small apps).
