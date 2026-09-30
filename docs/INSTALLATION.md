# NanoClip — Installation & Setup Guide

*by Techinfotics · v1.0.0*

This guide covers every way to install NanoClip, how to set up your AI model
with a free API key, and how to create your first clips. Pick the option that
fits you — most people want **Option A**.

---

## Contents

- [System requirements](#system-requirements)
- [Option A — Windows installer (recommended)](#option-a--windows-installer-recommended)
- [Option B — macOS app](#option-b--macos-app)
- [Option C — Docker / self-hosted web UI](#option-c--docker--self-hosted-web-ui)
- [Option D — From source (CLI / developers)](#option-d--from-source-cli--developers)
- [🔑 API key & model setup](#-api-key--model-setup)
- [🎬 Your first clips (walkthrough)](#-your-first-clips-walkthrough)
- [Troubleshooting](#troubleshooting)
- [Updating](#updating)
- [For maintainers: building releases](#for-maintainers-building-releases)

---

## System requirements

| | Minimum | Recommended |
|---|---|---|
| **OS** | Windows 10 x64 / macOS 13 (Apple Silicon) / any Docker host | Same |
| **RAM** | 8 GB | 16 GB |
| **Disk** | 4 GB free (+ space for your videos) | 10 GB free |
| **Internet** | Needed for AI analysis & model downloads | Needed |

> The desktop installers bundle Python and FFmpeg — you don't install
> anything else.

---

## Option A — Windows installer (recommended)

1. Go to **[Releases](https://github.com/muazforu/NanoClip/releases/latest)**.
2. Download **`NanoClip_1.0.0_x64-setup.exe`** (under *Assets*).
3. Double-click the installer and follow the steps.
4. **SmartScreen warning?** Windows may show *"Windows protected your PC"*
   because the installer isn't code-signed yet. This is normal for new
   open-source apps — click **More info → Run anyway**.
5. Launch **NanoClip** from the Start menu.
6. Continue to [🔑 API key & model setup](#-api-key--model-setup) below —
   the app needs a model before it can find highlights.

---

## Option B — macOS app

1. Go to **[Releases](https://github.com/muazforu/NanoClip/releases/latest)**.
2. Download the **`.dmg`** file (Apple Silicon / arm64).
3. Open it and drag **NanoClip** into **Applications**.
4. On first launch, right-click → **Open** if macOS asks for confirmation.
5. Continue to [🔑 API key & model setup](#-api-key--model-setup).

> Intel Macs aren't covered by the prebuilt app — use
> [Option C (Docker)](#option-c--docker--self-hosted-web-ui) or
> [Option D (source)](#option-d--from-source-cli--developers) instead.

---

## Option C — Docker / self-hosted web UI

For Linux servers, NAS boxes, or anyone who prefers the browser UI.
Requires [Docker](https://docs.docker.com/get-docker/) and Docker Compose v2.

```bash
git clone https://github.com/muazforu/NanoClip.git
cd NanoClip

cp env.example .env
# Edit .env: choose LLM_PROVIDER and set the matching API key + model name.
# (You can also configure this in Settings after startup.)

mkdir -p data logs uploads
docker compose up -d --build
```

Then open:
- 🖥️ Web UI: <http://localhost:3000>
- 📚 API docs: <http://localhost:8000/docs>

> **Linux permission fix:** if bind-mounted folders cause permission errors:
> ```bash
> docker compose run --rm --no-deps --user root --entrypoint sh nanoclip \
>   -c 'chown -R nanoclip:nanoclip /app/data /app/logs /app/uploads'
> docker compose up -d
> ```
> Inside Docker, `localhost` means the *container*. To reach a model running
> on your host (e.g. Ollama), use an address reachable from the container,
> such as `http://host.docker.internal:11434/v1`.

---

## Option D — From source (CLI / developers)

For batch processing, agents, or hacking on the code.
Requires **Python 3.10+** (3.11 recommended) and **FFmpeg** on your PATH.

```bash
git clone https://github.com/muazforu/NanoClip.git
cd NanoClip

python3 -m venv venv
source venv/bin/activate          # Windows: venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
python -m pip install -e .
```

Check everything is healthy:

```bash
autoclip doctor --provider gemini
```

Process a video (needs a configured model — see below):

```bash
autoclip run talk.mp4 --provider gemini --json
# add --srt talk.srt if you already have subtitles
```

Export a clip project as vertical Shorts (replace `PROJECT_ID` with the ID
printed by `run`):

```bash
autoclip export PROJECT_ID --preset shorts
```

Start the MCP server for AI agents:

```bash
autoclip mcp
```

---

## 🔑 API key & model setup

NanoClip's highlight AI needs a language model. You have three routes —
**pick one**.

### Route 1 — Google Gemini (free, easiest) ⭐

1. Go to **[aistudio.google.com/apikey](https://aistudio.google.com/apikey)**
   and sign in with any Google account.
2. Click **"Create API key"** → copy the key.
   - It's **free** with a generous quota — plenty for personal use.
   - Google may ask you to accept the API terms; that's standard.
3. In NanoClip, open **Settings → Model**:
   - **Provider:** `Google Gemini`
   - **API key:** paste your key
   - **Model:** `gemini-flash-lite-latest` *(recommended — fast & cheap)*
   - Click **Test connection** → you should see success
   - Click **Save**
4. Done! Highlight detection, scoring and titles now use Gemini.

> 💡 **Model tip:** `gemini-flash-lite-latest` is the tested, reliable
> choice. `gemini-2.5-flash` can return errors for brand-new API keys —
> stick with flash-lite unless you know you need otherwise.

### Route 2 — Fully local, no key (Ollama / LM Studio)

Prefer zero cloud? Run the model on your own machine:

1. Install [Ollama](https://ollama.com) (or
   [LM Studio](https://lmstudio.ai)), download a model, e.g.:
   ```bash
   ollama pull qwen2.5:7b
   ```
2. In NanoClip **Settings → Model** choose the **Ollama** provider
   (default endpoint `http://localhost:11434/v1`, model `qwen2.5:7b`).
3. Test connection → Save. No API key needed.

> Needs a decent machine (16 GB RAM recommended) and model downloads are
> several GB — but everything stays 100% on your device.

### Route 3 — Any OpenAI-compatible provider

Have another API (OpenAI, DeepSeek, a proxy service…)? Choose the
**OpenAI-compatible** provider in Settings, enter its **Base URL**, your
**API key**, and the **model name**, then test & save.

### 🔒 Where does my key go?

- Stored **only** in NanoClip's settings on **your device**.
- Sent **only** to the provider you selected (e.g. Google's API).
- **Never** uploaded to Techinfotics, GitHub, or anywhere else.
- Your **videos** are never sent to the cloud — only transcript *text*
  goes to the AI for analysis. Cutting happens locally.

### 🗣️ What about transcription (subtitles)?

Transcription is separate from the AI model and works **offline**:
- First time you process a video without subtitles, NanoClip downloads the
  local Whisper speech model once (~a few hundred MB), then transcribes on
  your machine.
- Or import your own `.srt` subtitle file to skip transcription entirely.

---

## 🎬 Your first clips (walkthrough)

1. **Import** — Click *New project*, add a video file (or paste a YouTube /
   Bilibili link). A 3–5 minute video is perfect for the first try.
2. **Transcribe** — If there's no subtitle file, NanoClip transcribes
   automatically. Good subtitles = better highlights.
3. **Analyze** — AI scores every moment and proposes clips with titles.
   Adjust the score threshold (start at **0.7**; lower to **0.5** if you get
   too few clips).
4. **Review** — Open the Studio, check clip boundaries and titles, tweak
   anything.
5. **Export** — Pick a preset (e.g. *TikTok 9:16*), enable burned-in
   subtitles + title card, and export. Or connect an account and **Publish**.

---

## Troubleshooting

| Problem | Fix |
|---|---|
| *"Windows protected your PC"* during install | Click **More info → Run anyway** (installer isn't code-signed yet) |
| Model test connection fails | Check the key is pasted correctly (no spaces); confirm the model name; check internet |
| `gemini-2.5-flash` returns 404 | Known issue for new API keys — use `gemini-flash-lite-latest` |
| No clips generated | Lower score threshold 0.7 → 0.5; check transcription isn't empty; verify model connection |
| Transcription is slow / fails | First run downloads the Whisper model — needs internet & disk space; or import an `.srt` |
| Docker permission errors (Linux) | Run the `chown` fix in [Option C](#option-c--docker--self-hosted-web-ui) |
| Port already in use | Change ports in `docker-compose.yml` / `.env` |

Still stuck? Open an issue: <https://github.com/muazforu/NanoClip/issues/new>
— include your OS, app version, model/provider, and the error message
(**never** paste your API key).

---

## Updating

- **Desktop:** download the newer installer from
  [Releases](https://github.com/muazforu/NanoClip/releases/latest) and run it
  over the old version — your projects and settings are kept.
- **Docker:** `git pull && docker compose up -d --build`
- **Source:** `git pull && python -m pip install -r requirements.txt`

---

## For maintainers: building releases

Installers are built automatically by the **Desktop Build** workflow
(`.github/workflows/desktop-build.yml`):

- **Manual:** GitHub → *Actions* → *Desktop Build* → *Run workflow* →
  choose platforms → the `.exe` / `.dmg` appear as workflow artifacts.
- **Automatic release:** push a tag like `v1.0.0` and the workflow builds
  both platforms and attaches them to a GitHub Release.

---

<div align="center">

Made with ❤️ by **Techinfotics** · [MIT License](../LICENSE)

</div>
