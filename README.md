# Samery 🎵

> A private, single-user Telegram bot that finds and archives music at the best quality a source offers, pulls video and transcripts out of almost any link, talks back through a language model, and checks in every night.

<p align="center">
  <img alt="Python" src="https://img.shields.io/badge/Python-3.11%2B-3776AB?logo=python&logoColor=white">
  <img alt="python-telegram-bot" src="https://img.shields.io/badge/python--telegram--bot-21.x-2CA5E0?logo=telegram&logoColor=white">
  <img alt="Platform" src="https://img.shields.io/badge/runs%20on-macOS%20%C2%B7%20Linux-lightgrey">
  <img alt="Status" src="https://img.shields.io/badge/status-personal%20project-success">
  <img alt="Use" src="https://img.shields.io/badge/scope-personal%20use%20only-orange">
</p>

---

## Overview

**Samery** is a personal Telegram bot that turns a single chat into a full media workflow. Send a song name or a link and it finds the best available audio, tags it, embeds cover art and lyrics, and delivers it back. Send almost any video link and it downloads the video (or just the audio), pulls the transcript, summarises it, translates it, or answers questions about it. It also works as an AI chat companion and a small audio toolbox.

It is **private by design** — it answers only its owner, keeps everything on the machine it runs on, and never exposes a public interface.

**27 commands · 14 sources · Audio · Video · Text · Images · runs on my own Mac**

## Features

### 🎧 Music
- **Search by name** — send a song title; it searches YouTube first, then offers a button to pull the same track from freely-licensed sources instead.
- **Any link** — YouTube, SoundCloud, Spotify, musicdel.ir, Pinterest, and any other download site `yt-dlp` can read.
- **Whole albums & playlists** — paste a set link and it walks the entire thing, labelling each file (e.g. track 4 of 12).
- **Forward-to-fetch** — forward any audio file and the bot identifies it from its tags and finds a cleaner, higher-quality copy.
- **Covers, lyrics & dates** — tags files with artwork, lyrics and release dates from LRCLIB, MusicBrainz and the Cover Art Archive.
- **Searchable archive** — every download lands in a local database you can search, browse, and pull recent items from.
- **Favourites** — a star button under every track; starred songs come back instantly from Telegram's cache without re-downloading.
- **Trim a clip** — reply to a track with a time range and it cuts that section out and sends it back.
- **Channel workflow** — one-tap button to publish a track to your own channel, with duplicate detection.

### 🎬 Video
- **Social platforms** — Instagram, TikTok, Twitter/X, Facebook, Reddit, Twitch, Vimeo, Dailymotion and Aparat. Send the link, that's the whole interaction.
- **Live progress** — reports a percentage while fetching, so it's clear whether a job is moving or stuck.
- **Audio-only option** — every result offers both the video and just the sound as a music file.

### 📝 Text & AI
- **Full transcript** — extracts YouTube captions (hand-written or auto-generated) and cleans them up.
- **Summarise** — turns a long video into a handful of paragraphs with one button.
- **Translate** — runs a long English transcript through in sections and returns readable Persian, with progress.
- **Ask about a video** — ask a question and it answers from the captions, without you reading the whole thing.
- **Read the comments** — pulls up to 300 comments and reports what was said and where opinion landed.
- **Image understanding** — send a photo and it describes what's there and transcribes any text in it.
- **Voice notes** — transcribes any voice message and converts it into a proper audio file.
- **Chat mode** — a conversational assistant that can fetch a song mid-conversation without switching modes.

### ⏰ Automations
- **Nightly check-in** — at midnight it asks how you're doing, with a one-tap reply that sends comforting music.
- **Daily suggestion** — each morning it picks a track from a set of seeds describing your taste.
- **Follow a channel** — follow a YouTube channel or playlist and every new upload arrives automatically, with buttons for audio, video, transcript and comments.
- **Podcasts from RSS** — give it a podcast feed URL and it lists episodes ready to pull down.

## Architecture

Samery uses a small **provider architecture**: every source is its own module implementing a common interface, so adding a new one is a few hours of work rather than a rewrite.

```
samery/
├── bot.py              # Telegram handlers, chat brain, wiring
├── config.py           # environment-driven configuration
├── downloader.py       # resumable download, integrity checks, tagging
├── enrich.py           # cover art (iTunes / Cover Art Archive) + lyrics (LRCLIB)
├── library.py          # SQLite: downloads, sent-cache, enrich-cache, posted
└── providers/
    ├── base.py         # Track / Release data model + Provider protocol
    ├── ytdlp.py        # YouTube / SoundCloud / social platforms (+ playlists)
    ├── generic.py      # catch-all link grabber (yt-dlp → page scrape)
    ├── musicdel.py     # example direct-download site adapter
    └── ...             # additional sources
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
| AI | Any OpenAI-compatible endpoint (tool calling) |
| Runtime | Python 3.11+, runs as a background service (`launchd` / `systemd`) |

## Sources

YouTube · SoundCloud · Spotify · archive.org · Jamendo · ccMixter · musicdel.ir · Pinterest · Instagram · TikTok · X/Twitter · Aparat · Podcast RSS · + any other link `yt-dlp` supports.

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
| `LLM_PROVIDER` | The chat/AI provider |
| `*_API_KEY` | Credentials for the chosen provider |

### Run

```bash
./run.sh          # or: python bot.py
```

## Commands

| Command | What it does |
|---|---|
| `/search <query>` | Find a track by name |
| `/yt`, `/sc` | Fetch from a YouTube / SoundCloud link |
| `/text` | Get a video transcript |
| `/library`, `/stats`, `/recent` | Browse the downloaded archive |
| `/lyrics <artist - title>` | Fetch lyrics only |
| `/enrich` | Add artwork, lyrics and dates to a file |
| `/favs`, `/fav` | Favourites |
| `/cut 2:30 3:10` | Trim a section out of a track |
| `/follow`, `/follows` | Follow a channel / list follows |
| `/rss` | List episodes from a podcast feed |
| `/checkin`, `/suggest` | Nightly check-in / daily suggestion |
| `/chat` | Toggle the AI chat companion |

Plain messages, links, forwarded audio, photos and voice notes are all handled automatically.

## Roadmap

> These are directions I'd like to explore — ideas and planned work, not firm commitments. They may change as the project evolves.

- **More lossless sources** — additional providers, made easy by the provider-based architecture.
- **Smarter archive** — faster search across the personal library and better duplicate handling.
- **In-chat settings** — change bot settings (like switching the AI model) with a command, without editing files.
- **Automated tests** — tests for the providers, so adding a new source doesn't break existing ones.

## Notes & limitations

- Telegram bots can upload files up to **50 MB** and download up to **20 MB** via the Bot API; larger files are saved locally instead.
- Audio from lossy platforms (SoundCloud/YouTube) is only as good as the source.
- The AI features depend on the configured provider's availability and quota.

## Disclaimer

Samery is a **personal-use** project for managing a private media library. It is not intended for redistribution of copyrighted material or any commercial use. Respect the terms of service of the platforms you access and the rights of content owners.

## License

Released under the MIT License. See [`LICENSE`](LICENSE) for details.

---

<p align="center"><sub>Built as a personal project · maintained by <a href="https://github.com/omidalighadr">@omidalighadr</a></sub></p>
