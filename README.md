# Honyaku Konnyaku 🍃

A personal pipeline for watching online videos with localized subtitles. Downloads video, transcribes audio, translates subtitles, and plays locally — all in one command.

Named after the Doraemon gadget that lets you understand any language.

## Quick Start

First-time setup (installs yt-dlp, ffmpeg, whisper, mpv/IINA):

```
/setup
```

Then, process any video:

```
/process https://www.youtube.com/watch?v=...
```

## Pipeline

```
URL
 │
 ├─ /download    yt-dlp → videos/<title>.mp4 + .en.srt (if available)
 ├─ /transcribe  Whisper → .en.srt (only if no YouTube subtitles)
 ├─ /translate   Claude Code sub-agent → .zh-tw.srt
 ├─ /proofread   Claude Code sub-agent → terminology + fluency pass
 └─ /watch       mpv/IINA with subtitle auto-load
```

If the source video already has subtitles on the platform, transcription is skipped automatically.

## File Layout

```
honyaku-konnyaku/
├── videos/      # .mp4 + .en.srt + .zh-tw.srt (same directory for auto-load)
├── audio/       # .wav extracted for Whisper (not committed)
└── .claude/
    └── skills/  # Claude Code slash commands
```

## Dependencies

| Tool | Purpose | Install |
|------|---------|---------|
| yt-dlp | Download video | `brew install yt-dlp` |
| ffmpeg | Extract audio | `brew install ffmpeg` |
| whisper | Transcribe audio | `pip install openai-whisper` |
| mpv | Playback (CLI) | `brew install mpv` |
| IINA | Playback (GUI) | `brew install --cask iina` |

Translation uses the active Claude Code session — no extra API key needed.

## Supported Sources

Any site supported by yt-dlp (1800+ extractors): YouTube, Vimeo, Bilibili, Twitch, Twitter/X, and more.
