# Kirikiroid2-Yuri 1.4.0beta-ui1: UI refresh notes

Status: **source changes only. Nothing here has been compiled or run.**
The author's sandbox had no Android SDK/NDK and no network, so every item below is
"written, reviewed against the source, not built, not tested on a device".

## How the app is put together (what was inspected)

- Java: `MainActivity` just extends `org.tvp.kirikiri2.KR2Activity`. That class owns the GL surface,
  forwards touch/key events to C++ over JNI, shows native `AlertDialog`s for engine message/input boxes,
  and handles permissions and storage paths.
- Everything you see in the launcher, file browser and settings is **Cocos2d-x C++**
  (`src/core/environ/ui/*.cpp`, game screen in `src/core/environ/cocos2d/MainScene.cpp`).
  Layouts are Cocos Studio files: sources in `project/ui/cocosstudio/ui/*.csd`, compiled `.csb`
  loaded at runtime from `assets/ui/`. The `.csb` files are not in this repo; the build takes the
  whole `assets/` folder from the 1.3.9 release APK (see `script/_fetch.sh`, CI workflow).
- UI text comes from `project/ui/Resources/res/locale/*.xml`. The stock build ships the locale files
  from the 1.3.9 APK instead, so edits to the repo copies never reached the app. This is now fixed
  by a `syncLocale` Gradle task (see below).
- Because the UI is not Android views, Compose/XML cannot restyle it, and Compose is not used.

## What changed

| Area | Change | Files |
|---|---|---|
| Dark theme | Optional recolor of fully opaque solid panels that use the stock Cocos Studio colors (light-blue header to deep navy, grey panels darker). Layout, sizes and input untouched. Toggle: Global Settings, "Dark Theme Refresh". | `ui/BaseForm.cpp` |
| Interface size | New slider 80% to 160% (default 100%) multiplying the existing global UI scale. Applies after restarting the app. All screens use this one scale, so row and button heights scale together. | `ui/PreferenceConfig.h`, `cocos2d/MainScene.cpp` |
| Clearer text | Reworded and de-typo'd English settings labels and error messages (placeholders `%1`, `%d` kept); better read-only-storage message that points to "All files access". About text now points bug reports at the Yuri repo. | `locale/en_us.xml` |
| New strings | `preference_ui_theme_dark`, `preference_ui_scale`, `err_create_folder` added to all four locale files (English text in ja/zh until translated). | `locale/*.xml` |
| Folder error | "Fail to create folder." now uses the localized, more helpful message. | `ui/MainFileSelectorForm.cpp` |
| Permissions | Explains and requests "All files access" on Android 11+ (checked via reflection, no compileSdk change). Declares `MANAGE_EXTERNAL_STORAGE`. Existing READ/WRITE permissions kept. | `KR2Activity.java`, `AndroidManifest.xml` |
| Dialogs | Engine message/input boxes use the dark Material dialog theme. | `KR2Activity.java` |
| Build | `syncLocale` task copies repo locale files into `assets/locale` before every build. Debug builds get applicationId `com.yuri.kirikiri2.dev` and name "Kirikiroid2-Yuri Dev". Debug signing falls back to the standard debug keystore when no release key is configured. Version name `1.4.0beta-ui1`. | `app/build.gradle`, `app/debug_res/` |
| CI | Runner `ubuntu-22.04` (20.04 images are retired), manual "Run workflow" trigger, installs SDK/NDK/CMake packages, uploads the APK as an artifact, release step only on tags. | `.github/workflows/build_android.yml` |

Preserved: native engine, plugins, native libraries, save/load code, key mapping, `sign.jks`.

## Application ID and signing

- Release builds keep `com.yuri.kirikiri2`. Debug builds are `com.yuri.kirikiri2.dev`, so they
  install **next to** your current app and do not replace it.
- An update over an installed copy only works if it is signed with the same key as that copy.
  Unknown for your installed APK; the repo's `sign.jks` passwords are CI secrets, not in the repo.
- Game saves live with the game files, not in the app's private data. Not verified for every game.
  Back up a game folder before first testing any build.

## Building

CI (easiest): push to a GitHub fork, Actions tab, "build android", Run workflow, download
`krkr2yuri-debug-apk`. The workflow downloads prebuilt dependencies from the upstream project's
`deps` release and the engine `assets` from the 1.3.9 APK.

Local: Android Studio or CLI with JDK 11, NDK 25.2.9519653, CMake 3.22.1, build-tools 33.0.2,
platform android-29. Provide `thirdparty/build` and `thirdparty/port` (tarballs from the upstream
`deps` release) and `assets/` (from the 1.3.9 APK), then in `project/android`: `./gradlew assembleDebug`.
Output: `build_android/outputs/apk/debug/krkr2yuri_v1.4.0beta-ui1.apk`.

## Test checklist (not yet run)

1. App starts; dark dialogs appear; "All files access" explanation appears once and opens the right Settings page.
2. Browse to a game folder with `startup.tjs`; it launches; save, quit, relaunch, load.
3. Global Settings: toggle Dark Theme Refresh and reopen screens; move Interface Size, restart, check tap targets.
4. Open Settings for an individual game; confirm per-game settings still save.
5. Rotate and fullscreen behave on the S25 Ultra (cutout, gesture bars).
6. Read-only/unwritable location shows the new storage message.

## Known limits / not done

- Full visual restyle is not done: the screens are binary `.csb` layouts and editing them needs
  Cocos Studio (discontinued) or rebuilding them in C++. The color pass is a safe subset and might not
  catch every panel; if something looks wrong, switch it off in Global Settings.
- File browser: no structural changes yet (row layout, breadcrumbs, search). Larger targets come only
  from the Interface Size slider.
- Control customization beyond the existing key map and touch/mouse modes: not added.
- Fullscreen/cutout handling unchanged (still `sensorLandscape`, `Theme.NoTitleBar.Fullscreen`).
- Storage still relies on the app's legacy storage approach plus All files access; SAF access to
  external SD cards is still the upstream TODO.
- The upstream README says this beta is aimed at debugging and recommends 1.3.9.
- Translations for ja/zh strings added here are English placeholders.
