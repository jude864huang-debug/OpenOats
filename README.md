# OpenOats

A meeting note-taker that talks back.

## Interview Copilot MVP

This fork adds a macOS copilot for remote product and business interviews. System audio is treated
as the interviewer and microphone audio as the candidate. It shows short answer cues first, never
speaks for the candidate, and does not save raw audio.

Main answers prefer Terra. With an OpenAI API key, the formally supported path uses the public
Responses API. The Codex subscription and Spark path remains an experimental compatibility mode
that requires the user to sign in locally and depends on account access. If Terra fails before a
usable opening appears, the rest of that interview stays on Spark. **Running Details** records the
actual model, fallback reason, and timing for each answer.

### Development setup

```bash
cd worker
npm install
codex login

cd ../OpenOats
swift run OpenOats
```

In **Settings → Copilot**:

1. Choose one role folder containing Markdown, TXT, text-based PDF, or DOCX files.
2. Organize it into `01-resume`, `02-story-bank`, `03-job-description`, `04-company`, and
   `05-domain`. Only the first two categories may support claims about the candidate.
3. Configure Tencent Cloud standard realtime ASR (`16k_zh`). AppID is stored in app settings;
   SecretID and SecretKey are kept in macOS Keychain. The installed local Qwen3-ASR 0.6B is
   used once as a slow whole-turn fallback when Tencent fails. The standard engine is selected
   by default so usage can be deducted from Tencent's realtime-ASR free quota; check the Tencent
   console for the current quota and disable postpaid billing if you want a hard cost ceiling.
4. Choose **API preferred** for low-latency OpenAI Responses API generation, or the experimental
   **Codex Pro only** compatibility path. API keys are kept in macOS Keychain. Without a key, the
   app can use a locally signed-in Codex subscription, subject to the account's model access.
   **Codex compact cue** is enabled by default: it prewarms a persistent worker and sends a short
   source brief. The cue model is editable and is passed unchanged to the local Codex CLI; Spark is
   only the default, not a hard-coded model.
5. Wait for the interview material package to show `Ready`, then start a session.

The complete compiled package remains local. Generation requests use a configurable compact brief,
defaulting to approximately 8k tokens, with explicit budget shares for resume, story bank, JD,
company, and domain sources. This prevents one long document from crowding every other category out
of the prompt and does not introduce vector retrieval.

The primary turn shortcut defaults to `⌃⌥G` and is configurable. Press it once when the
interviewer finishes, then again when the candidate finishes. `⌃⌥M` merges the previous
interviewer segment and `⌃⌥S` stops only the current answer generation. Transcripts and generated
cues are stored locally; raw audio is not stored. Tencent ASR and the OpenAI API are billed
separately from ChatGPT Pro.

The first response contains a direct opening, 3–5 talking points, evidence anchors, and likely
follow-ups. If no personal example is supported by the resume or story bank, it still gives a
professional method, decision process, trade-offs, or an explicit hypothetical answer instead of
showing a generic “missing facts” warning. With the OpenAI API, a 45–90 second structured reference
answer starts automatically after the fast cue; the Codex subscription path keeps that long answer
manual so it cannot block the next question.
Use the app only when the interview rules and applicable recording laws allow it. This fork does
not add screen-share hiding or monitoring-evasion features.

For a local app bundle, first run `npm install` in `worker/`, then:

```bash
SKIP_SIGN=1 SKIP_INSTALL=1 ./scripts/build_swift_app.sh
```

Maintainer releases are intentionally stricter than local builds. GitHub Actions installs the
locked worker dependencies with `npm ci`, runs the worker and Swift checks, then requires a
Developer ID Application certificate, Apple notarization credentials, and separate Sparkle
`SPARKLE_EDDSA_KEY` / `SPARKLE_PUBLIC_ED_KEY` repository secrets. Missing runtime files or release
credentials stop the workflow before a DMG is published; day-to-day local development is unchanged.

<p align="center">
  <a href="https://github.com/jude864huang-debug/OpenOats/releases/latest">
    <img src="https://img.shields.io/badge/Download_for_Mac-DMG-black?style=for-the-badge&logo=apple&logoColor=white" alt="Download for Mac" />
  </a>
</p>

OpenOats sits next to your call, transcribes both sides of the conversation in real time, and searches your own notes to surface things worth saying — right when you need them.

---

### Sponsored by Recall.ai — API for desktop recording

If you're looking for a hosted desktop recording API, consider checking out [Recall.ai](https://dub.sh/openoats), an API that records Zoom, Google Meet, Microsoft Teams, in-person meetings, and more.

---

