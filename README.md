# Hebrish Agent Skills

Skills that let an AI agent install, set up and troubleshoot [Hebrish](https://hebrish.app) for
you: on-device English dictation that writes Hebrew words correctly, in romanized form
(makolet, arnona, teudat zehut). Hebrish runs fine-tuned NVIDIA Parakeet models on your own
computer with sherpa-onnx; nothing is sent to a server. It is a personal
project by Daniel Rosehill.

The skills use the open [Agent Skills](https://agentskills.io) format (a folder with a
`SKILL.md`), so they work with Claude Code, Codex, Gemini CLI and any other agent that reads
skills, or with any agent that can read a web page.

## Skills

| Skill | What it does |
| --- | --- |
| [`install-hebrish-linux`](skills/install-hebrish-linux/SKILL.md) | Checks the system, adds the signed Hebrish APT repository (verifying its key), installs the `hebrish` package, walks through first-run setup (hotkey permission, choosing and downloading a model, hotkey), explains updates, and uninstalls. |
| [`troubleshoot-hebrish-linux`](skills/troubleshoot-hebrish-linux/SKILL.md) | Fixes the common problems: hotkey does nothing, text not pasted, no microphone, model download fails, no tray icon, APT errors. |
| [`transcribe-with-hebrish`](skills/transcribe-with-hebrish/SKILL.md) | Transcribes audio files (one, or a folder) on this computer through the Hebrish desktop app's local API (Settings → Local API, `http://127.0.0.1:8723`, OpenAI-compatible, no key, nothing leaves the machine). |
| [`update-hebrish`](skills/update-hebrish/SKILL.md) | Checks for and installs updates to the desktop app and its models, and switches model, through the local API. |
| [`read-hebrish-history`](skills/read-hebrish-history/SKILL.md) | Reads, searches and exports the desktop app's transcript history through the local API. |
| [`fix-hebrish-mistranscriptions`](skills/fix-hebrish-mistranscriptions/SKILL.md) | Finds what the desktop app keeps getting wrong, tests find-and-replace rules against real transcript history, and adds them to the personal dictionary. |
| [`sideload-hebrish-android`](skills/sideload-hebrish-android/SKILL.md) | Sideload: installs or updates the latest Hebrish APK on the Android phone you name, over adb (USB or wireless debugging), checking the download's SHA-256 and reporting the installed version. |
| [`setup-hebrish-fdroid-android`](skills/setup-hebrish-fdroid-android/SKILL.md) | Sets up the Hebrish F-Droid repository on an Android phone over adb (installing F-Droid if needed), so Hebrish installs and updates automatically. You confirm the prompts on the phone. |

**Supported:** Ubuntu 26.04 (amd64) for the desktop skills; Android 9+ on arm64, from any computer with adb, for the Android skills. Other Linux distributions and versions are not supported yet;
see [hebrish.app/download](https://hebrish.app/download) for every platform, including Android
and Windows.

The agent shows you each command that needs `sudo` and asks before running it.

## Use it

### Any agent: paste this prompt

```text
Install Hebrish on this computer using https://raw.githubusercontent.com/Hebrish-ASR/Hebrish-Agent-Skills/main/skills/install-hebrish-linux/SKILL.md
```

If something does not work afterwards:

```text
Hebrish isn't working on this computer. Troubleshoot it using https://raw.githubusercontent.com/Hebrish-ASR/Hebrish-Agent-Skills/main/skills/troubleshoot-hebrish-linux/SKILL.md
```

To transcribe recordings on this computer with Hebrish (the desktop app, with Settings → Local API on):

```text
Transcribe /path/to/recording.mp3 with Hebrish using https://raw.githubusercontent.com/Hebrish-ASR/Hebrish-Agent-Skills/main/skills/transcribe-with-hebrish/SKILL.md
```

To fix a word or phrase Hebrish keeps getting wrong (the desktop app, 1.018 or later, with Settings → Local API and its control option on):

```text
Hebrish keeps writing "My Word Prompts" for "My Weird Prompts". Fix it using https://raw.githubusercontent.com/Hebrish-ASR/Hebrish-Agent-Skills/main/skills/fix-hebrish-mistranscriptions/SKILL.md
```

The desktop app also installs its skills itself: Settings → Local API → Install agent skills, or `hebrish --install-agent-skills`.

To install Hebrish on an Android phone from this computer over adb (USB debugging or wireless debugging on):

```text
Sideload the latest Hebrish onto my phone using https://raw.githubusercontent.com/Hebrish-ASR/Hebrish-Agent-Skills/main/skills/sideload-hebrish-android/SKILL.md
```

```text
Set up the Hebrish F-Droid repository on my phone using https://raw.githubusercontent.com/Hebrish-ASR/Hebrish-Agent-Skills/main/skills/setup-hebrish-fdroid-android/SKILL.md
```

### Claude Code

```text
/plugin marketplace add Hebrish-ASR/Hebrish-Agent-Skills
/plugin install hebrish@hebrish-agent-skills
```

Then ask "install Hebrish". The skills appear as `/hebrish:install-hebrish-linux`,
`/hebrish:troubleshoot-hebrish-linux`, `/hebrish:sideload-hebrish-android` and so on.

### Codex, Gemini CLI and other skill-aware agents

Copy the skill folders into the agent's skills directory (see its documentation for the
location), for example:

```bash
git clone https://github.com/Hebrish-ASR/Hebrish-Agent-Skills
cp -r Hebrish-Agent-Skills/skills/* <your-agent-skills-directory>/
```

### No agent

Each `SKILL.md` is plain Markdown: you can follow it by hand. Each skill folder also has a
Hebrew version, `SKILL_HE.md`.

## What gets installed

- APT source `/etc/apt/sources.list.d/hebrish.sources` for `https://cdn.hebrish.app/apt`
  (suite `stable`, component `main`, amd64), signed by the key
  `4745 D635 77C8 1400 DBB7  43DF DBD1 F5A2 2CA1 B27B` ("Hebrish APT Repository
  <info@hebrish.app>"), installed as `/etc/apt/keyrings/hebrish-archive-keyring.gpg`.
- The `hebrish` package and its dependencies from Ubuntu.
- On first start, the model you choose (Sabich is recommended; about 660 MB, downloaded once).

## Links

- Website: <https://hebrish.app>
- App source and issues: <https://github.com/Hebrish-ASR/Hebrish-App>
- Report a problem with these skills: open an issue in this repository.

## License

[MIT](LICENSE). The Hebrish app and models have their own licences; see hebrish.app.
