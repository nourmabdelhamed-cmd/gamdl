# Gamdl (Glomatico's Apple Music Downloader)

[![PyPI version](https://img.shields.io/pypi/v/gamdl?color=blue)](https://pypi.org/project/gamdl/)
[![Python versions](https://img.shields.io/pypi/pyversions/gamdl)](https://pypi.org/project/gamdl/)
[![License](https://img.shields.io/github/license/glomatico/gamdl)](https://github.com/glomatico/gamdl/blob/main/LICENSE)
[![Downloads](https://img.shields.io/pypi/dm/gamdl)](https://pypi.org/project/gamdl/)

A command-line app for downloading Apple Music songs, music videos and post videos.

**Join our Discord Server:** <https://discord.gg/aBjMEZ9tnq>

## ✨ Features

- 🎵 **High-Quality Songs** - Download songs in AAC 256kbps and other codecs
- 🎬 **High-Quality Music Videos** - Download music videos in resolutions up to 4K
- 📝 **Synced Lyrics** - Download synced lyrics in LRC, SRT, or TTML formats
- 🏷️ **Rich Metadata** - Automatic tagging with comprehensive metadata
- 🎤 **Artist Support** - Download all albums or music videos from an artist
- ⚙️ **Highly Customizable** - Extensive configuration options for advanced users

## 📋 Prerequisites

### Required

- **Python 3.10 or higher**
- **Active Apple Music subscription**
- **Apple Music cookies** (unless using Wrapper) - export your browser cookies in Netscape format while logged in at [Apple Music](https://music.apple.com):
  - **Firefox**: [Export Cookies](https://addons.mozilla.org/addon/export-cookies-txt)
  - **Chromium**: [Get cookies.txt LOCALLY](https://chromewebstore.google.com/detail/get-cookiestxt-locally/cclelndahbckbenkjhflpdbgdldlbecc)

The exported file must contain the `media-user-token` cookie for
`.music.apple.com`. Treat it like a password: do not commit or share it, and
export a new copy if the session expires.

### Optional Dependencies

#### Wrapper

Run the [Wrapper v2](https://github.com/glomatico/wrapper-v2) server for wrapper-backed account, playback, and decryption requests. Enable it with `--use-wrapper` or `use_wrapper = true`. Configure wrapper HTTP account/playback calls with `--wrapper-url` or `wrapper_url`, and configure WV2D batch TCP decrypt with `--wrapper-decrypt-host` / `--wrapper-decrypt-port`.

This version of Gamdl requires Wrapper v2; the original Wrapper service is not
compatible. Follow the Wrapper v2 README for its Docker, Apple native-library,
and login setup.

gamdl builds a private Rust extension from `gamdl/downloader/ammuxer` as `gamdl._ammuxer`. That native media engine handles wrapper TCP decrypt/reassembly plus MP4/M4A writing and muxing; Python remains responsible for the CLI, downloads, metadata tagging, and high-level orchestration.

The wrapper is recommended when using the `alac` song codec. ALAC can be attempted without wrapper, but it probably won't work due to API limitations.

**Note:**

- When using the Wrapper, you'll be asked to insert your credentials to login if you haven't already.
- Gamdl 3.8.4 requires Wrapper v2 HTTP API version `0.0.2` and uses the WV2D batch TCP protocol on port `10020` for decryption by default.
- Song codecs other than `alac` do not require the wrapper.
- Cookies can be skipped when using the wrapper.

The default endpoints work when both programs run on the same host. If Wrapper
v2 runs in another container or on another machine, use reachable HTTP and TCP
addresses instead of `127.0.0.1`, for example:

```bash
gamdl --use-wrapper \
  --wrapper-url http://wrapper-host \
  --wrapper-decrypt-host wrapper-host \
  --wrapper-decrypt-port 10020 \
  URL
```

#### N_m3u8DL-RE

Use [N_m3u8DL-RE](https://github.com/nilaoda/N_m3u8DL-RE/releases/latest) as a faster download alternative to the default yt-dlp download mode. Enable it with `--download-mode nm3u8dlre` or `download_mode = nm3u8dlre`.

If the executable is not available in your system PATH, set its location with `--nm3u8dlre-path` or `nm3u8dlre_path`.

N_m3u8DL-RE also needs FFmpeg. If the FFmpeg executable is not available in your system PATH, set its location with `--ffmpeg-path` or `ffmpeg_path`.

## 📦 Setup

Gamdl 3.8.4 publishes `cp310-abi3` wheels for CPython 3.10+ on Linux x86-64
and ARM64 systems with glibc 2.34 or newer. On older glibc systems,
Alpine/musl, other architectures, or an interpreter without a matching wheel,
the installer attempts a source build; that path may not be supported by every
interpreter. Check a glibc-based system with:

```bash
uname -m
getconf GNU_LIBC_VERSION
```

### 1. Install Gamdl

Confirm that Python 3.10 or newer is available:

```bash
python3 --version
```

#### pipx (recommended)

Gamdl is a command-line application, so [`pipx`](https://pipx.pypa.io/) is the
recommended installation method. It keeps Gamdl and its dependencies isolated
from the operating system's Python installation.

On current Debian and Ubuntu releases:

```bash
sudo apt update
sudo apt install pipx
pipx ensurepath
pipx install gamdl
```

Open a new shell after `pipx ensurepath` if the `gamdl` command is not found.

#### Virtual environment

If `pipx` is unavailable, use a virtual environment:

```bash
sudo apt update
sudo apt install python3-venv

python3 -m venv "$HOME/.venvs/gamdl"
. "$HOME/.venvs/gamdl/bin/activate"
python -m pip install --upgrade pip
python -m pip install gamdl
```

Activate the virtual environment again in each new shell before using `gamdl`.
Do not use `sudo pip` or `--break-system-packages`; many current Linux
distributions intentionally prevent `pip` from modifying the system Python.

#### Source builds (including PyPI fallback)

When no compatible wheel is available, `pip` and `pipx` automatically attempt
to build the source distribution. Source builds compile the private
`gamdl._ammuxer` Rust extension.
Install your distribution's native build tools first; on Debian and Ubuntu:

```bash
sudo apt update
sudo apt install build-essential git python3-dev python3-venv
```

Install a current stable Rust toolchain using the
[official Rust installation instructions](https://www.rust-lang.org/tools/install),
then verify that `rustc` and `cargo` are available:

```bash
rustc --version
cargo --version

git clone https://github.com/glomatico/gamdl.git
cd gamdl
python3 -m venv .venv
. .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install .
python -c 'import gamdl._ammuxer; print("native extension OK")'
```

Use `python -m pip install -e .` instead for an editable development install.
After installing the build prerequisites, users installing from PyPI can simply
rerun `pipx install gamdl` or `python -m pip install gamdl` instead of cloning
the repository.

### 2. Store the cookies file

Keep the cookie file in a stable, private location:

```bash
install -d -m 700 "$HOME/.gamdl"
install -m 600 /path/to/exported-cookies.txt "$HOME/.gamdl/cookies.txt"
```

Pass it explicitly when running Gamdl:

```bash
gamdl --cookies-path "$HOME/.gamdl/cookies.txt" URL
```

Alternatively, set `cookies_path` in the configuration file described below.
Use an absolute path in the configuration file; shell shortcuts such as `~` and
`$HOME` are not expanded there. The default `./cookies.txt` path is relative to
the directory from which Gamdl is run.

### 3. Install optional dependencies

The default `ytdlp` download mode does not need an external executable. See
[Optional Dependencies](#optional-dependencies) for Wrapper and N_m3u8DL-RE
setup.

For N_m3u8DL-RE on Linux, use `uname -m` to select the matching `linux-x64` or
`linux-arm64` release. Make the downloaded binary executable and place it on
`PATH`, or pass its absolute path with `--nm3u8dlre-path`. Install FFmpeg from
your distribution when using this mode; on Debian and Ubuntu:

```bash
sudo apt install ffmpeg
command -v N_m3u8DL-RE
command -v ffmpeg
```

Executable names are case-sensitive on Linux. Use absolute paths rather than
literal `~` or `$HOME` values when configuring external tools in `config.ini`.

### 4. Verify the installation

```bash
command -v gamdl
gamdl --version
gamdl --help
```

These commands verify the package and command-line entry point. An authenticated
download also requires a current cookie file or Wrapper session, writable output
and temporary directories, and any dependency required by the selected mode.

## 🚀 Usage

```bash
gamdl [OPTIONS] URLS...
```

### Supported URL Types

- Songs (Catalog/Library)
- Albums (Catalog/Library)
- Playlists (Catalog/Library)
- Music Videos (Catalog/Library)
- Artists
- Post Videos
- Apple Music Classical

### Examples

**Download a song:**

```bash
gamdl "https://music.apple.com/us/album/never-gonna-give-you-up-2022-remaster/1624945511?i=1624945512"
```

**Download an album:**

```bash
gamdl "https://music.apple.com/us/album/whenever-you-need-somebody-2022-remaster/1624945511"
```

**Download from an artist:**

```bash
gamdl "https://music.apple.com/us/artist/rick-astley/669771"
```

**Interactive Prompt Controls:**

| Key            | Action            |
| -------------- | ----------------- |
| **Arrow keys** | Move selection    |
| **Space**      | Toggle selection  |
| **Ctrl + A**   | Select all        |
| **Enter**      | Confirm selection |

## ⚙️ Configuration

Configure Gamdl using command-line arguments or a config file.

**Config file location:**

- Linux: `~/.gamdl/config.ini`
- Windows: `%USERPROFILE%\.gamdl\config.ini`

The file is created automatically on the first normal run. It is an INI file and
must contain a `[gamdl]` section. Command-line arguments override config values.
Use absolute paths, especially for cookies and external tools.

Example Linux configuration:

```ini
[gamdl]
cookies_path = /home/your-user/.gamdl/cookies.txt
output_path = /home/your-user/Music/Apple Music
temp_path = /home/your-user/.cache/gamdl
```

### Configuration Options

| Option                          | Description                                                       | Default                       |
| ------------------------------- | ----------------------------------------------------------------- | ----------------------------- |
| **General Options**             |                                                                   |                               |
| `--read-urls-as-txt`, `-r`      | Read URLs from text files                                         | `false`                       |
| `--config-path`                 | Config file path                                                  | `<home>/.gamdl/config.ini`    |
| `--log-level`                   | Logging level                                                     | `INFO`                        |
| `--log-file`                    | Log file path                                                     | -                             |
| `--no-exceptions`               | Don't print exceptions                                            | `false`                       |
| `--artist-auto-select`          | Automatically select artist content to download (artist URLs)     | -                             |
| `--database-path`               | Path to the SQLite database file for registering downloaded media | -                             |
| `--no-config-file`, `-n`        | Don't use a config file                                           | `false`                       |
| **Apple Music Options**         |                                                                   |                               |
| `--cookies-path`, `-c`          | Cookies file path                                                 | `./cookies.txt`               |
| `--wrapper-url`                 | Wrapper HTTP control base URL                                     | `http://127.0.0.1`            |
| `--wrapper-decrypt-host`        | Wrapper TCP decrypt host                                          | `127.0.0.1`                   |
| `--wrapper-decrypt-port`        | Wrapper TCP decrypt port                                          | `10020`                       |
| `--language`, `-l`              | Metadata language                                                 | `en-US`                       |
| **Interface Options**           |                                                                   |                               |
| `--cover-format`                | Cover format                                                      | `jpg`                         |
| `--cover-size`                  | Cover size in pixels                                              | `1200`                        |
| `--wvd-path`                    | .wvd file path                                                    | -                             |
| `--use-wrapper`                 | Use wrapper for account, playback, and decryption requests        | `false`                       |
| **Song Options**                |                                                                   |                               |
| `--synced-lyrics-format`        | Synced lyrics format                                              | `lrc`                         |
| `--song-codec-priority`         | Comma-separated codec priority                                    | `aac-web`                     |
| `--use-album-date`              | Use album release date for songs                                  | `false`                       |
| `--no-synced-lyrics`            | Don't download synced lyrics                                      | `false`                       |
| `--synced-lyrics-only`          | Download only synced lyrics                                       | `false`                       |
| **Music Video Options**         |                                                                   |                               |
| `--music-video-resolution`      | Max music video resolution                                        | `1080p`                       |
| `--music-video-codec-priority`  | Comma-separated codec priority                                    | `h264,h265`                   |
| `--music-video-remux-format`    | Music video remux format                                          | `m4v`                         |
| **Post Video Options**          |                                                                   |                               |
| `--uploaded-video-quality`      | Post video quality                                                | `best`                        |
| **Download & Path Options**     |                                                                   |                               |
| `--output-path`, `-o`           | Output directory path                                             | `./Apple Music`               |
| `--temp-path`                   | Temporary directory path                                          | `.`                           |
| `--nm3u8dlre-path`              | N_m3u8DL-RE executable path                                       | `N_m3u8DL-RE`                 |
| `--ffmpeg-path`                 | FFmpeg executable path                                            | `ffmpeg`                      |
| `--download-mode`               | Download mode                                                     | `ytdlp`                       |
| **Template Options**            |                                                                   |                               |
| `--album-folder-template`       | Album folder template                                             | `{album_artist}/{album}`      |
| `--compilation-folder-template` | Compilation folder template                                       | `Compilations/{album}`        |
| `--no-album-folder-template`    | No album folder template                                          | `{artist}/Unknown Album`      |
| `--playlist-folder-template`    | Playlist folder template                                          | `Playlists/{playlist_artist}` |
| `--single-disc-file-template`   | Single disc file template                                         | `{track:02d} {title}`         |
| `--multi-disc-file-template`    | Multi disc file template                                          | `{disc}-{track:02d} {title}`  |
| `--no-album-file-template`      | No album file template                                            | `{title}`                     |
| `--playlist-file-template`      | Playlist file template                                            | `{playlist_title}`            |
| `--date-tag-template`           | Date tag template                                                 | `%Y-%m-%dT%H:%M:%SZ`          |
| `--exclude-tags`                | Comma-separated tags to exclude                                   | -                             |
| `--truncate`                    | Max filename length                                               | -                             |
| **File Output Options**         |                                                                   |                               |
| `--overwrite`                   | Overwrite existing files                                          | `false`                       |
| `--save-cover`, `-s`            | Save cover as separate file                                       | `false`                       |
| `--save-playlist`               | Save M3U8 playlist file                                           | `false`                       |

### Template Variables

**Tags for templates and exclude-tags:**

- `album`, `album_artist`, `album_id`
- `artist`, `artist_id`
- `composer`, `composer_id`
- `date` (supports strftime format: `{date:%Y}`)
- `disc`, `disc_total`
- `media_type`
- `playlist_artist`, `playlist_id`, `playlist_title`, `playlist_track`
- `title`, `title_id`
- `track`, `track_total`

**Tags for exclude-tags only:**

- `album_sort`, `artist_sort`, `composer_sort`, `title_sort`
- `comment`, `compilation`, `copyright`, `cover`, `gapless`, `genre`, `genre_id`, `lyrics`, `rating`, `storefront`, `xid`
- `all` (special: skip all tagging)

### Logging Level

- `DEBUG`, `INFO`, `WARNING`, `ERROR`

### Download Mode

- `ytdlp`, `nm3u8dlre`

> [!NOTE]
>
> - **yt-dlp is only used as a file download library**. Media is still fetched directly from Apple Music's servers, and yt-dlp is only responsible for handling the file download process.

### Cover Format

- `jpg`
- `png`
- `raw` - Raw format as provided by the artist (requires `save_cover` to be enabled as it doesn't embed covers into files)

### Metadata Language

Use ISO 639-1 language codes (e.g., `en-US`, `es-ES`, `ja-JP`, `pt-BR`). Don't always work for music videos.

### Song Codecs

**Web:**

- `aac-web` - AAC 256kbps 44.1kHz
- `aac-he-web` - AAC-HE 64kbps 44.1kHz

**Non-web** (`alac` can be attempted without wrapper, but it probably won't work due to API limitations):

- `aac` - AAC 256kbps up to 48kHz
- `aac-he` - AAC-HE 64kbps up to 48kHz
- `aac-binaural` - AAC 256kbps binaural
- `aac-downmix` - AAC 256kbps downmix
- `aac-he-binaural` - AAC-HE 64kbps binaural
- `aac-he-downmix` - AAC-HE 64kbps downmix
- `atmos` - Dolby Atmos 768kbps
- `ac3` - AC3 640kbps
- `alac` - ALAC up to 24-bit/192kHz
- `ask` - Interactive codec selection

### Synced Lyrics Format

- `lrc`
- `srt` - SubRip subtitle format (more accurate timing)
- `ttml` - Native Apple Music format (not compatible with most media players)

### Music Video Codecs

- `h264`
- `h265`
- `ask` - Interactive codec selection

### Music Video Resolutions

- H.264: `240p`, `360p`, `480p`, `540p`, `720p`, `1080p`
- H.265 only: `1440p`, `2160p`

### Music Video Remux Formats

- `m4v`, `mp4`

### Post Video Quality

- `best` - Up to 1080p with AAC 256kbps
- `ask` - Interactive quality selection

### Artist Auto-Select Options

- `main-albums`
- `compilation-albums`
- `live-albums`
- `singles-eps`
- `all-albums`
- `top-songs`
- `music-videos`

## 🐍 Embedding

Use Gamdl as a library in your Python projects:

```python
import asyncio

from gamdl.api import AppleMusicApi
from gamdl.downloader import (
    AppleMusicBaseDownloader,
    AppleMusicDownloader,
    AppleMusicMusicVideoDownloader,
    AppleMusicSongDownloader,
    AppleMusicUploadedVideoDownloader,
)
from gamdl.interface import (
    AppleMusicBaseInterface,
    AppleMusicInterface,
    AppleMusicMusicVideoInterface,
    AppleMusicSongInterface,
    AppleMusicUploadedVideoInterface,
)


async def main():
    # Create AppleMusicApi instance from cookies
    apple_music_api = await AppleMusicApi.create_from_netscape_cookies(
        cookies_path="cookies.txt",
    )

    # Check subscription
    if not apple_music_api.active_subscription:
        print("No active Apple Music subscription")
        return

    # Create base interface
    base_interface = await AppleMusicBaseInterface.create(
        apple_music_api=apple_music_api,
    )

    # Create specialized interfaces
    song_interface = AppleMusicSongInterface(
        base=base_interface,
    )
    music_video_interface = AppleMusicMusicVideoInterface(
        base=base_interface,
    )
    uploaded_video_interface = AppleMusicUploadedVideoInterface(
        base=base_interface,
    )

    # Create main interface
    interface = AppleMusicInterface(
        song=song_interface,
        music_video=music_video_interface,
        uploaded_video=uploaded_video_interface,
    )

    # Create base downloader
    base_downloader = AppleMusicBaseDownloader(
        interface=interface,
    )

    # Create specialized downloaders
    song_downloader = AppleMusicSongDownloader(base=base_downloader)
    music_video_downloader = AppleMusicMusicVideoDownloader(
        base=base_downloader,
    )
    uploaded_video_downloader = AppleMusicUploadedVideoDownloader(base=base_downloader)

    # Create main downloader
    downloader = AppleMusicDownloader(
        song=song_downloader,
        music_video=music_video_downloader,
        uploaded_video=uploaded_video_downloader,
    )

    # Download from URL
    url = "https://music.apple.com/us/album/never-gonna-give-you-up-2022-remaster/1624945511?i=1624945512"

    download_queue = []
    async for media in downloader.get_download_item_from_url(url):
        download_queue.append(media)

    for download_item in download_queue:
        try:
            await downloader.download(download_item)
        except Exception as e:
            print(f"Error downloading: {e}")


if __name__ == "__main__":
    asyncio.run(main())
```

## 📄 License

MIT License - see [LICENSE](LICENSE) file for details

## 🤝 Contributing

Currently, I'm not interested in reviewing pull requests that change or add features. Only critical bug fixes will be considered. However, feel free to open issues for bugs or feature requests.
