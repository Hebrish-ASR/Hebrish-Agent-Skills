---
name: troubleshoot-hebrish-linux
description: Diagnose and fix problems with the Hebrish dictation app (hebrish.app) on Linux - the hotkey does nothing, text is not pasted or lands only on the clipboard, no microphone or no level, the model download fails, no tray icon, and APT errors from the Hebrish repository (NO_PUBKEY or signature errors, wrong architecture, unsupported Ubuntu version, package or source missing after a purge). Use when a user reports that Hebrish on Ubuntu or Linux is not working, or an apt command mentions cdn.hebrish.app.
metadata:
  publisher: hebrish.app
  updated: "2026-10-21"
  source: https://github.com/Hebrish-ASR/Hebrish-Agent-Skills
---

# Troubleshoot Hebrish on Linux

Hebrish is Daniel Rosehill's one-person, not-for-profit dictation app (hebrish.app). The Linux
app is the `hebrish` package; settings are in `~/.config/hebrish/config.json`, models, history
and state in `~/.local/share/hebrish/`, and the log goes to stderr.

## Rules for the agent

- **Show, then ask, before every `sudo` command.** Read-only checks can run without asking.
- Only apply the fixes below. Do not invent flags, config keys or paths, and do not edit
  `/usr/lib/hebrish/`. If nothing here matches, collect the facts from "Gather facts" and send the
  user to https://github.com/Hebrish-ASR/Hebrish-App/issues (or the app's About tab, which opens
  a pre-filled email to feedback@hebrish.app).
- Do not run `pkill -f hebrish`: the pattern also matches the shell running it. Quit the app from
  the tray menu (**Quit**) instead.

## Gather facts (read-only)

```bash
. /etc/os-release; echo "$ID $VERSION_ID $VERSION_CODENAME / $(dpkg --print-architecture)"
dpkg -s hebrish | grep -E '^(Status|Version):'
apt-cache policy hebrish
echo "session=$XDG_SESSION_TYPE desktop=$XDG_CURRENT_DESKTOP WAYLAND_DISPLAY=$WAYLAND_DISPLAY DISPLAY=$DISPLAY"
id -nG | tr ' ' '\n' | grep -qx input && echo "in input group" || echo "NOT in input group"
ls -l /dev/uinput; getfacl -p /dev/uinput 2>/dev/null | grep user
command -v wl-copy xclip ffmpeg
hebrish-model status
```

**See the app's log:** quit Hebrish from the tray, then start it from a terminal with `hebrish`
and reproduce the problem; messages and tracebacks print there. Also read any banner at the top
of the window: the app reports hotkey and paste problems there about 4 s after start.

## The hotkey does nothing

1. **Not in the `input` group** (the banner says "Can't read /dev/input, so the hotkey is off").
   The hotkey is read from `/dev/input`. Fix, with consent:
   `sudo usermod -aG input $USER`, then **log out and back in** (a new terminal is not enough;
   `id -nG` in the new session must list `input`). Meanwhile, dictate with the Talk button, the
   tray (middle-click the tray icon) or `hebrish --toggle`.
2. **The key does not exist on this keyboard.** The default toggle key is F14, which most
   keyboards lack. Set another in **Settings → Dictation → Toggle key** (or a **Push-to-talk
   key**).
3. **Another app reacts to the same key.** Keys are read without being grabbed, so a key shared
   with another dictation tool starts both, and the key also reaches the focused app. Parlance
   defaults to F13 and Hebrish to F14 for that reason. Pick a key nothing else uses.
4. **A modifier as the hotkey** (for example Right Ctrl): pressing another key while holding it
   cancels the take, by design, so Right Ctrl+C stays a shortcut.
5. Keyboards plugged in after start are picked up within about 3 s; no restart needed.
6. Still nothing: check the hotkey works through the app at all by running `hebrish --toggle`.
   If that records, the problem is key input; report it with the gather-facts output.

## Text is not pasted

How it works: the app puts the text on the clipboard, then sends Ctrl+V through a virtual
keyboard on `/dev/uinput` (named `hebrish-keyboard`). Every take also appears in the Dictate
tab's text box.

1. **Output mode.** Dictate tab → Output must be *Paste at the cursor* (the default). *Clipboard
   only* and *This window's text box only* do not paste by design.
2. **Banner "Cannot open /dev/uinput (...); transcripts go to the clipboard only".** The package
   ships the udev rule `/usr/lib/udev/rules.d/70-hebrish-uinput.rules`, which grants the logged-in
   seat user access to `/dev/uinput` (`uaccess`). Check it is there and reload it, with consent:
   ```bash
   cat /usr/lib/udev/rules.d/70-hebrish-uinput.rules
   sudo udevadm control --reload-rules && sudo udevadm trigger --sysname-match=uinput
   ```
   Then log out and back in (the access is granted per login session) and restart the app. If the
   rule file is missing, reinstall: `sudo apt install --reinstall hebrish`.
3. **Terminals** paste with Ctrl+Shift+V, not Ctrl+V. Set **Settings → Paste shortcut** to
   `ctrl+shift+v` if the user dictates mostly into terminals. The app does not switch per app.
4. **Clipboard tool missing** (error "Install wl-clipboard and xclip"): the package depends on
   both, so reinstall the dependencies: `sudo apt install wl-clipboard xclip`.
   Which tool is used: on Wayland, `wl-copy` (KDE, wlroots compositors, Hyprland); on GNOME,
   `xclip` first through XWayland, because GNOME's Mutter lacks the data-control protocol and
   `wl-copy` there would steal focus from the target window. `xclip` needs `$DISPLAY` set (XWayland
   running); without it the app falls back to `wl-copy`.
