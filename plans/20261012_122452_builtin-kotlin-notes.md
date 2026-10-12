# Plan: Update built-in Kotlin notes for Flutter 3.47+ / AGP 9+

**Status:** completed

## Files to be changed

1. `flutter_build_flavors_guide.md`: the "Notes:" list under "Android Toolchain Baseline
   (AGP 9+ / Gradle 9+)", first note (lines 174–177).
2. `flutter_project_engineering_standard.md`: the "Migrator flags" bullet (lines 447–449).

No other files. No git submodule files are touched.

## What the issue is

Both guides tell projects to keep `android.builtInKotlin=false`. That is out of date for
Flutter 3.47+ with AGP 9+. AGP now compiles Kotlin by itself. If the app module still
applies `kotlin-android`, every build warns that this will fail in a future Flutter version.

Search done across the repo for `builtInKotlin`, `kotlin-android`,
`org.jetbrains.kotlin.android` and `newDsl`. Spots with the old advice:

- `flutter_build_flavors_guide.md:174-177`: "Keep them" (`builtInKotlin=false` and
  `newDsl=false`).
- `flutter_project_engineering_standard.md:447-449`: "Migrator flags": "Keep them until the
  Flutter app template drops them."

No guide tells projects to apply `kotlin-android` in the app module. The
`settings.gradle.kts` example in the flavors guide (line 142) already has
`id("org.jetbrains.kotlin.android") version "2.4.0" apply false`, which is still correct,
so it stays.

## Plan for the fix

### 1. `flutter_build_flavors_guide.md`

Replace the first note in the "Notes:" list with these three notes. The other notes (Gradle 9
has no `project.exec`, the clean build check) stay unchanged.

```markdown
- **Built-in Kotlin (Flutter 3.47+, AGP 9+).** AGP compiles Kotlin by itself. Set
  `android.builtInKotlin=true` in `android/gradle.properties`, and do NOT apply
  `kotlin-android` (or `org.jetbrains.kotlin.android`) in the app module's `plugins { }`
  block. If the app still applies it, every build warns that this will fail in a future
  Flutter version. Keep `id("org.jetbrains.kotlin.android") version "..." apply false` in
  `android/settings.gradle.kts`: it puts that Kotlin version on the build classpath, and AGP
  uses it instead of its own older bundled Kotlin. The `kotlin { compilerOptions { } }`
  block above stays as it is.
- Before turning built-in Kotlin on, check each Android plugin's
  `android/build.gradle(.kts)`. A plugin that always applies KGP (the Kotlin Gradle Plugin)
  breaks the build. Plugins that skip KGP when `android.builtInKotlin` is on, or that never
  apply it, are fine. Rollback: apply `kotlin-android` again and set
  `android.builtInKotlin=false`.
- The Flutter migrator also adds `android.newDsl=false`. Keep it until a separate migration
  to the new AGP DSL.
```

### 2. `flutter_project_engineering_standard.md`

Replace the "Migrator flags" bullet with a short version that matches, and point to the
flavors guide for details:

```markdown
- **Built-in Kotlin.** Set `android.builtInKotlin=true` in `android/gradle.properties` and do
  NOT apply `kotlin-android` (or `org.jetbrains.kotlin.android`) in the app module. Keep the
  `apply false` Kotlin plugin line in `android/settings.gradle.kts`. Check every Android
  plugin first: one that always applies KGP breaks the build. Details and rollback are in
  `docs/guidelines/flutter_build_flavors_guide.md` ("Android Toolchain Baseline").
- **`android.newDsl=false`.** The Flutter migrator adds it. Keep it until a separate
  migration to the new AGP DSL.
```

### 3. After the edits

- Write a change log in `change_log/` and set this plan's status to `completed`.
- Commit only the two guide files, this plan and its change log. The other uncommitted
  changes already in the working tree are left out. Then push to `origin/master`.
