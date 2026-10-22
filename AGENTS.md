# AGENTS.md (maintainers)

Public skills for installing and troubleshooting the Hebrish desktop app on Linux. Readers are
end users' agents, so every command must be runnable as written and every claim must match the
app.

## Layout

- `skills/<name>/SKILL.md`: Agent Skills format. Frontmatter is **only** `name` (= folder
  name), `description` (single-quoted; includes "Use when" and "Do NOT use for"), `license`,
  `compatibility`. No `metadata:` key: Skills IL (below) requires that, because Claude Desktop
  rejects it. Each skill is self-contained, because most agents fetch one raw `SKILL.md` URL and
  nothing else. Long reference material, if ever needed, goes in `skills/<name>/references/`.
- `skills/<name>/SKILL_HE.md`: Hebrew translation of the body, no frontmatter. Same `##`/`###`
  headings in the same order, all in Hebrew; code blocks byte-identical to `SKILL.md` and free
  of Hebrew. **Any change to a `SKILL.md` body goes into its `SKILL_HE.md` in the same commit.**
- `skills/<name>/metadata.json`: Skills IL metadata (`author`, `version`, `category`
  `localization`, bilingual `tags` of equal length, `display_name`, `display_description`,
  `supported_agents`). Bump `version` when the skill changes; this replaced the frontmatter
  `updated` date on 2026-10-21.
- `.github/workflows/release.yml`: a `v*` tag creates a GitHub release with a Sigstore
  provenance attestation (Skills IL `signed_release` signal). Tag after a skill change.
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

If any of these change (key rotation, new suite, new arch), update both skills (and their
`SKILL_HE.md`) and the README in the same commit, and bump `version` in each changed skill's
`metadata.json`.

## Copies elsewhere

- hebrish.app serves byte-identical copies of each `SKILL.md` at
  `/.well-known/agent-skills/<name>/SKILL.md` (Hebrish-App repo; README there says how to
  recompute the `index.json` digest). Copy them over after every change.
- Skills IL (<https://agentskills.co.il>, GitHub org `skills-il`, run by Yootech) lists skills
  from this repo, submitted through its form as an external repository. It checks: the
  `validate-skill.sh` rules
  (<https://raw.githubusercontent.com/skills-il/developer-tools/master/scripts/validate-skill.sh>;
  run it on `skills/*/SKILL.md`, it needs PyYAML), `gh skill publish --dry-run`, secret
  scanning, CodeQL default setup, a signed release and a detected LICENSE
  (<https://agentskills.co.il/en/guides/github-verification-checklist>). Rules read
  2026-10-21.
- **Submitted 2026-10-21**, both skills, through <https://agentskills.co.il/en/submit>, tab
  "I Already Have a Skill" (needs Sign in with GitHub; the `skills-il` OAuth app, email
  read-only, is authorized on `danielrosehill`). One submission per skill: paste the skill
  *folder* URL (`.../tree/main/skills/<name>`), "Fetch SKILL.md", then fill the form. Values
  used: category Localization, creator `danielrosehill`, email info@hebrish.app, the eight
  agents in `metadata.json`, Hebrew name/description/tags copied from `metadata.json`. Status:
  awaiting their review.
- What the form actually does (observed 2026-10-21): it does **not** read `metadata.json`; it
  suggests `metadata.display_name.he` etc. in the frontmatter, which their own creator skill
  forbids. Ignore that hint and type the Hebrew fields in. Its repo checks reported 1/5
  (only the licence) although secret scanning, push protection, CodeQL default setup (language
  `actions`, first analysis 18:46 UTC) and a v1.1.1 attestation were all confirmed through the
  GitHub API; "spec compliant" showed as pending. The form still accepted the submission. If
  the review asks about these, point them at the repo settings and the v1.1.1 release.

## Writing rules

- Hebrish is Daniel Rosehill's one-person project: never "we"/"our", never "non-profit" or "not-for-profit" (Daniel, 2026-10-22).
- The skills tell the agent to show and ask before every `sudo` command; keep that.
- Unknown problems go to https://github.com/Hebrish-ASR/Hebrish-App/issues, not to guesses.
