---
name: sideload-hebrish-android
description: 'Sideload the latest Hebrish Android app (hebrish.app) onto a phone over adb - pick the device the user names (USB serial or wireless debugging), check it can run Hebrish, download the latest APK from cdn.hebrish.app and verify its SHA-256, install or update it with adb install -r, and report the installed version. Use when a user asks to sideload, install or update Hebrish on an Android phone or tablet from a computer with adb, or says "push the latest Hebrish APK to my phone". Do NOT use for adding the Hebrish F-Droid repository (use setup-hebrish-fdroid-android) or for Hebrish on Linux (use install-hebrish-linux).'
license: MIT
compatibility: A computer with adb (Android platform-tools), curl and a SHA-256 tool; an Android 9+ arm64 device with USB or wireless debugging on; network access to cdn.hebrish.app.
---

# Sideload Hebrish onto an Android device

Hebrish is on-device English dictation that writes Hebrew words in romanized form correctly
(code-switching). It is a personal project by Daniel Rosehill. The Android app is package
`app.hebrish.dictation`, for **Android 9 or later on arm64 (arm64-v8a)** only. This skill
downloads the latest release APK and installs it over adb on the device the user chooses.

Sideloaded copies do not update themselves. For automatic updates, the recommended install is
the Hebrish F-Droid repository (skill `setup-hebrish-fdroid-android`). Both use the same signing
key, so a sideloaded copy and an F-Droid copy update each other without losing data.

Facts used below:

| | |
| --- | --- |
| Package | `app.hebrish.dictation` |
| Release manifest | `https://cdn.hebrish.app/downloads/latest.json`, entry `files.android` (`url`, `name`, `sha256`, `size`, `version`) |
| Stable download | `https://cdn.hebrish.app/apps/android/Hebrish-latest.apk` (also `https://hebrish.app/get/android`) |
| APK signing certificate (SHA-256) | `5cf6822350d0b2ff124021a0df10bec1c63a6e5e6159fa6029507134a1446994` |

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
  debugging. A new computer is paired first with "Pair device with pairing code", which shows its
  own IP:port and a code; then connect to the IP:port shown on the Wireless debugging screen
  itself (a different port):

```bash
adb pair IP:PAIRING_PORT
adb connect IP:PORT
```

  The serial is then `IP:PORT`.
- **Several devices and none named:** list them and ask which one. Never guess.
- `unauthorized`: ask the user to unlock the phone and accept "Allow USB debugging?" (tick
  "Always allow from this computer"), then list again. `offline`: unplug and replug, or
  `adb kill-server` and list again. Nothing listed over USB: USB debugging is off (Settings →
  About phone → tap Build number seven times, then Developer options → USB debugging) or the
  cable is charge-only.

## 3. Check the device

```bash
adb -s SERIAL shell getprop ro.product.model
adb -s SERIAL shell getprop ro.build.version.sdk
adb -s SERIAL shell getprop ro.product.cpu.abilist
adb -s SERIAL shell dumpsys package app.hebrish.dictation | grep -E "versionName|versionCode|installerPackageName"
```

- `ro.build.version.sdk` must be **28 or higher** (Android 9), and `ro.product.cpu.abilist` must
  contain `arm64-v8a`. Otherwise stop: Hebrish does not run on this device.
- The last command shows the version already installed, if any (no output: not installed).

## 4. Download and verify the APK

```bash
curl -fsSL https://cdn.hebrish.app/downloads/latest.json
```

Read `files.android` from it: `version`, `name`, `sha256`. If the installed `versionName` from
step 3 already equals `version`, tell the user it is up to date and ask whether to reinstall.

```bash
curl -fL -o Hebrish-latest.apk https://cdn.hebrish.app/apps/android/Hebrish-latest.apk
sha256sum Hebrish-latest.apk
```

(macOS: `shasum -a 256 Hebrish-latest.apk`; Windows PowerShell:
`Get-FileHash Hebrish-latest.apk -Algorithm SHA256`.) The hash must equal `files.android.sha256`.
If it doesn't, wait five minutes (the CDN can serve the previous file for that long after a
release), download again and compare again. If it still differs, stop and report both hashes.

If `apksigner` (Android build-tools) is available, also check the signer:

```bash
apksigner verify --print-certs Hebrish-latest.apk
```

The `Signer #1 certificate SHA-256 digest` must be
`5cf6822350d0b2ff124021a0df10bec1c63a6e5e6159fa6029507134a1446994`. Without apksigner, say that
the signer was not checked; Android itself refuses an update signed with a different key.

## 5. Install

```bash
adb -s SERIAL install -r Hebrish-latest.apk
```

`-r` updates an existing install and keeps its data and settings. `Success` means it installed.
Failures:

| Message | What to do |
| --- | --- |
| `INSTALL_FAILED_UPDATE_INCOMPATIBLE` | The installed copy is signed with a different key. Only uninstalling fixes it, and that **deletes the app's data** (history, dictionary, downloaded models). Explain this and ask before running `adb -s SERIAL uninstall app.hebrish.dictation`, then install again. |
| `INSTALL_FAILED_VERSION_DOWNGRADE` | The phone has a newer version than the download. Leave it. |
| `INSTALL_FAILED_NO_MATCHING_ABIS` or `INSTALL_FAILED_OLDER_SDK` | The device is not arm64 or is older than Android 9 (step 3). |
| `INSTALL_FAILED_INSUFFICIENT_STORAGE` | Free space on the phone; the models it downloads later need several hundred MB more. |
| `INSTALL_FAILED_USER_RESTRICTED` or `INSTALL_FAILED_ABORTED` | The phone blocked or asked about the install: watch the phone's screen and accept. Some phones (for example Xiaomi) also need Developer options → "Install via USB" turned on. |

## 6. Confirm and report

```bash
adb -s SERIAL shell dumpsys package app.hebrish.dictation | grep -E "versionName|versionCode"
```

`versionName` must equal `files.android.version`. Report to the user: the device (model and
serial), the version before and after, the APK's SHA-256 and whether it matched, and whether the
signer was checked.

To open the app on the phone for first-run setup (it downloads a speech model and asks for the
microphone and, for typing into other apps, keyboard or accessibility permissions):

```bash
adb -s SERIAL shell monkey -p app.hebrish.dictation -c android.intent.category.LAUNCHER 1
```

Delete `Hebrish-latest.apk` afterwards if the user doesn't want to keep it. For a problem this
skill doesn't cover, send the user to <https://hebrish.app/feedback>.
