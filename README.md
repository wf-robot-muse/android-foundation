# Android Foundation

Native Android starter app (Kotlin + Jetpack Compose) maintained
through GitHub only: all building and testing runs on GitHub Actions
Linux runners. No local PC is required.

## Current configuration

| Item | Value |
| --- | --- |
| Language / UI | Kotlin 2.2.21, Jetpack Compose (Material 3) |
| Android Gradle Plugin | 8.13.0 |
| Gradle | 8.14.3 (installed in CI by setup-gradle, version-pinned in the workflow) |
| JDK | 17 (Temurin, set up in CI) |
| compileSdk / targetSdk | 36 (Android 16) — see upgrade note below |
| minSdk | 29 (Android 10) |
| Application ID | `com.wf.robot.muse.androidfoundation` |
| Signing | Not configured yet (CI bundle is unsigned; debug APK is debug-signed) |

Android 17 (API 37) is the current Android release. compileSdk stays at
36 until the pinned AGP release confirms API 37 support — bumping it is
the first upgrade task, and it already exceeds Google Play's target API
requirements. Emulator tests in CI already run on an API 37 system
image.

## How CI works

`.github/workflows/android.yml` has two jobs on every push to `main`,
on pull requests, and on manual dispatch:

1. **Build** (Linux runner): builds the debug APK and the release App
   Bundle, runs JVM unit tests, and uploads both artifacts.
2. **Emulator test** (Linux runner, KVM enabled): boots an API 37
   (Android 17) x86_64 emulator and runs the instrumented tests —
   the closest available profile to current Pixel/Galaxy hardware.

Public repositories get unlimited standard GitHub Actions minutes, so
this pipeline costs nothing to run.

## What you can and cannot do yet

- The **debug APK** artifact is installable on any Android phone
  (allow "install from unknown sources" for the browser/files app used
  to open it). This is the fastest way to try builds on real hardware.
- The **release AAB** is **unsigned**. It cannot be uploaded to Google
  Play until the upload key is wired in.

## Signing and Google Play

Activates when Wade's Google Play Console account ($25 one-time) is in
place. Wade handles the account, fee, and Play submission.

1. Generate an upload keystore and store it (base64) plus its passwords
   as the GitHub secrets listed in `fastlane/Fastfile`.
2. Add the `signingConfig` in `app/build.gradle.kts` (pattern described
   in the Fastfile comments) so release builds are signed in CI.
3. Add the Play service-account JSON secret and implement the
   `internal` lane to upload to the internal testing track.

## Upgrading when Google releases new versions

- **New Android version / API level:** when AGP adds support, raise
  `compileSdk` and `targetSdk` in `app/build.gradle.kts` (one-line
  change each), and raise `api-level` in the emulator-test job in
  `.github/workflows/android.yml` to match.
- **Toolchain:** AGP, Kotlin, and the Compose plugin versions are
  pinned together in the root `build.gradle.kts` — upgrade them as a
  set and let CI verify. The Gradle version is pinned in
  `gradle/wrapper/gradle-wrapper.properties`.
- **New Pixel/Galaxy devices:** the CI emulator uses the generic API 37
  x86_64 system image; device-specific behavior is best verified with
  the debug APK on real hardware.

## Gradle wrapper note

The wrapper scripts (`gradlew`, `gradlew.bat`) and
`gradle/wrapper/gradle-wrapper.properties` (with the distribution
checksum pinned) are in the repo, but the binary
`gradle/wrapper/gradle-wrapper.jar` is not: the connector used to push
this foundation transfers text files only, and the JAR is binary. CI
therefore installs Gradle 8.14.3 directly (setup-gradle) and does not
use the wrapper. To restore the JAR, run
`gradle wrapper --gradle-version 8.14.3` in any local clone and commit
the result, or upload a `gradle-wrapper.jar` to `gradle/wrapper/` via
the GitHub web UI. After that, `./gradlew` works as usual and the
workflow and Fastfile can be switched back to it.

## Repository visibility

This repository is public by design: it contains no secrets, signing
material lives in GitHub secrets, and public visibility is what makes
the Actions minutes free and unlimited.

