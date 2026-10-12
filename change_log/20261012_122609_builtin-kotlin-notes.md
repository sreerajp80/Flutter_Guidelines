# Change log: Update built-in Kotlin notes for Flutter 3.47+ / AGP 9+

Implements plan: `plans/20261012_122452_builtin-kotlin-notes.md`

## What changed

### `flutter_build_flavors_guide.md`

In "Android Toolchain Baseline (AGP 9+ / Gradle 9+)", the "Notes:" list:

- Removed the old first note. It said to keep `android.builtInKotlin=false` and
  `android.newDsl=false`.
- Added three notes in its place:
  1. **Built-in Kotlin (Flutter 3.47+, AGP 9+)**: set `android.builtInKotlin=true`. Do not apply
     `kotlin-android` / `org.jetbrains.kotlin.android` in the app module. Keep the
     `apply false` Kotlin plugin line in `settings.gradle.kts`. The
     `kotlin { compilerOptions { } }` block stays as it is.
  2. Check each Android plugin's build file before turning built-in Kotlin on. A plugin that
     always applies KGP breaks the build. Rollback steps are included.
  3. Keep `android.newDsl=false` until a separate migration to the new AGP DSL.
- The other notes (no `project.exec` in Gradle 9, the clean build check) are unchanged.

### `flutter_project_engineering_standard.md`

- Replaced the "Migrator flags" bullet with two bullets:
  - **Built-in Kotlin**: a short version of the rule above. It points to the flavors guide for
    details and rollback.
  - **`android.newDsl=false`**: keep it until a separate migration to the new AGP DSL.

## Not changed

- The `settings.gradle.kts` example in the flavors guide already has the `apply false` Kotlin
  plugin line, so it was left as it is.
- No other guide in the repo said to keep `builtInKotlin=false` or to apply `kotlin-android`
  in the app module.
