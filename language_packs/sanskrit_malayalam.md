# Language Pack — Sanskrit (`sa`) and Malayalam (`ml`)

**Applies only when** a project declares `sa` and/or `ml` in its `docs/PROJECT_PROFILE.md`
(languages row). Projects that do not ship these languages ignore this file.

This pack adds language-specific rules on top of the generic localization rules in
`flutter_project_engineering_standard.md` §8. Where this pack and the standard both speak, both
apply; this pack is stricter for these two languages.

Conformance words: **MUST** = required, **SHOULD** = expected default, **MAY** = optional.

| Section | What it covers |
|---|---|
| 1 | Framework gaps for Sanskrit (delegate, `intl`) |
| 2 | Fonts and script coverage for Malayalam and Devanagari |
| 3 | Language-picker labels |
| 4 | Sanskrit & Malayalam quality — rules, Hindi-marker gate, glossary |
| 5 | Short-label budget for these languages |
| 6 | Store listings |
| 7 | Checklist |

---

## 1. Framework gaps for Sanskrit

### 1.1 No Flutter framework translation for `sa`

Flutter's Material / Cupertino / Widgets localizations do not include `sa`. The generic
**fallback delegate** in the engineering standard §8.3.1 handles this: list `sa` in the
fallback set so framework strings load from English. Use **English, never Hindi**, as the
fallback so no Hindi text can leak into a Sanskrit UI.

A widget test MUST cover this: pump the app with `locale: Locale('sa')`, open a date picker and a
dialog, and assert no exception. This is the single most likely Sanskrit runtime failure.

### 1.2 `intl` formatting under Sanskrit

`intl` has no `sa` date or number symbols, so `DateFormat.yMMMMd('sa')` throws. Always format with
the `formattingLocale(...)` helper from standard §8.3.2, which falls back to English. Do not fall
back to `hi`.

## 2. Fonts and script coverage

Malayalam and Devanagari glyphs are not guaranteed on every Android device, on Windows, or on
Linux, and a missing glyph renders as a blank box — a silent, ship-blocking bug.

- The app MUST either bundle fonts covering both scripts (e.g. Noto Sans Malayalam and Noto Sans
  Devanagari) or declare an explicit `fontFamilyFallback` chain and verify rendering on a clean
  device image for each declared platform.
- Verification is per release: open every screen in `ml` and in `sa` on a clean device and confirm
  no boxes, no clipped ascenders/descenders (Malayalam and Devanagari are taller than Latin), and
  no overflow.

## 3. Language-picker labels

The picker lists each language by its endonym (standard §8.4):

| Option | Shown as |
|---|---|
| System default | localized label, e.g. "System default" / "സിസ്റ്റം സ്വതവേ" / "तन्त्रसिद्धम्" |
| English | `English` |
| Malayalam | `മലയാളം` |
| Sanskrit | `संस्कृतम्` |

## 4. Sanskrit & Malayalam Quality — Standard UI Glossary

Apps that declare Malayalam and/or Sanskrit need deliberate linguistic care for both. The two
common pitfalls are Hindi leakage into Sanskrit (the scripts are shared — Devanagari) and
awkward English transliterations or calques in Malayalam.

### 4.1 Sanskrit Quality Rules (Pure Sanskrit, Never Hindi)

Sanskrit's derivational system — verbal roots (`धातु`), prefixes (`उपसर्ग`), suffixes
(`कृत्` / `तद्धित प्रत्यय`), and compounds (`समास`) — can derive a term for any UI concept.

Because Sanskrit and Hindi share the Devanagari script, Hindi text *looks* like Sanskrit to anyone
who does not read it. Never use Hindi anywhere as a substitute, crutch, or fallback for Sanskrit.
`app_sa.arb` MUST be authentic, uncompromised Sanskrit.

- **Classical vocabulary and grammar**: Use authentic Sanskrit nominal stems, proper case endings,
  and correct verbal forms (e.g. polite passive imperative `परिवर्त्यताम्`, not Hindi `बदलें`).
- **No transliterated English loans**: Never transliterate English words into Devanagari when a
  standard Sanskrit word exists (`सेटिंग्स` is Hindi/English in Devanagari; use `विन्यासः`).
- **No Hindi function words or syntax**: Do not use Hindi postpositions (`का`, `की`, `के`, `को`,
  `में`, `से`, `पर`), copulas (`है`, `हैं`, `था`, `थे`, `थी`, `हूं`), or verb endings (`करें`,
  `करना`, `रहा`, `गया`, `चाहिए`).
