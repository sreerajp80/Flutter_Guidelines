# Plan: Make the About screen dynamic in the guideline

**Status:** completed

## What is the issue

`guideline.md` (§1) defines the `app_config.json` `details` map and the `AppConfig` model,
but it does not state that the **About screen MUST render the `details` map dynamically**.
The user wants: whatever key/value fields are added to `details` in `app_config.json` MUST
appear on the About screen automatically, with no code change.

The good news — the reference Text App already does exactly this
(`about_section.dart` loops `for (final entry in c.details.entries)`). This is just not
captured as a rule in the guideline.

## The plan for the fix

Edit `guideline.md` only. Add a new subsection under §1 (e.g. **§1.6 About screen — render
`details` dynamically**) that states:

- The About screen MUST iterate over `AppConfig.details` and render one row per entry, in
  order, skipping entries with an empty key or value.
- It MUST NOT hard-code the field names (Author, Email, …) — adding/removing a key in the
  JSON is the only change needed to add/remove a row.
- Show a short reference snippet based on the TextApp pattern:
  ```dart
  for (final entry in config.details.entries)
    if (entry.key.trim().isNotEmpty && entry.value.trim().isNotEmpty)
      ListTile(
        title: Text(entry.key),
        subtitle: Text(entry.value),
      ),
  ```
- Note the optional nicety: if a key equals `email`, make the row tappable (`mailto:`).

Also add one line to the §4 checklist: "About screen renders `details` dynamically (no
hard-coded field names)."

### Files to be changed

- **Edit:** `guideline.md` (add §1.6 + one checklist line).
- No app source files are touched.

## After implementation

Write a change log to `change_log/`.
