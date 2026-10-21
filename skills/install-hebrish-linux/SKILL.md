---
name: install-hebrish-linux
description: Install and set up the Hebrish dictation app (hebrish.app) on Linux from its signed APT repository - preflight checks, adding the repository and verifying its signing key, installing the hebrish package, first-run setup (hotkey permissions, choosing and downloading a model), how updates arrive, and uninstalling. Use when a user asks to install, set up, update, reinstall or remove Hebrish (or "the Hebrish dictation app") on Ubuntu or another Linux computer.
metadata:
  publisher: hebrish.app
  updated: "2026-10-21"
  source: https://github.com/Hebrish-ASR/Hebrish-Agent-Skills
---

# Install Hebrish on Linux

Hebrish is on-device English dictation that writes Hebrew words in romanized form correctly
(code-switching): fine-tuned NVIDIA Parakeet models run on the CPU with sherpa-onnx. It is a
one-person, not-for-profit project by Daniel Rosehill. The Linux app is the `hebrish` package:
a window plus a tray icon; press a hotkey, talk, and the text is pasted at the cursor.

## Rules for the agent

- **Show, then ask, before every `sudo` command.** Print the exact command(s) you are about to
  run and get the user's go-ahead. Read-only checks (no `sudo`) can run without asking.
- Do not skip the key fingerprint check, and stop if it does not match.
- Do not edit files under `/usr/lib/hebrish/`, and do not invent flags, paths or settings that
  this skill does not name. If something here does not cover the user's situation, use the
  `troubleshoot-hebrish-linux` skill or point the user to
  https://github.com/Hebrish-ASR/Hebrish-App/issues.
- The model download (about 660 MB) is done by the app, not by `apt`. Mention it before it
  starts on a metered connection.

## 1. Preflight (read-only)

Run these and read the output:

```bash
. /etc/os-release; echo "ID=$ID VERSION_ID=$VERSION_ID CODENAME=$VERSION_CODENAME"
dpkg --print-architecture
dpkg -s hebrish 2>/dev/null | grep -E '^(Status|Version):' || echo "hebrish: not installed"
dpkg -l parakeet-dictation codeswitch-dictation voice-to-hebrish 2>/dev/null | grep '^ii' || echo "no predecessor apps"
echo "session: $XDG_SESSION_TYPE  desktop: $XDG_CURRENT_DESKTOP"
id -nG | tr ' ' '\n' | grep -qx input && echo "in input group" || echo "NOT in input group"
```

Decide:

