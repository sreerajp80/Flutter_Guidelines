# Common App Conventions for Flutter Apps

This guideline makes every Flutter app that adopts this guideline set follow the **same
conventions** for the common things they all share: About-screen constants, the Android release
keystore, and the overall `lib/` folder layout.

New apps MUST follow it from the start. Existing apps SHOULD be migrated toward it over time (migration is a separate task, not part of adopting this document).

Conformance words: **MUST** = required, **SHOULD** = expected default, **MAY** = optional.

**Per-project choices live in `docs/PROJECT_PROFILE.md`** (copied from
`PROJECT_PROFILE_TEMPLATE.md`): app name, author, contact, declared languages, target platforms,
stores, and whether the optional About badge (§1.7) is on. This guideline never hard-codes a
person, company, country or language; examples use placeholders such as `<Your App>`.

---

## 1. About-screen constants (single source of truth)

All values shown on the **About** screen (app name, version, author, contact, license, etc.) MUST
come from **one JSON asset file**, not from hard-coded Dart strings scattered in the UI. Changing
About content is then a config edit, not a code change.

This is the standard pattern for all apps.

### 1.1 The three fixed paths

| Purpose                | Path                                  | What lives here                                                        |
| ---------------------- | ------------------------------------- | ---------------------------------------------------------------------- |
| Data (source of truth) | `assets/config/app_config.json`       | The actual About values, human-editable                                |
| Typed model            | `lib/core/config/app_config.dart`     | `AppConfig` class: `fromJson` + a safe `fallback`                      |
| Loader                 | `lib/core/config/config_service.dart` | `ConfigService`: loads the asset, degrades to fallback, checks version |

Every app MUST use exactly these paths and these class names (`AppConfig`, `ConfigService`).

### 1.2 The JSON file

`assets/config/app_config.json` — example for an app that declares English (`en`) and Spanish
(`es`). Use one entry per **declared language** from `docs/PROJECT_PROFILE.md`:

```json
{
  "appName": {
    "en": "<Your App>",
    "es": "<Tu aplicación>"
  },
  "description": {
    "en": "One-line description of what the app does.",
    "es": "Descripción de una línea de lo que hace la aplicación."
  },
  "version": "1.0.0",
  "build": "1",
  "details": {
    "author": {
      "en": "<Author or company name>",
      "es": "<Author or company name>"
    },
    "email": "<support contact address>",
    "website": "https://<your-domain>/",
    "privacyPolicy": "https://<your-domain>/privacy",
    "license": {
      "en": "All libraries used are open source.",
      "es": "Todas las bibliotecas utilizadas son de código abierto."
    }
  }
}
```

- `appName`, `description`, `version`, `build` are required top-level fields.
- `details` is a free map. Add or remove rows as needed; the About screen renders each
  entry as a labelled row. Common optional rows: `author`, `email`, `website`, `privacyPolicy`,
  `license`, `aiUsed`, `ideUsed`. Use only what the project profile asks for.
- **Only technical, non-display values** that are never shown as user-facing text — such as
  email addresses, URLs, version strings, and build numbers — MAY be plain strings.
- **`appName` and every display row** (author, license, and any other text a user reads) MUST use
  the full locale map with **one entry per declared language**. Names of people or companies MAY
  repeat the same spelling in each entry, or use a transliteration where the script differs.
- Sentences, prose, and anything a user reads as descriptive text MUST use the locale map
  with every declared language filled in (§3, engineering standard §8).
- **Detail keys are identifiers, not labels.** Use `lowerCamelCase` keys (`author`,
  `privacyPolicy`); the visible label comes from ARB key `aboutDetail<Key>` (`aboutDetailAuthor`,
  `aboutDetailPrivacyPolicy`) so the label itself is translated. See §1.6.
- Keep `version` and `build` in sync with `pubspec.yaml`. `ConfigService` will log a
  non-fatal debug note if they drift (see §1.5).
- **No secrets** in this file — it ships inside the app and anyone can read it.

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
  `Map<String, LocalizedText> details`.
- A `LocalizedText` value type that holds either one string for all languages or one string
  per locale, and resolves against the active locale with the template language (usually English)
  as the last fallback.
- A `static const AppConfig fallback` with safe built-in values, so a missing or
  malformed config never crashes the app.
- A `factory AppConfig.fromJson(Map<String, dynamic> json)` that reads field by field and
  falls back per field on a missing value or wrong type (never throws).