- **No nukta consonants**: The Perso-Arabic consonants with nukta (`क़`, `ख़`, `ग़`, `ज़`, `ड़`, `ढ़`,
  `फ़`) do not occur in Sanskrit.
- **Strict grammatical agreement**: Participles and adjectives must agree with their subject in
  gender and case. In "No data found", `दत्तांशः` is masculine nominative, so the participle must be
  `प्राप्तः` and the indefinite pronoun `कोऽपि`: `न कोऽपि दत्तांशः प्राप्तः` (never neuter `न किमपि दत्तांशं प्राप्तम्`).
- **Valid morphological derivation**:
  - Do not invent verbs by slapping verbal endings onto nouns. "Copy" is `प्रतिलिख्यताम्` (from verb
    root `लिख्` with `प्रति`) or `प्रतिलिपिः क्रियताम्`, never pseudo-verb `प्रतिलिप्यताम्`.
  - The past passive participle for "Copied" is `प्रतिलिखितम्` (or `प्रतिलिपीकृतम्`), never `प्रतिलिपितम्`.
  - Causative passive of `या` (go) is `निर्याप्यते` / `निर्याप्यताम्` (Export), never `निर्यात्यताम्`.
  - "Confirm" is `स्थिरीक्रियताम्` or `दृढीक्रियताम्` (let it be made firm), never `संपुष्यताम्` (which means "let it be nourished").
  - Do not use Hindi loanwords for concepts that have native Sanskrit terms (use `उपयोक्तृविवरणम्` for Account, never Hindi `खाता`; `लेखा` strictly means a line/furrow).
  - Use `ध्वनिः` for audio/sound to avoid confusion with `शब्दः` (Word).
- **Form conventions**:
  - A button or menu item (action commanding the app): polite `-ताम्` imperative (`लोट्`). For a verb
    that takes an object it is passive (`कर्मणि`), e.g. `रक्ष्यताम्` (Save), `अन्विष्यताम्` (Search); for a
    verb that takes no object it is impersonal (`भावे`), e.g. `निष्क्रम्यताम्` (Exit).
  - A title, tab, label, heading, or status: nominal / abstract noun, e.g. `अन्वेषणम्` (Search), `विन्यासः` (Settings).
  - A confirmation or boolean response: indeclinable, e.g. `आम्` (Yes), `न` (No), `अस्तु` (OK).
  - Direction words (Back, Next, Previous, More) are nominal or adverbial labels and MAY keep that
    form on a button, like Yes / No. Close and Exit are actions: on a button they MUST use the
    imperative (`पिधीयताम्`, `निष्क्रम्यताम्`); the nominal form (`निष्क्रमणम्`) is for titles and labels.
- **Punctuation**: Use the **daṇḍa** `।` to end a sentence in descriptive prose; UI labels take no terminator.
- **Locale marker**: Always set `"@@locale": "sa"` at the top of `app_sa.arb`.
- **Pre-release review**: Machine translation tools commonly output Hindi for Sanskrit requests. All
  Sanskrit ARB entries MUST be reviewed before release.

**Forbidden markers.** None of these tokens may appear anywhere in `app_sa.arb`, `assets/config/app_config.json`,
or any `*_sa.*` asset file. They are reliable Hindi giveaways and make a good grep-based gate:

```text
है  हैं  था  थे  थी  हूं  हो  करें  करना  करके  रहा  रही  रहे  गया  गयी  चाहिए
नहीं  और  लेकिन  क्या  आपका  आपकी  आपके  हमारा  मेरा  कृपया  सेटिंग्स  ऐप
◌़ (nukta U+093C, and the precomposed nukta letters U+0958–U+095F)
```

