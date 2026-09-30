# Release Signing Guide — Setoran Tahfidz

Google Play requires every app to be signed, and new apps must be uploaded as an
**Android App Bundle (`.aab`)** signed with your **upload key**. This folder
documents how to create that key and wire it into the Gradle build — it does
**not** contain any real keystore or secret. Never commit a real `.jks` file or
its passwords to git (both are already excluded via `.gitignore`).

## 1. Generate an upload keystore

Run this once, from any machine with a JDK installed (the JDK's `bin` folder
must be on your `PATH`, or give the full path to `keytool.exe`):

```powershell
keytool -genkey -v -keystore upload-keystore.jks -keyalg RSA -keysize 2048 -validity 10000 -alias upload
```

You will be prompted for:
- A keystore password (remember this — you cannot recover it if lost)
- A key password for the `upload` alias (can be the same as the keystore password)
- Your name / organization / city / state / country code (for the certificate; cosmetic only)

This produces `upload-keystore.jks`. **Store it safely outside of git**
(e.g. a password manager vault, or a private secrets store) — if you lose it
and haven't enrolled in Play App Signing you will not be able to publish
updates to your app ever again under the same listing. Google Play App Signing
(enabled by default for new apps) mitigates this: Google keeps the real
signing key and you only need to keep this **upload key** safe to authenticate
new uploads; if you ever lose the upload key, Google support can help you
reset it since it's not the actual signing key used to sign the APK served to
users.

## 2. Create `android/keystore.properties`

This file is **gitignored** — create it locally (or in CI as a secret file)
with your real values:

```properties
storeFile=C:\\path\\to\\upload-keystore.jks
storePassword=your-keystore-password
keyAlias=upload
keyPassword=your-key-password
```

Place it at `android/keystore.properties` (sibling to `android/app/`).

## 3. Gradle already wired for signing

`android/app/build.gradle` in this repo already contains a `signingConfigs.release`
block that reads from `keystore.properties` **if it exists**, and falls back to
the debug config (with a warning) if it doesn't — so a normal `assembleDebug`
still works for anyone without the file. Once you've created
`android/keystore.properties` as above, running:

```powershell
cd android
./gradlew bundleRelease
```

produces a **signed, release-ready** `.aab` at:

```
android/app/build/outputs/bundle/release/app-release.aab
```

This is the file you upload to Google Play Console.

## 4. (Optional) Signed release APK for manual testing

```powershell
cd android
./gradlew assembleRelease
```

produces `android/app/build/outputs/apk/release/app-release.apk`, also signed
with the same upload key, useful for sideloading/testing before you submit to
Play.

## Summary of what NOT to commit

- `upload-keystore.jks` (or any `*.jks` / `*.keystore` file)
- `android/keystore.properties`
- Any plaintext passwords

All of the above are already covered by the repository's root `.gitignore`.