Reference implementation:

```dart
/// A config text that may be language-independent (one string) or translated
/// (one string per locale code). Resolution order: exact locale → 'en' → any.
/// If the project's template language is not English, change 'en' below.
class LocalizedText {
  final Map<String, String> _byLocale;
  final String _plain;

  const LocalizedText.plain(String value)
      : _plain = value,
        _byLocale = const {};

  const LocalizedText.byLocale(Map<String, String> byLocale)
      : _plain = '',
        _byLocale = byLocale;

  bool get isEmpty => _plain.trim().isEmpty && _byLocale.isEmpty;

  /// [languageCode] is the active app language, e.g. 'en'.
  String resolve(String languageCode) {
    if (_byLocale.isEmpty) return _plain;
    return _byLocale[languageCode] ??
        _byLocale['en'] ??
        (_byLocale.values.isEmpty ? '' : _byLocale.values.first);
  }

  /// Accepts a JSON string or a {"<lang>": ..., ...} map.
  factory LocalizedText.fromJson(Object? raw, {String fallback = ''}) {
    if (raw is String) return LocalizedText.plain(raw);
    if (raw is Map) {
      final out = <String, String>{};
      raw.forEach((k, v) {
        if (k is String && v is String) out[k] = v;
      });
      if (out.isNotEmpty) return LocalizedText.byLocale(out);
    }
    return LocalizedText.plain(fallback);
  }
}

/// Typed values for the About screen, loaded from `assets/config/app_config.json`.
/// Changing About content is a config edit, not a code change.
class AppConfig {
  final LocalizedText appName;
  final LocalizedText description;
  final String version;
  final String build;
  final Map<String, LocalizedText> details;

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
    appName: LocalizedText.plain('My App'),
    description: LocalizedText.plain('A Flutter application.'),
    version: '0.0.0',
    build: '0',
    details: {},
  );

  factory AppConfig.fromJson(Map<String, dynamic> json) {
    String str(String key, String fallbackValue) {
      final value = json[key];
      return value is String ? value : fallbackValue;
    }

    Map<String, LocalizedText> parseDetails(String key) {
      final raw = json[key];
      if (raw is! Map) return const {};
      final out = <String, LocalizedText>{};
      raw.forEach((k, v) {
        if (k is String) {
          final text = LocalizedText.fromJson(v);
          if (!text.isEmpty) out[k] = text;
        }
      });
      return out;
    }

    return AppConfig(
      appName: LocalizedText.fromJson(
        json['appName'],
        fallback: fallback.appName.resolve('en'),
      ),
      description: LocalizedText.fromJson(
        json['description'],
        fallback: fallback.description.resolve('en'),
      ),
      version: str('version', fallback.version),
      build: str('build', fallback.build),
      details: parseDetails('details'),
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
- Skip any entry whose key or resolved value is empty after trimming.
- **The row label MUST be localized.** Look the label up in ARB by the key
  `aboutDetail<Key>` (e.g. `author` → `aboutDetailAuthor`). The raw JSON key is a
  last-resort fallback only, so a newly added key still renders.
- **The row value MUST be resolved against the active language** via
  `LocalizedText.resolve(languageCode)`.
- Optional nicety: if a key equals `email` (case-insensitive), make its row tappable to
  open `mailto:<value>`; if the value is an `https://` URL (`website`, `privacyPolicy`), open it
  in the browser.
- **Store apps SHOULD show a privacy-policy row.** Several stores expect the policy to be reachable
  from inside the app as well as from the listing (`platform_store_readiness.md`).

Reference snippet (from the reference About screen):

```dart
final lang = Localizations.localeOf(context).languageCode;
final l10n = AppLocalizations.of(context);

// ...
for (final entry in config.details.entries)
  if (entry.key.trim().isNotEmpty && entry.value.resolve(lang).trim().isNotEmpty)
    ListTile(
      title: Text(aboutDetailLabel(l10n, entry.key)),
      subtitle: Text(entry.value.resolve(lang)),
      onTap: entry.key.trim().toLowerCase() == 'email'
          ? () => _openMail(entry.value.resolve(lang))
          : null,
    ),
```