```bash
# CI gate: fail the build if any Hindi marker appears in the Sanskrit ARB,
# app_config.json, or Sanskrit asset files.
# Uses PCRE (-P) with lookarounds so standalone copulas/words are not confused with
# legitimate Sanskrit roots (e.g. स्थाप्यताम्, स्थानम्) or indeclinables (यथा, तथा, कथा).
# Word edges: whitespace, quotes, brackets, punctuation, daṇḍa, XML/HTML tag edges (< >),
# and Markdown marks (* _ ` # | : ; ~ -) so help files in Markdown are checked too.
PATTERN='(?<=[\s"'\''([{<>।,*_`#|:;~-]|^)(?:था|थे|थी|हो|है|हैं|हूं|और)(?=[\s"'\''\)\]}<>।,.\?!*_`#|:;~-]|$)|करें|करना|करके|रहा|रही|रहे|गया|गयी|चाहिए|नहीं|लेकिन|क्या|कृपया|सेटिंग्स|ऐप|\x{093C}|[\x{0958}-\x{095F}]'

mapfile -t FILES < <(find . -path '*/build' -prune -o -type f \( \
    -name 'app_sa.arb' -o \
    -path '*/assets/*_sa.*' -o \
    -path '*/assets/config/app_config.json' \) -print)

if [ "${#FILES[@]}" -eq 0 ]; then
  echo 'No Sanskrit files found — the project declares sa but app_sa.arb is missing.'; exit 1
fi

# Self-test: the pattern must still catch a Hindi copula in ARB/JSON, XML and Markdown text.
for sample in '"greeting": "है"' '<b>है</b>' 'यह **है**' 'यह `है`' 'वह *था*'; do
  if ! printf '%s\n' "$sample" | LC_ALL=C.UTF-8 grep -qP "$PATTERN"; then
    echo "Sanskrit check self-test failed on: $sample"; exit 1
  fi
done

if LC_ALL=C.UTF-8 grep -nP "$PATTERN" "${FILES[@]}"; then
  echo 'Hindi markers found in Sanskrit text (language pack §4.1).'; exit 1
fi
exit 0
```

> The gate is a smoke test, not a proof of correctness: passing it means no obvious Hindi marker is
> present, not that the Sanskrit is good. Human review still applies.
> Standalone words (था, थे, थी, हो, है, हैं, हूं, और) match only between word edges: whitespace,
> quotes, brackets, punctuation (including the daṇḍa `।`), the tag edges `<` and `>`, and the
> Markdown marks `*`, `_`, `` ` ``, `#`, `|`, `:`, `;`, `~`, `-`. So `यह **है**` in a help file fails
> the build, while legitimate Sanskrit such as `स्थाप्यताम्`, `स्थानम्`, `पुनःस्थाप्यताम्`, `यथा`,
> `तथा` and `कथा` never does.


### 4.2 Malayalam Quality Rules (Natural Malayalam, Not English Transliterations)

Malayalam UI strings must sound natural and idiomatic to native Malayalam speakers.

- **Avoid lazy English transliterations; established loanwords allowed**: Do not phonetically
  transliterate English UI jargon into Malayalam script when standard, authentic Malayalam words exist.
  - Save: `സൂക്ഷിക്കുക` (never bare `സേവ്`).
  - Print: `അച്ചടിക്കുക` (never `പ്രിന്റ്`).
  - Vibration: `കമ്പനം` (never `വൈബ്രേഷൻ`).
  - Optional: `ഐച്ഛികം` (never `ഓപ്ഷണൽ`).
  - Number: `സംഖ്യ` (never `നമ്പർ`).
  - Page: `താൾ` (never `പേജ്`).
  - Widely established digital loanwords (such as `ഹോം`, `മെനു`, `പ്രൊഫൈൽ`, `അക്കൗണ്ട്`, `ഡൗൺലോഡ്`,
    `ഓഫ്‌ലൈൻ`, `തീം`, `ഫയൽ`, `ഫോൾഡർ`, `ലിങ്ക്`, `ലൈസൻസ്`) are accepted where no single native term
    carries universal recognition.
- **Action buttons use verb forms**: Action buttons commanding an operation MUST use the verbal
  form ending in `-ക്കുക` / `-ക` (`തിരുത്തുക`, `സൂക്ഷിക്കുക`, `നീക്കുക`, `തുറക്കുക`, `പുറത്തുകടക്കുക`,
  `ലോഗൗട്ട് ചെയ്യുക`), never a bare English noun or uninflected loan.
- **Accurate negation (`ഇല്ല` vs `അല്ല`)**:
  - `ഇല്ല` denotes non-existence, absence, or refusal to perform an action. For confirmation dialog
    action buttons (Yes / No), use **`അതെ` / `ഇല്ല`**.
  - `അല്ല` denotes negation of identity or qualification ("is not", e.g. `ശരിയല്ല`). Do not put
    `അല്ല` on a confirmation prompt's "No" button when the dialog asks if an action should be done.
- **Avoid ungrammatical standalone postpositions**: Postpositions like `കുറിച്ച്` govern an accusative
  noun (e.g. `ആപ്പിനെക്കുറിച്ച്`); standing alone as a screen title or heading, `കുറിച്ച്` is
  ungrammatical. Use `ആപ്പിനെക്കുറിച്ച്` for "About"; `വിവരണം` is reserved for Description.
- **Contextual accuracy over literal calques**:
  - Preferences: `താൽപ്പര്യങ്ങൾ` or `ഇഷ്ടങ്ങൾ` (matches Sanskrit `रुचयः`). `മുൻഗണനകൾ` strictly means
    **Priorities** (precedence/rank) and is a misleading false friend.
  - Apply (theme/filters): `പ്രയോഗിക്കുക` or `നടപ്പിലാക്കുക`. `ബാധകമാക്കുക` means legal liability/enforcement.
  - Sort: `ക്രമീകരിക്കുക` (arrange in order / sort sequence). `അടുക്കുക` means to stack or draw near.
- **Modern Unicode orthography**: Always use standard Unicode Malayalam atomic chillu characters (`ൺ`, `ൻ`, `ർ`, `ൽ`, `ൾ`). Avoid legacy ZWJ sequences or non-standard glyphs.

### 4.3 Bad → Good Translations

| English | Bad (Hindi / English loan / Calque) | Good (Sanskrit) | Good (Malayalam) | Linguistic Rationale |
|---|---|---|---|---|
| Settings | सेटिंग्स / സെറ്റിംഗ്സ് | विन्यासः | ക്രമീകരണങ്ങൾ | Standard native terminology |
| Save | सेव करें / സേവ് | रक्ष्यताम् | സൂക്ഷിക്കുക | Polite imperative in SA; `-ക്കുക` verb in ML |
| Delete | डिलीट करें / ഡിലീറ്റ് | लुप्यताम् / विलुप्यताम् | ഇല്ലാതാക്കുക | Authentic verbal action |
| Cancel | कैंसिल / ക്യാൻസൽ | निरस्यताम् | റദ്ദാക്കുക | Native rejection/dismissal term |
| Copy | कॉपी करें / കോപ്പി | प्रतिलिख्यताम् | പകർത്തുക | `प्रति + लिख्` verb in SA; NOT `प्रतिलिप्यताम्` |
| Export | निर्यात करें / എക്സ്പോർട്ട് | निर्याप्यताम् | കയറ്റുമതി ചെയ്യുക | Correct causative passive of `या` in SA |
| Search | खोजें / സെർച്ച് | अन्वेषणम् (title) / अन्विष्यताम् (action) | തിരയുക | Distinct noun title vs. action button |
| No data found | कोई डेटा नहीं मिला / ഡാറ്റ ഇല്ല | न कोऽपि दत्तांशः प्राप्तः | വിവരങ്ങളൊന്നും കണ്ടെത്തിയില്ല | Gender agreement in SA (`दत्तांशः` masculine nom.) |
| Preferences | प्रेफरेंसेस / മുൻഗണനകൾ | रुचयः | താൽപ്പര്യങ്ങൾ / ഇഷ്ടങ്ങൾ | `മുൻഗണനകൾ` means priorities, not preferences |
| Confirm | संपुष्यताम् / കൺഫേം | स्थिरीक्रियताम् / दृढीक्रियताम् | സ്ഥിരീകരിക്കുക | `पुष्` means nourish; `स्थिरी` means confirm |
| Print | प्रिंट करें / പ്രിന്റ് | मुद्र्यताम् | അച്ചടിക്കുക | Standard Malayalam verb |
| About | ऐप के बारे में / കുറിച്ച് / परिचयः | विषयपरिचयः | ആപ്പിനെക്കുറിച്ച് | `കുറിച്ച്` is a bound postposition, not a title; `विषये` is locative ("regarding"). One term per meaning: `परिचयः` is Profile and `വിവരണം` is Description (§4.4). |

### 4.4 Standard UI Glossary

Use these exact terms across all apps that declare these languages. When a term you need is missing, add it
**here**, in this pack, rather than inventing inconsistent per-app variants. All short UI terms
MUST fit within the 22-character limit defined in §5 of this pack.

**Review rule.** A new or changed Malayalam or Sanskrit glossary term MUST be reviewed by a fluent
reader before any app uses it. The change that adds the term lists it in its change log as
"needs native-reader review" until that review is done.

#### Navigation and structure

| English | Malayalam | Sanskrit |
|---|---|---|
| Home | ഹോം | गृहम् |
| Back | പിന്നോട്ട് | प्रत्यागमनम् |
| Next | അടുത്തത് | अग्रिमम् |
| Previous | മുമ്പത്തേത് | पूर्वम् |
| Menu | മെനു | सूची |
| More | കൂടുതൽ | अधिकम् |
| Close | അടയ്ക്കുക | पिधीयताम् |
| Exit (button) | പുറത്തുകടക്കുക | निष्क्रम्यताम् |
| Exit (title, label) | പുറത്തുകടക്കൽ | निष्क्रमणम् |
| Profile | പ്രൊഫൈൽ | परिचयः |
| Notifications | അറിയിപ്പുകൾ | सूचनाः |
| Favorites | പ്രിയപ്പെട്ടവ | प्रियाणि |
| History | നാൾവഴി | इतिवृत्तम् |
| Details | വിശദാംശങ്ങൾ | विवरणम् |
| List | പട്ടിക | आवली |
| Category | വിഭാഗം | वर्गः |
| Page | താൾ | पृष्ठम् |
| Section | ഖണ്ഡം | खण्डः |

#### Actions (buttons, menu items)

| English | Malayalam | Sanskrit |
|---|---|---|
| Save | സൂക്ഷിക്കുക | रक्ष्यताम् |
| Cancel | റദ്ദാക്കുക | निरस्यताम् |
| Delete | ഇല്ലാതാക്കുക | लुप्यताम् |
| Edit | തിരുത്തുക | सम्पाद्यताम् |
| Add | ചേർക്കുക | योज्यताम् |
| Remove | നീക്കുക | अपनीयताम् |
| Create | സൃഷ്ടിക്കുക | सृज्यताम् |
| Update | നവീകരിക്കുക | अद्यतनीक्रियताम् |
| Copy | പകർത്തുക | प्रतिलिख्यताम् |
| Paste | ഒട്ടിക്കുക | स्थाप्यताम् |
| Undo | പഴയപടിയാക്കുക | प्रत्यावर्त्यताम् |
| Redo | വീണ്ടും ചെയ്യുക | पुनःक्रियताम् |
| Search | തിരയുക | अन्विष्यताम् |
| Filter | അരിക്കുക | परिशोध्यताम् |
| Sort | ക്രമീകരിക്കുക | क्रमीक्रियताम् |
| Refresh | പുതുക്കുക | नवीक्रियताम् |
| Share | പങ്കിടുക | वितीर्यताम् |
| Send | അയയ്ക്കുക | प्रेष्यताम् |
| Download | ഡൗൺലോഡ് ചെയ്യുക | अवतार्यताम् |
| Upload | അപ്‌ലോഡ് ചെയ്യുക | आरोप्यताम् |
| Import | ഇറക്കുമതി ചെയ്യുക | आनीयताम् |
| Export | കയറ്റുമതി ചെയ്യുക | निर्याप्यताम् |
| Print | അച്ചടിക്കുക | मुद्र्यताम् |
| Select | തിരഞ്ഞെടുക്കുക | चीयताम् |
| Select all | എല്ലാം തിരഞ്ഞെടുക്കുക | सर्वं चीयताम् |
| Clear | മായ്ക്കുക | रिक्तीक्रियताम् |
| Reset | പുനഃസജ്ജമാക്കുക | पुनःसज्जीक्रियताम् |
| Confirm | സ്ഥിരീകരിക്കുക | स्थिरीक्रियताम् |
| Apply | പ്രയോഗിക്കുക | प्रयुज्यताम् |
| Open | തുറക്കുക | उद्घाट्यताम् |
| Start | ആരംഭിക്കുക | आरभ्यताम् |
| Stop | നിർത്തുക | विरम्यताम् |
| Pause | നിർത്തിവയ്ക്കുക | स्थग्यताम् |
| Resume | പുനരാരംഭിക്കുക | पुनरारभ्यताम् |
| Continue | തുടരുക | अनुवर्त्यताम् |
| Skip | ഒഴിവാക്കുക | त्यज्यताम् |
| Retry | വീണ്ടും ശ്രമിക്കുക | पुनः प्रयत्यताम् |
| Login | പ്രവേശിക്കുക | प्रविश्यताम् |
| Logout | ലോഗൗട്ട് ചെയ്യുക | निर्गम्यताम् |

#### Settings and preferences

| English | Malayalam | Sanskrit |
|---|---|---|
| Settings | ക്രമീകരണങ്ങൾ | विन्यासः |
| Preferences | താൽപ്പര്യങ്ങൾ | रुचयः |
| Language | ഭാഷ | भाषा |
| Theme | തീം | रूपविन्यासः |
| Dark mode | ഇരുണ്ട രൂപം | श्यामरूपम् |
| Light mode | തെളിഞ്ഞ രൂപം | दीप्तरूपम् |
| System default | സിസ്റ്റം സ്വതവേ | तन्त्रसिद्धम् |
| Font size | അക്ഷരവലുപ്പം | अक्षरपरिमाणम् |
| Sound | ശബ്ദം | ध्वनिः |
| Vibration | കമ്പനം | कम्पनम् |
| Backup | കരുതൽശേഖരം | प्रतिलिपिरक्षणम् |
| Restore | പുനഃസ്ഥാപിക്കുക | पुनःस्थाप्यताम् |
| Permissions | അനുമതികൾ | अनुमतयः |
| Account | അക്കൗണ്ട് | उपयोक्तृविवरणम् |
| Privacy | സ്വകാര്യത | गोपनीयता |
| Security | സുരക്ഷ | सुरक्षा |
| Storage | സംഭരണം | सङ्ग्रहः |
| Data | വിവരങ്ങൾ | दत्तांशः |

#### Status, feedback, and empty states

| English | Malayalam | Sanskrit |
|---|---|---|
| Loading | ലോഡുചെയ്യുന്നു | आपूर्यते |
| Please wait | കാത്തിരിക്കുക | प्रतीक्ष्यताम् |
| Success | വിജയം | सफलम् |
| Failed | പരാജയപ്പെട്ടു | असफलम् |
| Error | പിശക് | दोषः |
| Warning | മുന്നറിയിപ്പ് | पूर्वसूचना |
| Information | വിവരം | सूचना |
| Done | പൂർത്തിയായി | समाप्तम् |
| Empty | ശൂന്യം | रिक्तम् |
| No results | ഫലങ്ങളില്ല | न किमपि प्राप्तम् |
| Offline | ഓഫ്‌ലൈൻ | असंयुक्तम् |
| Online | ഓൺലൈൻ | संयुक्तम् |
| Saved | സൂക്ഷിച്ചു | रक्षितम् |
| Deleted | ഇല്ലാതാക്കി | लुप्तम् |
| Copied | പകർത്തി | प्रतिलिखितम् |
| Updated | നവീകരിച്ചു | अद्यतनीकृतम् |
| Required | ആവശ്യം | आवश्यकम् |
| Optional | ഐച്ഛികം | वैकल्पिकम् |
| Invalid | അസാധു | अमान्यम् |

#### Time and date

| English | Malayalam | Sanskrit |
|---|---|---|
| Date | തീയതി | दिनाङ्कः |
| Time | സമയം | समयः |
| Today | ഇന്ന് | अद्य |
| Yesterday | ഇന്നലെ | ह्यः |
| Tomorrow | നാളെ | श्वः |
| Now | ഇപ്പോൾ | इदानीम् |
| Day | ദിവസം | दिनम् |
| Week | ആഴ്ച | सप्ताहः |
| Month | മാസം | मासः |
| Year | വർഷം | वर्षम् |
| Duration | ദൈർഘ്യം | कालावधिः |

#### Content and fields

| English | Malayalam | Sanskrit |
|---|---|---|
| Title | ശീർഷകം | शीर्षकम् |
| Name | പേര് | नाम |
| Description | വിവരണം | वर्णनम् |
| Note | കുറിപ്പ് | टिप्पणी |
| Text | പാഠം | पाठः |
| Image | ചിത്രം | चित्रम् |
| Audio | ഓഡിയോ | श्रव्यम् |
| Video | വീഡിയോ | दृश्यम् |
| File | ഫയൽ | सञ्चिका |
| Folder | ഫോൾഡർ | संपुटम् |
| Document | രേഖ | लेखः |
| Link | ലിങ്ക് | अनुबन्धः |
| Word | വാക്ക് | शब्दः |
| Line | വരി | पङ्क्तिः |
| Number | സംഖ്യ | सङ्ख्या |
| Phone number | ഫോൺ നമ്പർ | दूरभाषसङ्ख्या |
| Contact / Contacts | വിലാസവിവരം / വിലാസവിവരങ്ങൾ | सम्पर्कः / सम्पर्काः |
| Call (phone call) | ഫോൺ വിളി | आह्वानम् |
| Tag / Tags | അടയാളം / അടയാളങ്ങൾ | चिह्नम् / चिह्नानि |
| Total | ആകെ | योगः |
| Count | എണ്ണം | गणना |
| Size | വലുപ്പം | परिमाणम् |
| Type | തരം | प्रकारः |
| Status | നില | स्थितिः |

> **Phone number vs. Number.** A telephone number is `ഫോൺ നമ്പർ` in Malayalam; `സംഖ്യ` reads as
> "numeral" and stays the word for a plain number. Contact and Tag list the singular and plural
> forms because apps need both ("1 contact", "all contacts").

#### Confirmation words

| English | Malayalam | Sanskrit |
|---|---|---|
| Yes | അതെ | आम् |
| No | ഇല്ല | न |
| OK | ശരി | अस्तु |
| Are you sure? | ഉറപ്പാണോ? | निश्चयेन वा? |

#### About screen (matches the `aboutDetail<Key>` ARB keys in `guideline.md` §1.6)

| English | Malayalam | Sanskrit |
|---|---|---|
| About | ആപ്പിനെക്കുറിച്ച് | विषयपरिचयः |
| Version | പതിപ്പ് | संस्करणम् |
| Build | നിർമ്മിതി | निर्मितिसङ्ख्या |
| Author | രചയിതാവ് | लेखकः |
| Email | ഇമെയിൽ | विद्युत्पत्रम् |
| License | ലൈസൻസ് | अनुज्ञापत्रम् |
| AI used | ഉപയോഗിച്ച AI | प्रयुक्ता कृत्रिमबुद्धिः |
| IDE used | ഉപയോഗിച്ച IDE | प्रयुक्तं विकाससाधनम् |
| Help | സഹായം | साहाय्यम् |
| Feedback | പ്രതികരണം | प्रतिक्रिया |
| Contact | ബന്ധപ്പെടുക | सम्पर्कः |
| Terms | നിബന്ധനകൾ | नियमाः |
| Privacy policy | സ്വകാര്യതാ നയം | गोपनीयतानीतिः |

> About-screen row labels use the `aboutDetail<Key>` pattern (`guideline.md` §1.6), which is exempt
> from the short-label budget, so a long row label may wrap to two lines. Do not copy that liberty into a
> toolbar or a tab.

## 5. Short-label budget for these languages

These limits extend the generic budget table in standard §8.6:

| Language | Target | Hard limit |
|---|---|---|
| Malayalam | 1–2 words | 22 characters |
| Sanskrit | 1 word (nominal form preferred) | 22 characters |

- In Sanskrit follow the form convention in §4.1 of this pack: a nominal form for titles, tabs and
  labels (`अन्वेषणम्`), a single-word polite imperative for buttons (`अन्विष्यताम्`). Never a
  multi-word verb phrase.
- In Malayalam prefer the common everyday word over a Sanskritized formal one, unless the app's
  subject matter calls for the formal register.
- Never ALL CAPS in Malayalam or Sanskrit.

## 6. Store listings

- Google Play: provide the listing in Malayalam (`ml-IN`) as well as English, with screenshots that
  show the real Malayalam UI.
- **Sanskrit is not an available listing language** on Google Play or the Apple App Store. It ships
  *inside* the app only; do not drop it from the app because a store cannot list it.

## 7. Checklist

- [ ] Fallback delegate covers `sa`; date picker + dialog widget test passes under `sa` (§1.1).
- [ ] All dates and numbers use `formattingLocale(...)` (§1.2).
- [ ] Fonts cover Malayalam and Devanagari on every declared platform (§2).
- [ ] `app_sa.arb` passes the Hindi-marker gate (§4.1) and uses the glossary (§4.4).
- [ ] Malayalam follows §4.2 (no lazy transliterations, correct negation, atomic chillu).
- [ ] New glossary terms are marked "needs native-reader review" in the change log.
- [ ] Malayalam / Sanskrit short labels fit within 22 characters (§5).
