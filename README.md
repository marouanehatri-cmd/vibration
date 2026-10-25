# Vibration — lightweight virtual space for Huawei Y8p (no GMS)

Package `com.vibration.app` · minSdk 28 · targetSdk 33 · arm64-v8a · built on BlackBox.

## Build
Push to GitHub → Actions → "Build Vibration" → download the `Vibration` artifact (`Vibration.apk`).
Before the first build, open https://github.com/FBlackBox/BlackBox and replace `+` in `app/build.gradle`
with the current JitPack tag. If a BlackBox class/method name differs from the one used in
`VibrationApp.java` / `Space.java`, adjust it there (those two files are the only BlackBox touchpoints,
plus `MainActivity.gameVersion()`).

## Why the APK stays under 15 MB
microG and the Play Store are NOT bundled (Play Store alone is bigger than the whole budget, and it is
Google's software, not ours to redistribute). You import them once, from the app, into the space.

## Put real Google Play inside the space (one-time)
Get these APKs (arm64) on the phone:
1. microG **GmsCore** (`com.google.android.gms`) — https://microg.org/download.html (or the microG F-Droid repo)
2. microG **Services Framework Proxy** (`com.google.android.gsf`) — same place
3. **Google Play Store** (`com.android.vending`) — an official, unmodified APK you obtained yourself (e.g. APKMirror, verified signature)

In Vibration: **Import APK into space** → select all three → Settings should show GmsCore/GSF/Play Store = yes.
Install order if you do it one by one: GmsCore → GSF proxy → Play Store.

### Signature spoofing (inside the virtual space ONLY)
The real Play Store checks that `com.google.android.gms` carries Google's signature. microG's does not,
so inside the space the virtual PackageManager must report Google's signature for `com.google.android.gms`
(and `com.google.android.gsf`) when queried by other virtual apps.
- This is done in BlackBox's virtual package manager (BPackageManagerService / its getPackageInfo path):
  when `packageName` is gms/gsf and the caller is inside the space, replace `PackageInfo.signatures`
  with the genuine Google signature set. The host system is never touched.
- Grant microG its permissions in the space: location, notifications, "display over other apps" not required.
- Play Store itself stays unmodified. No emulator hiding, no root/integrity bypass.
- BlackBox does not ship this spoof; it is a small patch to your BlackBox fork. Without it the Play Store
  will refuse to sign in or show "device not compatible".

### First sign-in
Open Play Store → sign in with your Google account (inside the space) → open eFootball → Update.
Then tick "I signed in" in Settings (status is manual: the space gives no reliable API for it).

## Fallback: Install APK as Update
Download the eFootball APK, tap **Install APK as Update**; it installs over the existing space copy
(data is kept as long as the package name and signature match).

## Known limits (be aware)
- FPS overlay shows the screen's frame cadence + free RAM, not the game's internal FPS.
- "60 FPS / 85% / Standard / OpenGL" are stored as the target profile; eFootball's own in-game settings
  control the final result — set graphics to Standard and frame rate to 60 there.
- Google may block sign-in or flag unusual devices in a virtual environment; no workaround is included.
- Using a virtual space with an online game carries some account risk under the game's terms; no hiding is done.