```dart
/// Maps a config detail key to its translated label, falling back to the raw key.
String aboutDetailLabel(AppLocalizations l10n, String key) {
  switch (key) {
    case 'author':
      return l10n.aboutDetailAuthor;
    case 'email':
      return l10n.aboutDetailEmail;
    case 'license':
      return l10n.aboutDetailLicense;
    case 'website':
      return l10n.aboutDetailWebsite;
    case 'privacyPolicy':
      return l10n.aboutDetailPrivacyPolicy;
    case 'aiUsed':
      return l10n.aboutDetailAiUsed;
    case 'ideUsed':
      return l10n.aboutDetailIdeUsed;
    default:
      return key;
  }
}
```

The fixed top rows (`appName` + `description`, and `version`/`build`) MAY be rendered
explicitly; everything else comes from the `details` loop. Their labels ("Version",
"Build") come from ARB like every other label — never a literal.

### 1.7 About screen — optional signature badge

A project MAY end its About screen with a short **signature badge** — for example
"Made with ❤️ by <Team>" or "Made with ❤️ from <Place>". It is **off by default**. The project
turns it on, and fixes its exact wording, in the About section of `docs/PROJECT_PROFILE.md`.

When the badge is enabled, these rules apply:

- **Identical in every app that enables it** for the same owner. It does not come from
  `app_config.json`, and once set it MUST NOT be removed, reworded, or replaced per screen or per
  release without changing the project profile first.
- **It is the last element** on the About screen, after the `details` rows, with vertical
  breathing room above it (≥ 24 dp) and safe-area padding below, horizontally centered.