<p align="center">
  <img src="assets/hero.svg" width="720" alt="OpenOats during a call — suggestions drawn from your own notes appear at the top, live transcript below" />
</p>

## Features

- **Visible floating copilot** — this fork does not enable screen-share hiding or monitoring evasion
- **Role-aware interview transcription** — only the manually selected system-audio or microphone channel is sent to Tencent realtime ASR; local Qwen is the slow failure fallback
- **Runs 100% locally** — pair with [Ollama](https://ollama.com/) for LLM suggestions and local embeddings, and nothing touches the network at all
- **Pick any LLM** — use [OpenRouter](https://openrouter.ai/) for cloud models (GPT-4o, Claude, Gemini) or Ollama for local ones (Llama, Qwen, Mistral)
- **Live transcript** — see both sides of the conversation as it happens, copy the whole thing with one click
- **Auto-saved sessions** — every conversation is automatically saved as a plain-text transcript and a structured session log, no manual export needed
- **Knowledge base search** — point it at a folder of notes and it pulls in what's relevant using [Voyage AI](https://www.voyageai.com/) embeddings, local Ollama embeddings, or any OpenAI-compatible endpoint (llama.cpp, llamaswap, LiteLLM, vLLM, etc.)

## How it works

1. You start a call and hit **Live**
2. OpenOats captures both channels, but sends only the currently selected role to realtime ASR
3. When the conversation hits a moment that matters — a question, a decision point, a claim worth backing up — it searches your notes and surfaces relevant talking points
4. You sound prepared because you are

## Recording Consent & Legal Disclaimer

**Important:** OpenOats records and transcribes audio from your microphone and system audio. Many jurisdictions have laws requiring consent from some or all participants before a conversation may be recorded (e.g., two-party/all-party consent states in the U.S., GDPR in the EU).

**By using this software, you acknowledge and agree that:**

- **You are solely responsible** for determining whether recording is lawful in your jurisdiction and for obtaining any required consent from all participants before starting a session.
- **The developers and contributors of OpenOats provide no legal advice** and make no representations about the legality of recording in any jurisdiction.
- **The developers accept no liability** for any unauthorized or unlawful recording conducted using this software.

**Do not use this software to record conversations without proper consent where required by law.**

The app will ask you to acknowledge these obligations before your first recording session.

## Download

Install via Homebrew:

```bash
brew tap jude864huang-debug/openoats https://github.com/jude864huang-debug/OpenOats
brew install --cask jude864huang-debug/openoats/openoats
```

To upgrade later:

```bash
brew upgrade --cask jude864huang-debug/openoats/openoats
```

Or grab the latest DMG from the [Releases page](https://github.com/jude864huang-debug/OpenOats/releases/latest).

Or build from source:

```bash
./scripts/build_swift_app.sh
```

## Quick start

1. Open the DMG and drag OpenOats to Applications
2. Launch the app and grant microphone + system audio recording permissions
3. Open Settings (`Cmd+,`) and pick your providers:
   - **Cloud**: add your OpenRouter and Voyage AI API keys
   - **Local**: select Ollama as your LLM and embedding provider (make sure Ollama is running)
   - **OpenAI-compatible**: select "OpenAI Compatible" as your embedding provider and point it at any `/v1/embeddings` endpoint
4. Point it at a folder of `.md` or `.txt` files — that's your knowledge base
5. Click **Idle** to go live

The first run downloads the local speech model (~600 MB).

## What you need

- Apple Silicon Mac, macOS 15+
- Xcode 26 / Swift 6.2
- **For cloud mode**: [OpenRouter](https://openrouter.ai/) API key + [Voyage AI](https://www.voyageai.com/) API key
- **For local mode**: [Ollama](https://ollama.com/) running locally with your preferred models (e.g. `qwen3:8b` for suggestions, `nomic-embed-text` for embeddings)
- **For OpenAI-compatible embeddings**: any server implementing `/v1/embeddings` (llama.cpp, llamaswap, LiteLLM, vLLM, etc.)

## Knowledge base

Point the app at a folder of Markdown or plain text files. That's it. OpenOats chunks, embeds, and caches them locally. When the conversation shifts, it searches your notes and only surfaces what's actually relevant.

Works well with meeting prep docs, research notes, pitch decks, competitive analysis, customer briefs — anything you'd want at your fingertips during a call.

## Privacy

- In Interview Copilot manual mode, the currently selected role's audio and hotwords are sent to Tencent Cloud ASR
- In the experimental GPT Realtime mode, both interview audio roles are sent to OpenAI
- Raw audio is held only in bounded memory for the current turn and is not saved to disk
- **With Ollama**: everything stays on your machine. Zero network calls.
- **With cloud providers**: KB chunks are sent to Voyage AI (or your chosen OpenAI-compatible endpoint) for embedding (text only, no audio), and conversation context is sent to OpenRouter for suggestions
- API keys are stored in your Mac's Keychain
- The interview Copilot does not enable screen-share hiding or monitoring-evasion behavior
- Transcripts are saved locally to `~/Documents/OpenOats/`

### Cloud mode: what data leaves your Mac

When using Interview Copilot, audio is sent only to the selected ASR provider as described above.
Knowledge-package and transcript text may additionally be sent to the selected generation provider.
The legacy knowledge-base features below have their own provider-specific network behavior.

#### 1. Knowledge base indexing — Voyage AI (`api.voyageai.com/v1/embeddings`)

**When:** Each time you index your knowledge base folder (on launch or when files change).

**What is sent:**
- Text chunks from your `.md` / `.txt` knowledge base files (split by markdown headings, 80–500 words each, with the header breadcrumb prepended)
- Model name (`voyage-4-lite`) and requested output dimensions (`256`)
- Input type (`document`)

Chunks are sent in batches of 32. Only new or changed files are embedded — unchanged files use a local cache.

#### 2. Knowledge base search — Voyage AI (`api.voyageai.com/v1/embeddings`)

**When:** Each time the suggestion pipeline runs (triggered by a substantive utterance from the other speaker, subject to a 90-second cooldown).

**What is sent:**
- 1–4 short query strings derived from the conversation: the latest utterance text, the current conversation topic, a short conversation summary, and the top open question
- Model name, dimensions, and input type (`query`)

#### 3. Knowledge base reranking — Voyage AI (`api.voyageai.com/v1/rerank`)

**When:** Immediately after step 2, if Voyage AI is the embedding provider.

**What is sent:**
- The primary search query (the latest utterance text)
- Up to 10 candidate KB chunk texts (from your own notes) for reranking
- Model name (`rerank-2.5-lite`)

#### 4. Conversation state update — OpenRouter (`openrouter.ai/api/v1/chat/completions`)

**When:** Periodically during a session when the conversation state needs refreshing.

**What is sent (as an LLM prompt):**
- The previous conversation state (topic, summary, open questions, tensions, recent decisions, goals — all derived from earlier LLM calls)
- Recent transcript utterances (both speakers, text only — labeled "You" / "Them")
- The latest utterance from the other speaker
- A system prompt instructing the model to update the conversation state

#### 5. Surfacing gate — OpenRouter (`openrouter.ai/api/v1/chat/completions`)

**When:** After the KB search returns relevant results, to decide whether a suggestion is worth showing.

**What is sent (as an LLM prompt):**
- The latest utterance from the other speaker
- Recent transcript exchange (both speakers, text only)
- Current conversation state (topic, summary, open questions, tensions)
- The detected trigger type and excerpt
- Up to 5 KB evidence chunks (text from your notes, with source file and header, plus relevance scores)
- Recently shown suggestion angles (short strings, to avoid repeats)

#### 6. Suggestion generation — OpenRouter (`openrouter.ai/api/v1/chat/completions`)

**When:** Only if the surfacing gate approves (all quality scores above threshold).

**What is sent (as an LLM prompt):**
- The latest utterance from the other speaker
- Current conversation state (topic and summary)
- The gate's reasoning string
- Up to 3 KB evidence chunks (text from your notes, with source file and header)

#### 7. Meeting notes generation — OpenRouter (`openrouter.ai/api/v1/chat/completions`)

**When:** When you click "Generate Notes" after a session.

**What is sent (as an LLM prompt):**
- The full session transcript (both speakers, with timestamps, labeled "You" / "Them") — truncated to ~60,000 characters if very long
- The meeting template's system prompt (e.g., instructions for formatting notes)

#### What is never sent

- **Audio** — transcription is always on-device via Apple Speech
- **File paths or filenames from your system** (only KB source filenames appear in prompts)
- **Your API keys to anyone other than the respective provider** (OpenRouter key to OpenRouter, Voyage key to Voyage)
- **Any data when using Ollama** — all requests go to your local machine

## Build

```bash
# Full build → sign → install to /Applications
./scripts/build_swift_app.sh

# Dev build only
cd OpenOats && swift build -c debug

# Package DMG
./scripts/make_dmg.sh
```

Optional env vars for code signing and notarization: `CODESIGN_IDENTITY`, `APPLE_ID`, `APPLE_TEAM_ID`, `APPLE_APP_PASSWORD`.

## Repo layout

```
OpenOats/             SwiftUI app (Swift Package)
scripts/              Build, sign, and package scripts
assets/               Screenshot and app icon source
```

## License

MIT
