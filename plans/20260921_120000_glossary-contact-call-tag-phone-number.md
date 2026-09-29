# Plan: Add Contact, Call, Tag and Phone number to the §8.5.4 glossary

**Status:** done — see [../change_log/20260921_120500_glossary-contact-call-tag-phone-number.md](../change_log/20260921_120500_glossary-contact-call-tag-phone-number.md)

## The issue

A contacts-and-dialer app built on this standard uses four terms on almost every screen that the
§8.5.4 glossary does not define: Contact, Call, Tag and Phone number. Without glossary entries, each
app would choose its own Malayalam and Sanskrit words, which is exactly what §8.5.4 exists to
prevent.

"Phone number" also clashes with the existing Number row, which says Malayalam `സംഖ്യ`, never
`നമ്പർ`. `സംഖ്യ` reads as "numeral" and does not fit a telephone number.

## Files to be changed

| File | Change |
|---|---|
| `flutter_project_engineering_standard.md` | §8.5.4 "Content and fields": add four rows after Number, and a short note under the table on Phone number vs. Number and on singular/plural forms. |

## The fix

Add these rows, as approved by a fluent reader (the repository owner):

| English | Malayalam | Sanskrit |
|---|---|---|
| Phone number | ഫോൺ നമ്പർ | दूरभाषसङ्ख्या |
| Contact / Contacts | വിലാസവിവരം / വിലാസവിവരങ്ങൾ | सम्पर्कः / सम्पर्काः |
| Call (phone call) | ഫോൺ വിളി | आह्वानम् |
| Tag / Tags | അടയാളം / അടയാളങ്ങൾ | चिह्नम् / चिह्नानि |

All four fit the §8.6 22-character budget.

## Risk

Low. Additive glossary rows. Apps that already used other words for these terms (for example the
loanwords `കോൺടാക്റ്റ്`, `കോൾ`, `ടാഗ്`) should move to the glossary words.
