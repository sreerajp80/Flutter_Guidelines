# Example Project Profile — English / Malayalam / Sanskrit app with a signature badge

> **This is a worked example, not a rule.** It shows a filled-in `docs/PROJECT_PROFILE.md` for an
> app that ships English, Malayalam and Sanskrit, enables the "Made with ❤️ from India" signature
> badge, and publishes to Google Play and the Microsoft Store. Copy
> `PROJECT_PROFILE_TEMPLATE.md` for your own app; reuse parts of this file only if they match your
> choices.

---

## 1. Identity

| Item | Value |
|---|---|
| App display name | `Example PDF App` |
| Short description (one line) | `Read and annotate PDF files offline.` |
| Organization / publisher | `<Publisher name>` |
| Reverse-DNS app id | `com.example.pdfapp` |
| Dart package name | `pdf_app` |
| Public support contact | `<public support address>` |
| Website | `none` |
| Privacy policy URL | `https://<your-domain>/pdfapp/privacy` |
| Copyright line | `© 2026 <Publisher name>` |
| License of the app's own code | `proprietary` |

## 2. Applicability profiles

- [x] `Core Baseline`
- [x] `Production App Extension`
- [ ] `Sensitive Data Extension`

## 3. Target platforms and distribution

| Platform | Ship? | Minimum OS | Distribution channels |
|---|---|---|---|
| Android | yes | `minSdk 24` (covers the target users' devices) | Google Play, split APKs for direct download |
| iOS | no | — | — |
| Windows | yes | Windows 10 1809 | Microsoft Store |
| macOS | no | — | — |
| Linux | no | — | — |
| Web | no | — | — |

Build flavors: `dev + prod`.

## 4. Languages

| Locale | Language | Script | Role | Language pack |
|---|---|---|---|---|
| `en` | English | Latin | Template ARB, ultimate fallback | — |
| `ml` | Malayalam | Malayalam | Full UI translation | `language_packs/sanskrit_malayalam.md` |
| `sa` | Sanskrit | Devanagari | Full UI translation | `language_packs/sanskrit_malayalam.md` |

- In-app language picker: required (3 languages). Endonyms: `English`, `മലയാളം`, `संस्कृतम्`.
- Languages without a Flutter framework translation: `sa` (fallback delegate, standard §8.3.1).
- Store-listing languages: Google Play `en-US` and `ml-IN`. Sanskrit is not a store listing
  language; it ships inside the app only.
- Native-reader reviewer: a fluent Malayalam reader and a fluent Sanskrit reader before each release.

## 5. About screen

| Item | Value |
|---|---|
| `details` rows to show | `author`, `email`, `license`, `privacyPolicy`, `aiUsed`, `ideUsed` |
| Signature badge | **on** |
| Badge text — English | `Made with {heart} from India` |
| Badge text — Malayalam | `സ്നേഹത്തോടെ {heart} ഇന്ത്യയിൽ നിന്ന്` |
| Badge text — Sanskrit | `सस्नेहं निर्मितम् {heart} भारततः` |
| Badge screen-reader text | `Made with love from India` / `സ്നേഹത്തോടെ ഇന്ത്യയിൽ നിന്ന്` / `सस्नेहं निर्मितम् भारततः` |

Example `assets/config/app_config.json` for this profile:

```json
{
  "appName": {
    "en": "Example PDF App",
    "ml": "എക്സാമ്പിൾ പിഡിഎഫ് ആപ്പ്",
    "sa": "उदाहरण-पीडीएफ्-अनुप्रयोगः"
  },
  "description": {
    "en": "One-line description of what the app does.",
    "ml": "ആപ്പ് എന്തു ചെയ്യുന്നു എന്നതിന്റെ ഒറ്റവരി വിവരണം.",
    "sa": "एषः अनुप्रयोगः किं करोति इति एकपङ्क्तिवर्णनम्।"
  },
  "version": "1.0.0",
  "build": "1",
  "details": {
    "author": {
      "en": "<Publisher name>",
      "ml": "<Publisher name in Malayalam script>",
      "sa": "<Publisher name in Devanagari>"
    },
    "email": "<public support address>",
    "license": {
      "en": "All libraries used are open source.",
      "ml": "ഉപയോഗിച്ച എല്ലാ ലൈബ്രറികളും ഓപ്പൺ സോഴ്സ് ആണ്.",
      "sa": "सर्वे प्रयुक्ताः पुस्तकालयाः मुक्तस्रोतसः सन्ति।"
    },
    "privacyPolicy": "https://<your-domain>/pdfapp/privacy",
    "aiUsed": {
      "en": "Anthropic Claude",
      "ml": "ആന്ത്രോപിക് ക്ലോഡ്",
      "sa": "आन्त्रोपिक् क्लोड्"
    },
    "ideUsed": {
      "en": "Visual Studio Code",
      "ml": "വിഷ്വൽ സ്റ്റുഡിയോ കോഡ്",
      "sa": "विश्वल् स्टुडियो कोड्"
    }
  }
}
```

## 6. Optional project-specific values

| Item | Value |
|---|---|
| App size budget override | default |
| State management | Riverpod |
| Structure tier | Tier 1 layer-first |
| Release owner (role) | app maintainer |
