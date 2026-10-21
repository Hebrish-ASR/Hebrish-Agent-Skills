# Hebrish Agent Skills

Skills that let an AI agent install, set up and troubleshoot [Hebrish](https://hebrish.app) for
you: on-device English dictation that writes Hebrew words correctly, in romanized form
(makolet, arnona, teudat zehut). Hebrish runs fine-tuned NVIDIA Parakeet models on your own
computer with sherpa-onnx; nothing is sent to a server. It is a one-person, not-for-profit
project by Daniel Rosehill.

The skills use the open [Agent Skills](https://agentskills.io) format (a folder with a
`SKILL.md`), so they work with Claude Code, Codex, Gemini CLI and any other agent that reads
skills, or with any agent that can read a web page.

## Skills

| Skill | What it does |
| --- | --- |
| [`install-hebrish-linux`](skills/install-hebrish-linux/SKILL.md) | Checks the system, adds the signed Hebrish APT repository (verifying its key), installs the `hebrish` package, walks through first-run setup (hotkey permission, choosing and downloading a model, hotkey), explains updates, and uninstalls. |
| [`troubleshoot-hebrish-linux`](skills/troubleshoot-hebrish-linux/SKILL.md) | Fixes the common problems: hotkey does nothing, text not pasted, no microphone, model download fails, no tray icon, APT errors. |

**Supported:** Ubuntu 26.04 (amd64). Other Linux distributions and versions are not supported yet;
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

### Claude Code

```text
/plugin marketplace add Hebrish-ASR/Hebrish-Agent-Skills
/plugin install hebrish@hebrish-agent-skills
```

Then ask "install Hebrish". The skills appear as `/hebrish:install-hebrish-linux` and
`/hebrish:troubleshoot-hebrish-linux`.

### Codex, Gemini CLI and other skill-aware agents

Copy the skill folders into the agent's skills directory (see its documentation for the
location), for example:

```bash
git clone https://github.com/Hebrish-ASR/Hebrish-Agent-Skills
cp -r Hebrish-Agent-Skills/skills/* <your-agent-skills-directory>/
```

### No agent

Each `SKILL.md` is plain Markdown: you can follow it by hand.

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