- **The heart is red** (`Color(0xFFE53935)` or the theme's error/red accent); the
  surrounding words use the theme's muted foreground
  (`Theme.of(context).colorScheme.onSurfaceVariant`) at `bodySmall`/`labelMedium` size.
- **The words are localized, the heart is not.** The text comes from ARB key
  `madeWithLove`, which contains the `{heart}` placeholder so the same widget paints the heart
  red in every declared language.
- **It is screen-reader friendly**: wrap it in `Semantics(label: l10n.madeWithLoveA11y)`
  so a screen reader announces words, not an emoji name.
- The badge is **not** a link and has no tap action.

ARB entries (one pair per declared language — see engineering standard §8):

```json
// lib/l10n/app_en.arb
"madeWithLove": "Made with {heart} by <Team>",
"@madeWithLove": {
  "description": "About-screen signature badge. {heart} is a red heart glyph.",
  "placeholders": { "heart": { "type": "String" } }
},
"madeWithLoveA11y": "Made with love by <Team>",
"@madeWithLoveA11y": { "description": "Screen-reader text for the About badge" }
```

```json
// lib/l10n/app_<code>.arb — the same two keys, translated
"madeWithLove": "<translated text with {heart} where the heart goes>",
"madeWithLoveA11y": "<translated text without the heart>"
```

The `{heart}` placeholder MAY sit anywhere in the string (start, middle or end) — word order
differs between languages, and the reference widget below handles every position.

A worked example of an enabled badge in three languages is in
`profiles/example_en_ml_sa_profile.md`.

Reference widget — `lib/widgets/made_with_love.dart`:

```dart
/// The optional signature badge shown at the bottom of the About screen
/// when the project profile enables it. Text is localized; the heart is always red.
class MadeWithLove extends StatelessWidget {
  const MadeWithLove({super.key});

  static const Color _heartColor = Color(0xFFE53935);

  @override
  Widget build(BuildContext context) {
    final l10n = AppLocalizations.of(context);
    final theme = Theme.of(context);
    final base = theme.textTheme.bodySmall!
        .copyWith(color: theme.colorScheme.onSurfaceVariant);

    // Split the localized template around the {heart} placeholder so the
    // heart keeps its own colour in every language. A sentinel that can never
    // appear in translated text makes the split exact.
    const sentinel = '\u0000';
    final parts = l10n.madeWithLove(sentinel).split(sentinel);

    return Semantics(
      label: l10n.madeWithLoveA11y,
      excludeSemantics: true,
      child: Padding(
        padding: const EdgeInsets.symmetric(vertical: 24),
        child: Center(
          child: Text.rich(
            TextSpan(
              children: [
                TextSpan(text: parts.first, style: base),
                WidgetSpan(
                  alignment: PlaceholderAlignment.middle,
                  child: Icon(
                    Icons.favorite,
                    size: (base.fontSize ?? 12) * 1.1,
                    color: _heartColor,
                  ),
                ),
                TextSpan(text: parts.length > 1 ? parts[1] : '', style: base),
              ],
            ),
            textAlign: TextAlign.center,
          ),
        ),
      ),
    );
  }
}
```

> **Why `WidgetSpan` and not a character.** Rendering the heart as an emoji or text glyph
> (`❤`) lets OEM system fonts (e.g. on Samsung or Xiaomi devices) override the character with a
> platform-specific emoji glyph, ignoring the text colour. `WidgetSpan` with `Icon(Icons.favorite)`
> guarantees an exact vector heart in `#E53935` red on every screen and OS version.


> **Note on other constants.** This JSON pattern is only for **About-screen** metadata.
> Technical constants (database names, preference keys, thresholds, channel IDs) do NOT
> go in the JSON. Keep those in a plain Dart file such as
> `lib/core/constants/app_constants.dart` (class `AppConstants`, values only, no logic).

---

## 2. Android release keystore + `key.properties`

Every app that ships a signed Android release MUST follow this. Other platforms: see §2.5.

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

### 2.4 Production release build commands (Android)

Every production build MUST be built using `--release`, `--obfuscate`, and `--split-debug-info`. Omitting any of these flags produces an unhardened, easily reverse-engineered artifact.

#### Why these flags are required to secure the APK:
1. **`--release`**: Enables AOT compiler optimizations, strips assertions and debug logic, and ensures `android:debuggable` is set to `false`.
2. **`--obfuscate`**: Scrambles Dart class, method, and field identifiers into meaningless symbols within `libapp.so`, preventing trivial static analysis and decompilation of proprietary app logic.
3. **`--split-debug-info=<path>`**: Strips and extracts the symbol table into a separate directory outside the APK. **Mandatory** when `--obfuscate` is enabled; without archived symbol files, crash stack traces from production cannot be decoded.
4. **`--split-per-abi`** (for APKs): Builds separate native binaries per CPU architecture (`arm64-v8a`, `armeabi-v7a`, `x86_64`) rather than an oversized "fat" universal APK.
5. **`appbundle`** (for Google Play): Builds an Android App Bundle (`.aab`), allowing Google Play to serve device-optimized APKs and leverage Google Play App Signing and Play Integrity protection.

#### Ready-to-use production build examples

##### Example 1: Standard App (No Flavors) — Split APKs (Direct Distribution / Sideloading)

**Bash / macOS / Linux:**
```bash
flutter build apk \
  --release \
  --obfuscate \
  --split-debug-info=build/symbols/android-release/ \
  --split-per-abi
```

**PowerShell (Windows):**
```powershell
flutter build apk `
  --release `
  --obfuscate `
  --split-debug-info=build/symbols/android-release/ `
  --split-per-abi
```
*Output artifacts:* `build/app/outputs/flutter-apk/app-arm64-v8a-release.apk`, `app-armeabi-v7a-release.apk`, etc.

##### Example 2: Standard App (No Flavors) — App Bundle (Google Play Store)

**Bash / macOS / Linux:**
```bash
flutter build appbundle \
  --release \
  --obfuscate \
  --split-debug-info=build/symbols/android-release/
```

**PowerShell (Windows):**
```powershell
flutter build appbundle `
  --release `
  --obfuscate `
  --split-debug-info=build/symbols/android-release/
```
*Output artifact:* `build/app/outputs/bundle/release/app-release.aab`

##### Example 3: Multi-Flavor App — Production Split APKs

**Bash / macOS / Linux:**
```bash
flutter build apk \
  --flavor prod \
  --release \
  --obfuscate \
  --split-debug-info=build/symbols/android-prod/ \
  --split-per-abi
```

**PowerShell (Windows):**
```powershell
flutter build apk `
  --flavor prod `
  --release `
  --obfuscate `
  --split-debug-info=build/symbols/android-prod/ `
  --split-per-abi
```
*Output artifacts:* `build/app/outputs/apk/prod/release/app-arm64-v8a-prod-release.apk`, etc.

##### Example 4: Multi-Flavor App — Production App Bundle (Google Play Store)

**Bash / macOS / Linux:**
```bash
flutter build appbundle \
  --flavor prod \
  --release \
  --obfuscate \
  --split-debug-info=build/symbols/android-prod/
```

**PowerShell (Windows):**
```powershell
flutter build appbundle `
  --flavor prod `
  --release `
  --obfuscate `
  --split-debug-info=build/symbols/android-prod/
```
*Output artifact:* `build/app/outputs/bundle/prodRelease/app-prod-release.aab`

> **Symbol Archive Reminder**: Always archive `build/symbols/` immediately after every production build to a secure backup. Without it, production crash stack traces are permanently unreadable.

### 2.5 Signing on the other platforms

This section (§2) covers Android only. Signing for the other declared platforms follows the same
two rules — **signing material never enters the repository**, and **every secret has an offline
backup** — and is described where the platform is set up:

| Platform | Signing material | Where it is described |
|---|---|---|
| iOS | Apple Distribution certificate + provisioning profile | engineering standard §5.4, `platform_store_readiness.md` (App Store) |
| macOS | Apple Distribution (Mac App Store) or Developer ID Application certificate (direct download) + notarization | engineering standard §5.5.2, `platform_store_readiness.md` (macOS) |
| Windows | Microsoft Store signs Store packages; a code-signing certificate for direct download | engineering standard §5.5.1, `platform_store_readiness.md` (Windows) |
| Linux | Snap Store / Flathub sign on their side; optional GPG signature for direct packages | engineering standard §5.5.3, `platform_store_readiness.md` (Linux) |

Certificates (`*.p12`, `*.pfx`), App Store Connect API keys (`*.p8`) and provisioning profiles MUST
be git-ignored (engineering standard §20.4).

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
  l10n/              # ARB files — app_<code>.arb, one per declared language. REQUIRED.
```

Rules:

- Root `CLAUDE.md` MUST exist at the project root and follow [CLAUDE_MD_GUIDELINE.md](CLAUDE_MD_GUIDELINE.md).
- Root `AGENTS.md` MUST exist at the project root and follow [AGENTS_MD_GUIDELINE.md](AGENTS_MD_GUIDELINE.md).
- `docs/PROJECT_PROFILE.md` MUST exist, filled in from
  [PROJECT_PROFILE_TEMPLATE.md](PROJECT_PROFILE_TEMPLATE.md). It declares the platforms, stores and
  languages every other rule depends on.
- `plans/` and `change_log/` files MUST use relative repository paths only and MUST NOT contain
  **local system details** (OS user name, computer/host name, home or drive-letter paths, network
  share names, LAN/internal IPs, local server URLs with ports, device serial numbers, personal
  email addresses) or any secret (keys, tokens, passwords, keystore passphrases, credentials, PII).
  These files are committed and may become public — write them as if a stranger will read them.
  Full rule and bad → good examples:
  [flutter_project_engineering_standard.md §21.1.1](flutter_project_engineering_standard.md).
- `l10n.yaml` MUST exist at the project root, and **one ARB file per declared language** MUST
  exist in `lib/l10n/`, with the template language (usually `app_en.arb`) as the template. All
  user-visible text MUST come from `AppLocalizations`, never a raw string literal in a widget —
  even in a single-language app. Full rules:
  [flutter_project_engineering_standard.md §8](flutter_project_engineering_standard.md).
- **Key parity is mandatory**: every ARB key exists in every declared language's file with a real
  translation. A feature is not done until its strings exist in every declared language (§8.7).
- **Language packs apply when declared.** If the project declares a language that has a pack in
  `language_packs/` (for example Sanskrit or Malayalam), that pack's rules are mandatory.
- **The app language is user-selectable** when two or more languages are declared. The default
  is the system locale (falling back to the template language when it is not declared); a
  language picker in Settings MUST let the user override it, and the choice MUST persist across
  restarts and apply immediately (§8.4).
- **Menu, label, button and tab text MUST be short** in every declared language (§8.6). Only
  descriptive text (help, empty-state explanations, About description, error detail) may be long.
- **Every icon-only control MUST have a localized tooltip** — `IconButton`, `FloatingActionButton`,
  `PopupMenuButton`, icon-only gestures, and icon-only navigation destinations (§7.8).
- `core/config/` MUST exist and hold `AppConfig` + `ConfigService` exactly as in §1.
- The About screen lives under `screens/` (e.g. `screens/about_screen.dart`) and reads its
  values from `ConfigService` / `AppConfig` — it MUST NOT hard-code About text. If the project
  profile enables the signature badge, the screen MUST end with it (§1.7).
- Apps that ship to users MUST pass the readiness gate of **every declared distribution channel**
  in [platform_store_readiness.md](platform_store_readiness.md) before the first upload.
- Pick **one** state-management folder name per app (`providers/` **or** `state/`) and use
  it consistently.
- Larger apps MAY introduce a layered or feature-first structure (e.g. `domain/`, `data/`,
  `application/`, `presentation/`, or feature folders). When they do, the `core/config/`
  About pattern and the `android/` keystore rules still apply unchanged.

---

## 4. Quick checklist for a new (or migrated) app

**Every app**

- [ ] Root `CLAUDE.md` exists at project root and follows [CLAUDE_MD_GUIDELINE.md](CLAUDE_MD_GUIDELINE.md) (**MUST**).
- [ ] Root `AGENTS.md` exists at project root and follows [AGENTS_MD_GUIDELINE.md](AGENTS_MD_GUIDELINE.md) (**MUST**).
- [ ] `docs/PROJECT_PROFILE.md` exists and is complete: identity, platforms, stores, languages,
      About options (**MUST**).
- [ ] Platform folders were created with `flutter create --platforms=...` for the declared
      platforms only (**MUST**, engineering standard §3.3).
- [ ] `plans/` and `change_log/` files use relative repository paths only and contain zero local
      system details and zero sensitive data — safe to publish on the internet (**MUST**).
- [ ] `l10n.yaml` exists at the project root and one ARB file exists per declared language (**MUST**).
- [ ] ARB key parity holds — every key is present and translated in every declared language (**MUST**).
- [ ] `supportedLocales` equals the declared languages; a fallback delegate is installed for any
      declared language Flutter has no framework translation for (**MUST**, engineering standard §8.3.1).
- [ ] With two or more languages: Settings has a language picker (System default + each language
      by its endonym) that persists and applies without a restart (**MUST**, §8.4).
- [ ] With two or more languages and Android declared: `android/app/build.gradle.kts` disables
      language splitting (`bundle.language.enableSplit = false`) (**MUST**, engineering standard §8.1).
- [ ] Every applicable language pack's checklist passes (**MUST**, §8.5).
- [ ] Toolchain is the latest stable Flutter with the AGP / KGP / Gradle versions it generates
      (Java 17 minimum), pinned in the project's own files — not copied from a guide or typed
      from memory; `android/app/build.gradle.kts` uses `kotlin { compilerOptions { } }`, not
      `kotlinOptions` (**MUST**, engineering standard §5.3).
- [ ] Material and Cupertino come from `material_ui` / `cupertino_ui`, pinned with `^`; no
      `package:flutter/material.dart` or `package:flutter/cupertino.dart` imports remain
      (**MUST**, engineering standard §6.1).
- [ ] Menu / label / button / tab text is within the length budget in every declared language
      (**MUST**, §8.6).
- [ ] Every icon-only control has a localized tooltip (**MUST**, §7.8).
- [ ] No hard-coded user-visible strings — all screen text comes from `AppLocalizations` (**MUST**).
- [ ] `assets/config/app_config.json` exists with `appName`, `description`, `version`,
      `build`, `details`; every display value has an entry per declared language (§1.2).
- [ ] `assets/config/` registered under `flutter: assets:` in `pubspec.yaml`.
- [ ] `lib/core/config/app_config.dart` defines `AppConfig` with `fromJson` + `fallback`.
- [ ] `lib/core/config/config_service.dart` defines `ConfigService` with `load()` +
      `loadAndVerify()`.
- [ ] About screen reads from `ConfigService`, not hard-coded strings.
- [ ] About screen renders `details` dynamically (loops the map, no hard-coded field names), with
      localized labels (`aboutDetail<Key>`) and locale-resolved values (§1.6).
- [ ] If the profile enables the signature badge: About screen ends with it — red heart,
      centered, localized words (§1.7).
- [ ] App icon generated for every declared platform; no default Flutter icon remains
      (engineering standard §17.5).
- [ ] `lib/` follows the baseline layout in §3 (subset is fine for small apps).

**Shipped apps — per declared platform**

- [ ] Android: release keystore at `android/<name>.jks`; `android/key.properties` points to it;
      both git-ignored (§2).
- [ ] Android: production builds run with `--release`, `--obfuscate`, and `--split-debug-info` (§2.4).
- [ ] iOS / macOS / Windows / Linux: signing set up as in §2.5, with no signing material in git.
- [ ] Every release build uses `--obfuscate` and `--split-debug-info`; debug symbols archived
      securely alongside the release artifacts.
- [ ] CI builds a release artifact for every declared platform (engineering standard §19.2).
- [ ] The readiness gate of every declared distribution channel passed before the first upload
      ([platform_store_readiness.md](platform_store_readiness.md)) (**MUST** for any app shipped
      to users).