5. **Focus.** The paste goes to whichever window has keyboard focus when decoding finishes. Keep
   the cursor in the target window until the text arrives.
6. **Hebrew keyboard layout:** not a cause. The app pastes rather than types, so it works with any
   active layout.

## No microphone, or the level stays flat

1. Dictate tab → **Microphone** shows the device in use (for *System default*, which device that
   is right now). Use the switcher (with rescan) to pick the real mic, then **Test mic**: the
   meter should reach the *good* zone when speaking; *too quiet* means raise the input volume in
   the system sound settings; a CLIP lamp means lower it.
2. Check the system sees the mic: `pactl list short sources` (PipeWire/PulseAudio). Make sure it
   is not muted in the desktop's sound settings.
3. **"No speech" / nothing inserted for a short take.** Voice detection (Dictate tab, on by
   default) skips takes it judges silent. With a very quiet mic, raise the input level, or turn
   Voice detection off to test.

## The model download fails

Downloads come from cdn.hebrish.app (Hugging Face for the stock model), are about 660 MB
(137 MB for Sabich-Parakeet-110M), and are SHA-256 checked.

1. **"Not enough disk space: need N MB, have M MB"**: free space in `~/.local/share/hebrish/` (or
   the folder chosen under Models tab → *Change folder…*), or choose another folder there.
2. **"SHA-256 mismatch"** or **"size ... != ..."**: a corrupted or cut-off transfer. Retry the
   download; the partial file is not used.
3. **Network errors / timeouts:** check `curl -fsSI https://cdn.hebrish.app/models/parakeet-sabich/latest.json`
   returns HTTP 200 from this machine (proxies and firewalls), then retry.
4. **CLI for a clearer error:** `hebrish-model check sabich` then `hebrish-model download sabich`
   prints progress and the exact error. `hebrish-model use sabich` selects it; restart the app.
5. A model can't be switched away from while it is downloading (the picker is disabled). Models
   other than Sabich, Sabich-Parakeet-110M, Jachnun and stock Parakeet do not run in the desktop
   app and have no Download button.

## No tray icon

The tray icon is a StatusNotifierItem. KDE shows it natively; Ubuntu's GNOME shows it through its
AppIndicator extension, enabled by default. Stock GNOME without that extension has no tray: closing
the window leaves the app running, so start it again from the app menu to get the window back, or
use `hebrish --toggle`.

## APT errors

| Error | Cause and fix |
| --- | --- |
| `NO_PUBKEY DBD1F5A22CA1B27B`, "The following signatures couldn't be verified", or "The signatures couldn't be verified because the public key is not available" | The keyring at `/usr/share/keyrings/hebrish-archive-keyring.gpg` is missing or wrong. Re-download it and check the fingerprint is `4745D63577C81400DBB743DFDBD1F5A22CA1B27B`: `curl -fsSL https://cdn.hebrish.app/apt/hebrish-archive-keyring.gpg \| sudo tee /usr/share/keyrings/hebrish-archive-keyring.gpg >/dev/null` then `gpg --show-keys /usr/share/keyrings/hebrish-archive-keyring.gpg`. Stop and report if the fingerprint differs. |
| Keyring missing but `/etc/apt/sources.list.d/hebrish.sources` still there | Typical after `sudo apt remove hebrish` (the sources file is a conffile and stays; the keyring goes with the package). Either finish the removal (`sudo apt purge hebrish`) or reinstall (restore the keyring as above, then `sudo apt update && sudo apt install hebrish`). |
| "doesn't support architecture 'i386'" or the package is not found on arm64 | The repository is amd64 only. The `Architectures: amd64` line in `hebrish.sources` keeps i386 out; restore the file exactly as in the install skill. On arm64 or other architectures Hebrish for Linux is not available yet. |
| `hebrish : Depends: python3 (< 3.15)` / `(>= 3.14)` but another version is to be installed, or "Unable to locate package hebrish" on another Ubuntu release | The package is pinned to Ubuntu 26.04's python3 (3.14). Other Ubuntu versions and other distros are not supported yet: remove the source (`sudo rm /etc/apt/sources.list.d/hebrish.sources`) and see https://hebrish.app/download. |
| "Unable to locate package hebrish" on Ubuntu 26.04 amd64 | The Hebrish source is not configured, for example after `apt purge hebrish` (purge deletes the sources file) or after the user deleted it (it then stays deleted across upgrades). Re-add the repository with the install skill's step 2, then `sudo apt update`. |
| Conflicts with `parakeet-dictation`, `codeswitch-dictation` or `voice-to-hebrish` | Expected: `hebrish` replaces those predecessor apps and apt removes them. Quit the old app before Hebrish's first start. `parlance` does not conflict. |
| dpkg asks about `/etc/apt/sources.list.d/hebrish.sources` ("modified (by you or by a script)") | The file on disk differs from the one the package ships. Keep the package's version (answer `Y`/install the maintainer's version) unless the user changed it on purpose. |

## After a fix

Restart the app (tray → Quit, then start Hebrish again) and repeat the quick test: cursor in a
text editor, hotkey, one sentence, hotkey. If the problem remains, open an issue at
https://github.com/Hebrish-ASR/Hebrish-App/issues with the gather-facts output and the terminal
log, and say what was already tried.