| Result | What to do |
| --- | --- |
| `ID=ubuntu`, `VERSION_ID=26.04` (resolute), arch `amd64` | Supported. Continue. |
| Any other distro, Ubuntu version or architecture | **Not supported yet.** The package is amd64 only and pinned to Ubuntu 26.04's python3 minor version (3.14), so `apt` would refuse it or it would not start. Stop and tell the user so; point them to https://hebrish.app/download. Do not add the repository. |
| `hebrish` already installed | Check whether the repository is already set up (step 3's `apt-cache policy`). If it is, an update is just `sudo apt update && sudo apt upgrade`. If not (installed from a .deb before 0.6.2), add the repository (step 2) so updates arrive. |
| A predecessor app installed (`parakeet-dictation`, `codeswitch-dictation`, `voice-to-hebrish`) | The `hebrish` package conflicts with and replaces them, so `apt` will offer to remove them. Tell the user, and ask them to **quit the old app before Hebrish's first start**: on first start Hebrish moves the old app's downloaded model into its own folder, and an old app still running would then download 660 MB again. Hebrish adopts its settings and model, so nothing is downloaded twice. |
| `parlance` installed | Fine. Parlance is a separate app and is installed alongside; on first start Hebrish copies Parlance's settings (not the hotkey) and hardlinks its models. Parlance uses F13 and Hebrish F14, so they do not share a hotkey. |

## 2. Add the repository and install

Repository: `https://cdn.hebrish.app/apt`, suite `stable`, component `main`, arch `amd64`.
Signing key: "Hebrish APT Repository <info@hebrish.app>", RSA 4096, fingerprint
`4745 D635 77C8 1400 DBB7  43DF DBD1 F5A2 2CA1 B27B`.

Show the user these steps, then run them one block at a time (each `sudo` needs consent):

```bash
# 2a. Download the signing keyring
sudo install -d -m 0755 /etc/apt/keyrings
curl -fsSL https://cdn.hebrish.app/apt/hebrish-archive-keyring.gpg | sudo tee /etc/apt/keyrings/hebrish-archive-keyring.gpg >/dev/null

# 2b. Verify the fingerprint (no sudo)
gpg --show-keys /etc/apt/keyrings/hebrish-archive-keyring.gpg
```

The output must show `4745D63577C81400DBB743DFDBD1F5A22CA1B27B` and the uid
`Hebrish APT Repository <info@hebrish.app>`. If it does not, **stop**, delete the file
(`sudo rm /etc/apt/keyrings/hebrish-archive-keyring.gpg`) and report it at
https://github.com/Hebrish-ASR/Hebrish-App/issues. (An armored copy of the same key is at
https://cdn.hebrish.app/apt/hebrish-archive-keyring.asc.)

```bash
# 2c. Add the source. Keep this content byte-for-byte as written: the package ships the
#     same file as a conffile, and an identical file is adopted silently on install.
sudo tee /etc/apt/sources.list.d/hebrish.sources >/dev/null <<'EOF'
Types: deb
URIs: https://cdn.hebrish.app/apt
Suites: stable
Components: main
Architectures: amd64
Signed-By: /etc/apt/keyrings/hebrish-archive-keyring.gpg
EOF

# 2d. Install
sudo apt-get update && sudo apt-get install -y hebrish
```

**Short form.** If the user prefers one command, the setup script does 2a, 2c and 2d (keyring,
sources file, `apt-get update`, `apt-get install hebrish`). It does not show the fingerprint, so
run 2b afterwards:

```bash
curl -fsSL https://cdn.hebrish.app/apt/setup.sh | sudo sh
```

**Alternative: the .deb.** Installing the package downloaded from https://hebrish.app/get/ubuntu
(`sudo apt install ./hebrish_*_amd64.deb`) also enrols the machine in the repository from
desktop 0.6.2 on: the package itself installs the keyring and the sources file above.

## 3. Verify

```bash
apt-cache policy hebrish        # the Installed version, and a line with https://cdn.hebrish.app/apt stable/main amd64
dpkg -s hebrish | grep -E '^(Status|Version):'    # Status: install ok installed
command -v hebrish hebrish-model                  # /usr/bin/hebrish, /usr/bin/hebrish-model
```

`apt` pulls in the dependencies itself (PySide6, python3-evdev, python3-sounddevice,
libportaudio2, wl-clipboard, xclip, qt6-wayland, ...); `ffmpeg` is recommended, for opening
audio files in any format.

## 4. First-run setup

**4a. Hotkey permission (needs a re-login).** The hotkey is read from `/dev/input`, which needs
the `input` group. If preflight said "NOT in input group", show and (with consent) run:

```bash
sudo usermod -aG input $USER
```

Then the user must **log out and back in** (or reboot) before the hotkey works. Until then the
app shows a banner about it, and dictation still works from the Talk button, the tray, or
`hebrish --toggle`.

The paste itself goes through a virtual keyboard on `/dev/uinput`. The package installs a udev
rule for that (`/usr/lib/udev/rules.d/70-hebrish-uinput.rules`, the same `uaccess` rule Steam
ships) and reloads udev rules on install; nothing else to do.

**4b. Start it.** Open **Hebrish** from the app menu (or run `hebrish`). It stays in the tray
when the window is closed. On Ubuntu's GNOME the tray icon shows through the AppIndicator
extension that Ubuntu enables by default.

**4c. Pick and download a model.** On first start the app opens the **Models** tab. Recommend
**Sabich** (the complete model: code-switching plus tech vocabulary). Each 0.6B model is a
one-time download of about 660 MB, SHA-256 checked. Models the desktop app can run:

| Model | id | Download | When |
| --- | --- | --- | --- |
| **Sabich** (recommended) | `sabich` | ~660 MB | Start here; any recent computer, no GPU, about 1.5 GB of RAM |
| Sabich-Parakeet-110M | `sabich-parakeet-110m` | ~137 MB | Older or low-memory computers: faster, more mistakes |
| Jachnun | `codeswitch` | ~660 MB | Hebrew words without the tech vocabulary (Sabich covers this too) |
| Parakeet (stock) | `stock` | ~630 MB | Plain English, no Hebrew words |

Other models listed on hebrish.app (Sabich-Qwen3-ASR-1.7B, Sabich-Canary-Qwen-2.5B,
Sabich-asr4all, Shakshuka) do not run in the desktop app yet; the Models tab shows them greyed
out. **Help me choose** on that tab picks one for the computer.

The user clicks **Download** on the Sabich card, then **Switch to Sabich** if it is not already
in use. Or, from a terminal (no sudo; the app re-reads the choice on its next start):

```bash
hebrish-model list               # ids and which one is selected/installed
hebrish-model download sabich
hebrish-model use sabich
```

**4d. Hotkey.** The default toggle key is **F14**: tap to start, tap again to stop; holding it
works as push-to-talk. Many keyboards have no F14, so ask which key the user wants. Change it in
**Settings → Dictation → Toggle key** (any key on any keyboard, or a foot pedal), and optionally
set a separate **Push-to-talk key** there. Avoid a key another dictation tool already uses: keys
are read without being grabbed, so both apps would react. For a desktop-environment shortcut
instead, bind it to the command `hebrish --toggle`.

**4e. Quick test.** Put the cursor in a text editor, press the hotkey, say a sentence with a
Hebrew word in it ("I need to renew my teudat zehut at the misrad hapnim"), press it again. The
text should be pasted at the cursor. Optional settings worth mentioning: **Start at login, in
the tray** (Settings), and **Paste shortcut** `ctrl+shift+v` for users who dictate mostly into
terminals.

If anything fails here, switch to the `troubleshoot-hebrish-linux` skill.

## 5. Updates

- **App:** through APT like any other package: `sudo apt update && sudo apt upgrade`, or the
  desktop's software updater (Software Updater, Discover, GNOME Software). The app also offers
  in-app updates.
- **Models:** the app checks the selected model for a new version on a schedule and installs it
  by itself (Settings → Model updates turns that into an Update button instead). The old version
  is deleted once the new one has verified.
- If the user deletes `/etc/apt/sources.list.d/hebrish.sources`, it stays deleted across package
  upgrades (it is a conffile), and updates stop until it is written again (step 2c).

## 6. Uninstall

Show and confirm, then:

```bash
sudo apt purge hebrish    # removes the app, the keyring and the repository's sources file
```

`sudo apt remove hebrish` removes the app but keeps the keyring and sources file (both are
conffiles), so the repository stays configured and a later `apt install hebrish` works again;
use `purge` to remove those too. Check for
leftovers (if the repository was added by hand before the package owned these files):

```bash
ls /etc/apt/sources.list.d/hebrish.sources /etc/apt/keyrings/hebrish-archive-keyring.gpg 2>/dev/null
# remove any that are listed, with consent:
# sudo rm /etc/apt/sources.list.d/hebrish.sources /etc/apt/keyrings/hebrish-archive-keyring.gpg
```

User data is not removed by `apt`. Ask before deleting it (this deletes the downloaded models
and the settings; no sudo):

```bash
grep -o '"models_dir": *"[^"]*"' ~/.config/hebrish/config.json   # a custom model folder, if one was chosen
rm -rf ~/.config/hebrish ~/.local/share/hebrish                  # settings; models, history, state
rm -f ~/.config/autostart/hebrish.desktop                        # only exists if Start at login was on
```

If `models_dir` was set, that folder holds the models too; delete it separately if the user
wants. Parlance's folders (`~/.config/parlance`, `~/.local/share/parlance`) belong to Parlance:
leave them.
