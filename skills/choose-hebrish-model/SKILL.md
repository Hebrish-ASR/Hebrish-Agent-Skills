---
name: choose-hebrish-model
description: 'Pick the right Hebrish speech model (hebrish.app) for a user from the live model inventory at https://hebrish.app/data/model-inventory.json: which models are maintained, what each is for (dictation, streaming, voice commands, text to speech), which platforms it runs on, whether the Hebrish apps offer it, and its latest version. Use when a user asks which Hebrish model to use, whether a model is still maintained or up to date, what Sabich, Shakshuka, Falafel, Jachnun, Meorav or a Sabich variant is for, or which model fits their phone, computer, GPU or wearable. Do NOT use to install the app (install-hebrish-linux, sideload-hebrish-android), to update or switch the model in an installed desktop app (update-hebrish), or for questions about accuracy numbers (point to https://hebrish.app/benchmarks).'
license: MIT
compatibility: Any agent that can fetch a URL (curl or a web fetch tool). Needs internet.
---

# Choose a Hebrish model

Hebrish is Daniel Rosehill's personal project (hebrish.app): speech models for English with Hebrew
words mixed in. There are many Hebrish models on Hugging Face, and most are older ones kept for
reference. This skill reads the **model inventory**, the one list that says which models are
maintained, and picks from it.

## Rules for the agent

- Always read the live inventory; never answer from memory. Models are retrained often.
- Recommend only models whose `support` is `recommended`, unless the user's need rules them all out.
- Every model can still be downloaded, and access requests for the older ones are still approved.
  Say so if the user asks for one, but strongly recommend the up-to-date ones: they are still being
  trained, so they recognise the widest vocabulary.
- A model with `support` `related` (Fritter, Gefilte) is a separate project, not a Hebrish model.
  Mention it only if the user wants English without Hebrew (Fritter) or Hebrew script (Gefilte).
- Do not quote accuracy figures. For results, link https://hebrish.app/benchmarks.

## 1. Read the inventory

```bash
curl -s https://hebrish.app/data/model-inventory.json
```

Each entry in `models` has:

| Field | Meaning |
| --- | --- |
| `support` | `recommended` (maintained, use these), `available` (download as is, updated rarely or never), `legacy` (replaced, see `successor`), `related` (separate project) |
| `state` | `active`, `work-in-progress`, `occasional`, `frozen`, `proof-of-concept`, `superseded` |
| `use_case` | `dictation`, `streaming-dictation`, `voice-commands`, `tts`, `accessory` |
| `platforms` | `android`, `linux`, `windows`, `macos`, `browser`, `gpu`, `low-power` |
| `in_apps` | where the Hebrish apps offer it: `android`, `desktop`, or empty |
| `current_version`, `last_release` | latest version and its date |
| `successor` | the model to use instead, for older ones |
| `repo`, `page` | Hugging Face repository and hebrish.app page |

## 2. Ask only what you need

If the user has not said, ask:

1. What for: typing by voice (dictation), seeing words while speaking (streaming), short commands to a
   computer or phone (voice commands), or reading text aloud (text to speech)?
2. Where: Android phone, Linux or Windows computer, Mac, a GPU server, or a small device?
3. In the Hebrish app, or in their own tools (sherpa-onnx, NeMo, transcribe.cpp, llama.cpp)?

## 3. Pick

Filter `models` by `use_case` and `platforms`, keep `support` = `recommended`, and prefer one whose
`in_apps` covers the user's device if they use the Hebrish app. Typical outcomes:

- Dictation on a phone or computer: **Sabich**, the default in the apps.
- Streaming dictation on Linux or Windows: recommend **Sabich**. Shakshuka (desktop app) shows words while you speak, but it is less accurate and is not recommended yet; mention it only if streaming matters more than accuracy.
- Voice commands: **Falafel** (0.6B on desktops, Falafel 110M on phones and laptops).
- Text to speech: **Meorav**, run with the user's own tools; the Hebrish apps have no text to speech.

If the user names an older model, give its `state` and `successor` and recommend the successor.

## 4. Answer

Give the model name, why it fits, its `current_version` and `last_release`, and its links (`page`
and `repo`). To get the apps: https://hebrish.app/download. The full table:
https://hebrish.app/models/inventory.
