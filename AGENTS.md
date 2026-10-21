# AGENTS.md (maintainers)

Public skills for installing and troubleshooting the Hebrish desktop app on Linux. Readers are
end users' agents, so every command must be runnable as written and every claim must match the
app.

## Layout

- `skills/<name>/SKILL.md`: Agent Skills format (YAML frontmatter `name`, `description`,
  `metadata`; `name` = folder name). Each skill is self-contained, because most agents fetch one
  raw `SKILL.md` URL and nothing else. Long reference material, if ever needed, goes in
  `skills/<name>/references/`.
- `.claude-plugin/marketplace.json` (marketplace `hebrish-agent-skills`) lists one plugin,
  `hebrish`, with `source: "./"` (this repo root); `.claude-plugin/plugin.json` must keep
  `"name": "hebrish"`. Install id: `hebrish@hebrish-agent-skills`. Bump `version` in
  `plugin.json` when a skill changes so installed copies update.
- `README.md` holds the ready-to-paste prompts with raw GitHub URLs: renaming a skill folder
  breaks them.

## Sources of truth

Read these before changing a skill; do not add commands, flags, paths or settings they do not
support:

- Hebrish-Dictation (`github.com/Hebrish-ASR/Hebrish-App`): `desktop/README.md`,
  `desktop/docs/maintaining.md`, `desktop/packaging/debian/` (README.Debian, the udev rule
  `70-hebrish-uinput.rules`, postinst), `desktop/packaging/apt/` (`hebrish.sources`, keyring),
  `desktop/build-deb.sh` (Depends, Conflicts, conffiles), `desktop/hebrish/config.py`
  (`DEFAULTS`, paths), `shared/catalogue.json` (models; the desktop app runs only engine
  `sherpa_offline_transducer`; `recommended` is `sabich`).
- hebrish.app `download.html` (#ubuntu) for the public wording.

## Must stay in sync with the APT repository

- Repository URL `https://cdn.hebrish.app/apt`, suite `stable`, component `main`, arch `amd64`.
- Key fingerprint `4745D63577C81400DBB743DFDBD1F5A22CA1B27B`, keyring paths
  `https://cdn.hebrish.app/apt/hebrish-archive-keyring.{gpg,asc}` and
  `/etc/apt/keyrings/hebrish-archive-keyring.gpg`.
- The `hebrish.sources` block in `install-hebrish-linux` must be byte-identical to
  `desktop/packaging/apt/hebrish.sources`: the package (0.6.2+) ships it as a conffile, and a
  differing file makes dpkg prompt.
- `setup.sh` behaviour (keyring, sources file, `apt-get update`, `apt-get install hebrish`).
- Supported release: Ubuntu 26.04 amd64 (python3 3.14 pin in Depends). When a new Ubuntu release
  or architecture is supported, update both skills' support tables and the README.

If any of these change (key rotation, new suite, new arch), update both skills and the README in
the same commit, and update the `updated` date in each changed skill's frontmatter.

## Writing rules

- Hebrish is Daniel Rosehill's one-person, not-for-profit project: never "we"/"our".
- The skills tell the agent to show and ask before every `sudo` command; keep that.
- Unknown problems go to https://github.com/Hebrish-ASR/Hebrish-App/issues, not to guesses.
