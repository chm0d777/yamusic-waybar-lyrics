# yamusic-waybar-lyrics

Synced Yandex Music lyrics for Waybar.

> Disclosure: this module was coded with an AI coding agent, GPT-5.5, rather than written entirely by hand.

The module reads Firefox MPRIS metadata/position, fetches LRC lyrics from Yandex Music, caches them on disk, and prints Waybar JSON. It can also seek to the next/previous lyric line with mouse wheel actions.

## Features

- Direct Yandex Music LRC backend.
- MPRIS playback position from Firefox.
- In-memory cache while running.
- Persistent cache in `~/.cache/yamusic-waybar-lyrics/`.
- Negative cache for missing lyrics for 3 days.
- Tooltip with current line plus next 5 lines.
- Mouse wheel seek to next/previous lyric line.
- Optional `sptlrx` fallback for non-Yandex/no-direct-lyrics cases.
- No token is stored in files or cache.

## Requirements

- Python 3.10+
- `dbus-python`
- Waybar
- Firefox exposing MPRIS for Yandex Music
- `playerctl` for the example click actions
- Optional: `sptlrx` for fallback

On Arch/Omarchy-like systems:

```sh
sudo pacman -S python-dbus playerctl waybar
```

## Install

```sh
install -Dm755 yamusic-waybar-lyrics ~/.local/bin/yamusic-waybar-lyrics
```

Set a Yandex Music OAuth token in your shell/session environment:

```sh
export YANDEX_TOKEN='your-token-here'
```

Do not commit tokens. Do not put real tokens into this repository.

## Waybar

Add `custom/lyrics` to your Waybar modules and use the snippet from `examples/waybar-config.jsonc`.

Add the CSS from `examples/waybar-style.css` or adapt it to your theme.

Restart Waybar after changes.

## Controls

- Left click: play/pause
- Right click: next track
- Middle click: previous track
- Scroll up: seek to next lyric line
- Scroll down: seek to previous lyric line

## Cache

Lyrics are cached by Yandex `track_id:album_id` in:

```text
~/.cache/yamusic-waybar-lyrics/
```

Positive cache entries are kept indefinitely. Missing-lyrics entries expire after 3 days.

Cache files contain track metadata and lyric timestamps/text only. They do not contain `YANDEX_TOKEN`.

## Privacy

The script reads `YANDEX_TOKEN` from the process environment at runtime. It never prints it and never writes it to disk.

The Yandex request signing value in the source is a public compatibility constant used by existing open Yandex Music clients. It is not an account token or private API key.

## License

GPL-3.0-or-later.

Canonical references:

- https://www.gnu.org/licenses/gpl-3.0.txt
- https://spdx.org/licenses/GPL-3.0-or-later.html
