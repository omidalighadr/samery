# Samery 🎵

> A private, single-user Telegram music companion — search, download, and enjoy high-quality audio from many sources, with an optional AI chat brain.

<p align="center">
  <img alt="Python" src="https://img.shields.io/badge/Python-3.11%2B-3776AB?logo=python&logoColor=white">
  <img alt="python-telegram-bot" src="https://img.shields.io/badge/python--telegram--bot-21.x-2CA5E0?logo=telegram&logoColor=white">
  <img alt="Platform" src="https://img.shields.io/badge/runs%20on-macOS%20%C2%B7%20Linux-lightgrey">
  <img alt="Status" src="https://img.shields.io/badge/status-personal%20project-success">
  <img alt="Use" src="https://img.shields.io/badge/scope-personal%20use%20only-orange">
</p>

---

## Overview

**Samery** is a personal Telegram bot that turns a single chat into a full music workflow: paste a link or a song name and it finds the best available audio, tags it, embeds cover art and lyrics, and delivers it straight back to you. It also doubles as a warm AI chat companion and a small audio toolbox (voice → MP3, voice → text).

It is built to be **private by design** — it answers only its owner, keeps everything on the machine it runs on, and never exposes a public interface.

## Features

### 🎧 Music
- **Multi-source search & download** — resolves a song name or a link across several providers and returns the best available quality.
- **Direct-link grabber** — paste a link from most download sites (Iranian or international); it tries `yt-dlp` first, then falls back to scraping a direct audio link off the page.
- **Playlist / album support** — expands a set into its individual tracks.
- **Forward-to-fetch** — forward any audio file and the bot identifies it from its tags and finds a clean copy, so you never have to retype the name.
- **Rich delivery** — real album covers (not thumbnails), embedded metadata, and lyrics rendered inside the message caption.
- **Channel workflow** — one-tap **approve / reject** buttons to publish a track to your own channel, with duplicate detection.
- **Personal archive** — a searchable local library of everything you've downloaded, with a Telegram `file_id` cache for instant re-sends.

### 🤖 AI companion
- **Chat mode** — a toggleable conversational assistant with memory of the conversation.
- **Pluggable brain** — works with Google Gemini out of the box, or **any OpenAI-compatible endpoint** (just change a few environment variables — no code changes).
- **Music inside chat** — ask for a song mid-conversation and it fetches it without leaving chat mode (function/tool calling).
- **Automatic retry & backoff** on rate limits, so quiet moments don't turn into errors.

### 🛠️ Audio tools
- **Voice / audio → MP3** — send a voice message or clip and get a clean MP3 back.
- **Voice → text** — one-tap transcription of any voice note or audio file.

### ⏰ Automations
- **Daily suggestion** — a track picked each morning from your favourite artists.
- **Nightly check-in** — a gentle "how are you?" with a one-tap reply that sends comforting music.

## Architecture

Samery uses a small **provider architecture**: every source implements a common `search()` / `tracks()` interface, so new sites can be added in isolation without touching the core.

```
samery/
├── bot.py              # Telegram handlers, chat brain, wiring
├── config.py           # environment-driven configuration
├── downloader.py       # resumable download, integrity checks, tagging
├── enrich.py           # cover art (iTunes / Cover Art Archive) + lyrics (LRCLIB)
├── library.py          # SQLite: downloads, sent-cache, enrich-cache, posted
└── providers/
    ├── base.py         # Track / Release data model + Provider protocol
    ├── ytdlp.py        # SoundCloud / YouTube (+ playlists)
    ├── generic.py      # catch-all link grabber (yt-dlp → page scrape)
    ├── musicdel.py     # example direct-download site adapter
    └── ...             # additional lossless sources
```

**Design principles**
- One responsibility per module; providers are independent and hot-swappable.
- The download pipeline verifies integrity, resumes interrupted transfers, and never overwrites existing tags.
- Every network call degrades gracefully with clear, human-readable errors.

## Tech stack

| Area | Tools |
|---|---|
| Bot framework | `python-telegram-bot` (async) |
| Media | `yt-dlp`, `ffmpeg`, `mutagen` |
| Metadata | iTunes Search API, Cover Art Archive, LRCLIB, MusicBrainz |
| Storage | SQLite |
| AI | Google Gemini / any OpenAI-compatible API |
| Runtime | Python 3.11+, runs as a background service (`launchd` / `systemd`) |

## Getting started

### Prerequisites
- Python 3.11+
- [`ffmpeg`](https://ffmpeg.org/)
- A Telegram bot token from [@BotFather](https://t.me/BotFather)

### Installation

```bash
git clone https://github.com/omidalighadr/samery.git
cd samery
python3 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
```

### Configuration

Copy the example environment file and fill in your values:

```bash
cp .env.example .env
```

| Variable | Description |
|---|---|
| `TELEGRAM_BOT_TOKEN` | Your bot token from BotFather |
| `ALLOWED_USER_IDS` | Your numeric Telegram user id (the bot answers only you) |
| `CHANNEL_ID` | (Optional) channel to publish approved tracks to |
| `LLM_PROVIDER` | `gemini` or `openai` |
| `GEMINI_API_KEY` / `OPENAI_*` | Credentials for the chosen chat brain |

### Run

```bash
./run.sh          # or: python bot.py
```

## Commands

| Command | What it does |
|---|---|
| `/search <query>` | Find a track by name |
| `/chat` | Toggle the AI chat companion on/off |
| `/library <text>` | Search your downloaded archive |
| `/lyrics <artist - title>` | Fetch lyrics only |
| `/suggest` | Send a random suggestion now |
| `/stats`, `/recent` | Archive stats and recent downloads |

Plain messages, links, forwarded audio, and voice notes are all handled automatically.

## Notes & limitations

- Telegram bots can upload files up to **50 MB** and download up to **20 MB** via the Bot API; larger files are saved locally instead.
- Audio from lossy platforms (SoundCloud/YouTube) is only as good as the source — there is no lossless where the source was never lossless.
- The AI features depend on the configured provider's availability and quota.

## Disclaimer

Samery is a **personal-use** project for managing a private music library. It is not intended for redistribution of copyrighted material or any commercial use. Respect the terms of service of the platforms you access and the rights of content owners.

## License

Released under the MIT License. See [`LICENSE`](LICENSE) for details.

---

<p align="center"><sub>Built as a personal project · maintained by <a href="https://github.com/omidalighadr">@omidalighadr</a></sub></p>
