# vdget

A command-line video downloader built for the modern web. Most video downloaders break on JavaScript-heavy sites where the stream URL is generated at runtime. vdget handles this by combining yt-dlp with a headless Chromium engine that renders the page like a real browser, intercepts the stream, and downloads it.

> **Legal notice:** This tool is intended for downloading content you have the right to access — content you own, content under a Creative Commons or open license, publicly available broadcasts, or content from platforms that permit downloading. Downloading copyrighted content without authorization may violate copyright law and the terms of service of the platform. You are solely responsible for how you use this tool.

---

## How it works

```
URL ──► yt-dlp probe ──► supported? ──► download ✓
                     └── not supported? ──► headless browser ──► intercept stream ──► download ✓
```

**Strategy 1 — yt-dlp direct**
For any site yt-dlp already supports (YouTube, Twitch, Vimeo, Twitter/X, TikTok, Reddit, Dailymotion, and 1000+ others), vdget runs a fast probe and downloads directly. HLS and DASH fragmented streams are handled natively and muxed by ffmpeg.

**Strategy 2 — Headless browser sniper**
For sites that generate stream URLs in JavaScript at runtime, vdget launches a real Chromium instance, loads the page, follows embed player redirects, intercepts all network traffic, finds the `.m3u8` / `.mpd` / `.mp4` stream URL, and feeds it back to yt-dlp for download. This covers multi-hop embed chains where the actual player is loaded from a third-party host.

---

## Requirements

- Python 3.10+
- ffmpeg

Install ffmpeg:
```bash
# Ubuntu / Debian
sudo apt install ffmpeg

# Arch
sudo pacman -S ffmpeg

# macOS
brew install ffmpeg

# Windows
winget install ffmpeg
```

---

## Installation

```bash
git clone https://github.com/yourname/vdget
cd vdget
pip install -r requirements.txt
python -m playwright install chromium
```

---

## Usage

```bash
python vdget.py <URL> [options]
```

### Basic examples

```bash
# Standard download — best available quality
python vdget.py https://example.com/video

# Specific quality
python vdget.py https://example.com/video -q 1080
python vdget.py https://example.com/video -q 720
python vdget.py https://example.com/video -q 480

# Audio only
python vdget.py https://example.com/video -q audio -f mp3

# Different container
python vdget.py https://example.com/video -f mkv

# Custom output directory
python vdget.py https://example.com/video -o ~/Videos
```

### Subtitles

```bash
# Download English subtitles as a separate .srt file
python vdget.py https://example.com/video --subs

# Multiple languages
python vdget.py https://example.com/video --subs en,fr,ar

# All available languages
python vdget.py https://example.com/video --subs all

# Burn subtitles into the video
python vdget.py https://example.com/video --subs embed
```

### Advanced

```bash
# Force headless browser mode, skip yt-dlp probe
python vdget.py https://example.com/video --snipe-only

# Show the browser window — useful when the site has a captcha
python vdget.py https://example.com/video --no-headless

# Pass browser cookies for authenticated content
python vdget.py https://example.com/video --cookies cookies.txt

# Find stream URL without downloading
python vdget.py https://example.com/video --dry-run

# Give the browser more time on slow-loading players
python vdget.py https://example.com/video -w 30
```

---

## Options

| Option | Default | Description |
|--------|---------|-------------|
| `-q`, `--quality` | `best` | `best` / `1080` / `720` / `480` / `audio` |
| `-f`, `--format` | `mp4` | `mp4` / `mkv` / `webm` / `mp3` |
| `-o`, `--output` | `./downloads` | Output directory |
| `-w`, `--wait` | `15` | Seconds to wait for stream in browser mode |
| `--subs` | off | Subtitle language(s) or `embed` to burn in |
| `--no-headless` | off | Show browser window |
| `--cookies` | — | Path to Netscape cookies.txt |
| `--snipe-only` | off | Skip yt-dlp, use browser mode only |
| `--dry-run` | off | Find stream URL without downloading |

---

## Cookies

Some platforms require authentication to access content. Export your browser session cookies using the **"Get cookies.txt LOCALLY"** extension (available for Chrome and Firefox). It exports in Netscape format which vdget reads directly.

```bash
python vdget.py https://example.com/members-only-video --cookies cookies.txt
```

---

## Supported stream types

| Type | Description |
|------|-------------|
| HLS `.m3u8` | Fragmented, segments downloaded sequentially to avoid rate limiting |
| DASH `.mpd` | Fragmented, audio and video muxed by ffmpeg |
| MP4 / MKV / WebM | Direct file download |
| Any URL | Anything yt-dlp can handle once the URL is extracted |

---

## Troubleshooting

**`yt-dlp not found`**
```bash
pip install yt-dlp
```

**`playwright not installed`**
```bash
pip install playwright
python -m playwright install chromium
```

**No streams found on a JS-heavy site**
```bash
# Give the browser more time
python vdget.py <url> --snipe-only -w 30

# Watch what the browser actually loads
python vdget.py <url> --snipe-only --no-headless

# Try with session cookies
python vdget.py <url> --snipe-only --cookies cookies.txt
```

**No audio in downloaded video**
Make sure ffmpeg is installed. vdget forces AAC audio re-encoding on merge to handle codec/container mismatches that would otherwise produce a silent video.

**Download failing with 429 Too Many Requests**
Already handled — vdget uses sequential fragment downloads with randomised sleep intervals to stay within CDN rate limits. If you still hit 429s, increase the sleep manually by editing `--sleep-interval` and `--max-sleep-interval` in `build_ytdlp_cmd`.

**Unicode filename errors on Windows**
Already handled via `--windows-filenames` and RFC 5987 Content-Disposition encoding.

---

## Tech stack

- [yt-dlp](https://github.com/yt-dlp/yt-dlp) — video extraction and download engine
- [Playwright](https://playwright.dev/python/) — headless Chromium automation
- [ffmpeg](https://ffmpeg.org/) — audio/video muxing and re-encoding

---

## Disclaimer

This tool is provided for educational and personal use. The authors are not responsible for any misuse. Always respect the terms of service of the platforms you interact with and the copyright of content owners.
