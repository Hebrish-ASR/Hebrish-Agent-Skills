---
name: setup-hebrish-fdroid-android
description: 'Set up the Hebrish F-Droid repository on an Android phone over adb, so Hebrish (hebrish.app) installs and updates automatically - pick the device the user names, install the F-Droid client if it is missing (verifying its signer), open the Hebrish repository link with its fingerprint in F-Droid, open the Hebrish app page there for the user to install, and confirm the result. Use when a user asks to add the Hebrish F-Droid repo, set up F-Droid for Hebrish, or install Hebrish "the recommended way" on an Android phone from a computer with adb. Do NOT use for a one-off APK install without F-Droid (use sideload-hebrish-android) or for Hebrish on Linux (use install-hebrish-linux).'
license: MIT
compatibility: A computer with adb (Android platform-tools) and curl; an Android 9+ arm64 device with USB or wireless debugging on, unlocked and in the user's hands to confirm prompts; network access to f-droid.org and fdroid.hebrish.app.
---

# Set up the Hebrish F-Droid repository over adb

Hebrish is on-device English dictation that writes Hebrew words in romanized form correctly
(code-switching). It is a personal project by Daniel Rosehill. F-Droid is the recommended way to
install the Android app (`app.hebrish.dictation`, Android 9+, arm64 only): once the Hebrish
repository is added, F-Droid installs Hebrish and offers each new version. This skill does the
setup from a computer over adb. Android asks the person holding the phone to confirm a few steps;
adb cannot press those buttons, so tell the user what to tap and wait.

Facts used below:

| | |
| --- | --- |
| Hebrish repository | `https://fdroid.hebrish.app/repo` |
| Repository signing key fingerprint (SHA-256) | `EF8C2D63D994A10CD489A978585510B47E47452EC09248591CB2C0F39C2BC1FA` |
| Add link (repository + fingerprint) | `https://fdroid.hebrish.app/repo?fingerprint=EF8C2D63D994A10CD489A978585510B47E47452EC09248591CB2C0F39C2BC1FA`; the same with scheme `fdroidrepos://` opens straight in an F-Droid client |
| F-Droid client | `https://f-droid.org/F-Droid.apk`, package `org.fdroid.fdroid`, signing certificate SHA-256 `43238d512c1e5eb2d6569f4a3afbf5523418b82e0a3ed1552770abb9a9c9ccab` |
| Hebrish packages in the repository | `app.hebrish.dictation` (Hebrish), plus `app.hebrish.keyboard` (Hebrish Keyboard) and `app.hebrish.meorav` (Meorav, beta) |

Run each command as written, show the user what it printed, and stop to ask when a step says so.
In every command, replace `SERIAL` with the device's serial from step 2.

## 1. adb

```bash
adb version
```

If adb is missing, offer to install Android platform-tools: Ubuntu/Debian `sudo apt install adb`
(show the command and ask before running anything with `sudo`), macOS `brew install
android-platform-tools`, Windows the platform-tools zip from
<https://developer.android.com/tools/releases/platform-tools>.

## 2. Choose the device

```bash
adb devices -l
```

- **The user named a device** (a serial, a model name, or an IP address): use that one. A model
  name matches the `model:` field in the list.
- **Wireless debugging** (Android 11+): on the phone, Settings → Developer options → Wireless
  debugging. Pair first with "Pair device with pairing code" (its own IP:port and code), then
  connect to the IP:port shown on the Wireless debugging screen itself:

```bash
adb pair IP:PAIRING_PORT
adb connect IP:PORT
```

- **Several devices and none named:** list them and ask which one. Never guess.
- `unauthorized`: ask the user to unlock the phone and accept "Allow USB debugging?", then list
  again. Nothing listed over USB: USB debugging is off or the cable is charge-only.

Check the device can run Hebrish:

```bash
adb -s SERIAL shell getprop ro.build.version.sdk
adb -s SERIAL shell getprop ro.product.cpu.abilist
```

The SDK must be **28 or higher** and the list must contain `arm64-v8a`; otherwise stop.

## 3. F-Droid client

```bash
adb -s SERIAL shell pm list packages org.fdroid.fdroid
```

`package:org.fdroid.fdroid` means F-Droid is installed: go to step 4. If the user uses another
F-Droid client (Neo Store `com.machiav3lli.fdroid`, Droid-ify `com.looker.droidify`), say so and
use it in step 4 in place of `org.fdroid.fdroid`. Otherwise install F-Droid:

```bash
curl -fL -o F-Droid.apk https://f-droid.org/F-Droid.apk
apksigner verify --print-certs F-Droid.apk
```

The `Signer #1 certificate SHA-256 digest` must be
`43238d512c1e5eb2d6569f4a3afbf5523418b82e0a3ed1552770abb9a9c9ccab`; if it differs, stop. If
`apksigner` (Android build-tools) isn't available, tell the user the signer was not checked and
ask whether to go on.

```bash
adb -s SERIAL install F-Droid.apk
```

`Success` means it installed. If the phone shows a prompt (some phones, for example Xiaomi, need
Developer options → "Install via USB"), ask the user to accept.

## 4. Add the Hebrish repository

Ask the user to unlock the phone, then:

```bash
adb -s SERIAL shell am start -a android.intent.action.VIEW -d "fdroidrepos://fdroid.hebrish.app/repo?fingerprint=EF8C2D63D994A10CD489A978585510B47E47452EC09248591CB2C0F39C2BC1FA" org.fdroid.fdroid
```

F-Droid opens an "Add repository" screen for `fdroid.hebrish.app`. Ask the user to check the
address and tap **Add** (wording varies by F-Droid version). F-Droid then downloads the
repository index; that can take a minute. If F-Droid says the fingerprint does not match, stop:
do not add the repository without it.

If `am start` prints `Error: Activity not started` or F-Droid opens without the Add screen, ask
the user to open F-Droid once, let it finish its first update, and run the command again. If it
still doesn't work, the user adds it by hand: F-Droid → Settings → Repositories → **+**, and pastes
the `https://` add link from the table above (it carries the fingerprint).

## 5. Install Hebrish from F-Droid

```bash
adb -s SERIAL shell am start -a android.intent.action.VIEW -d "market://details?id=app.hebrish.dictation" org.fdroid.fdroid
```

F-Droid opens the Hebrish page. Ask the user to tap **Install**. The first time, Android asks
whether F-Droid may install apps ("Install unknown apps"): the user allows it and goes back. If
F-Droid says the app is not found, the repository index has not finished updating: wait and run
the command again. If Hebrish is already installed (from an APK from hebrish.app or GitHub),
F-Droid shows **Update** or nothing to do: both use the same signing key, so F-Droid updates that
copy and keeps its data.

## 6. Confirm and report

```bash
adb -s SERIAL shell dumpsys package app.hebrish.dictation | grep -E "versionName|versionCode|installerPackageName"
```

A `versionName` means Hebrish is installed; `installerPackageName=org.fdroid.fdroid` (or the
other client) means F-Droid installed it and will offer its updates. Compare with the newest
version:

```bash
curl -fsSL https://cdn.hebrish.app/downloads/latest.json
```

(`files.android.version`.) Report to the user: the device (model and serial), whether F-Droid
was installed by this skill and its signer checked, that the Hebrish repository was added, and
the Hebrish version now installed. Then they open Hebrish for first-run setup (it downloads a
speech model and asks for the microphone, and for keyboard or accessibility permission to type
into other apps). Delete `F-Droid.apk` if it was downloaded. For a problem this skill doesn't
cover, send the user to <https://hebrish.app/feedback>.
