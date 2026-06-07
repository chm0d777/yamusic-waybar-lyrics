# yamusic-waybar-lyrics

[English](README.en.md) | [Русский](README.md)

Synced Yandex Music lyrics for Waybar.

> Disclosure: this module was coded with an AI coding agent, GPT-5.5, rather than written entirely by hand. Optimization may be imperfect.

The module reads MPRIS metadata/position, fetches LRC lyrics from Yandex Music, caches them on disk, and prints Waybar JSON. It can also seek to the next/previous lyric line with mouse wheel actions.

## Preview

![Waybar lyrics module screenshot](assets/screenshot.png)

## Animation Before First Line

<img src="assets/demo-loading.gif" width="218" alt="Animated loading dots demo">

## Features

- Direct Yandex Music LRC backend.
- MPRIS playback position from any compatible player/browser.
- In-memory cache while running.
- Persistent cache in `~/.cache/yamusic-waybar-lyrics/`.
- Negative cache for missing lyrics for 3 days.
- Tooltip with current line plus next 5 lines.
- Mouse wheel seek to next/previous lyric line.
- Default player target is Firefox, configurable with `MPRIS_PLAYER`.
- Optional `sptlrx` fallback for no-direct-lyrics cases.
- Separate Waybar module for liking/disliking the current track.
- No token is stored in files or cache.

## Requirements

- Python 3.10+
- `dbus-python`
- Waybar
- Any player/browser exposing MPRIS metadata and position. Firefox is the default target.
- Yandex Music OAuth token for direct LRC lookup. The current track does not have to come from the Yandex Music web UI, but it must be matchable in Yandex Music for direct lyrics.
- `playerctl` for the example click actions
- Optional: `sptlrx` for fallback

On Arch/Arch-based systems:

```sh
sudo pacman -S python-dbus playerctl waybar
```

## Install

```sh
mkdir -p ~/.local/bin
curl -fsSL https://raw.githubusercontent.com/chm0d777/yamusic-waybar-lyrics/main/yamusic-waybar-lyrics -o ~/.local/bin/yamusic-waybar-lyrics
curl -fsSL https://raw.githubusercontent.com/chm0d777/yamusic-waybar-lyrics/main/yamusic-waybar-like -o ~/.local/bin/yamusic-waybar-like
chmod +x ~/.local/bin/yamusic-waybar-lyrics
chmod +x ~/.local/bin/yamusic-waybar-like
```

If you cloned or downloaded this repository, install the local file instead:

```sh
install -Dm755 yamusic-waybar-lyrics ~/.local/bin/yamusic-waybar-lyrics
install -Dm755 yamusic-waybar-like ~/.local/bin/yamusic-waybar-like
```

`install -Dm755` copies the local `yamusic-waybar-lyrics` file, creates parent directories if needed, and marks the destination executable.

Set a Yandex Music OAuth token in your shell/session environment:

How to get a token:

- https://ym.marshal.dev/token/

```sh
export YANDEX_TOKEN='your-token-here'
```

Do not commit tokens. Do not put real tokens into this repository.

## Waybar

Add `custom/lyrics` to your Waybar modules and use the snippet from `examples/waybar-config.jsonc`.

```jsonc
"custom/lyrics": {
    "return-type": "json",
    "format": "{}",
    "hide-empty-text": true,
    "exec": "~/.local/bin/yamusic-waybar-lyrics",
    "on-click": "playerctl play-pause",
    "on-click-right": "playerctl next",
    "on-click-middle": "playerctl previous",
    "on-scroll-up": "~/.local/bin/yamusic-waybar-lyrics seek next",
    "on-scroll-down": "~/.local/bin/yamusic-waybar-lyrics seek prev"
}
```

For a non-Firefox MPRIS player, prefix the command with `MPRIS_PLAYER`. Example:

```jsonc
"exec": "MPRIS_PLAYER=chromium ~/.local/bin/yamusic-waybar-lyrics"
```

Add the CSS from `examples/waybar-style.css` or adapt it to your theme.

Example CSS:

```css
#custom-lyrics {
  color: #dcdcdc;
  background-color: alpha(@foreground, 0.035);
  border: 1px solid alpha(@foreground, 0.08);
  border-radius: 0;
  padding: 0 10px;
  min-width: 140px;
  font-weight: 600;
}

#custom-lyrics.active {
  color: #eeeeee;
  background-color: alpha(@foreground, 0.045);
  border-color: alpha(@foreground, 0.10);
}

#custom-lyrics.paused {
  color: #bdbdbd;
  background-color: alpha(@foreground, 0.025);
  border-color: alpha(@foreground, 0.06);
  font-style: italic;
}

#custom-like {
  font-weight: 900;
  padding: 0 10px;
  min-width: 140px;
  min-width: 24px;
}

#custom-like.liked {
  color: #e06c75;
}
```

For a separate like button, add `custom/like` to your Waybar modules and use the snippet from `examples/waybar-like-config.jsonc`.

```jsonc
"custom/like": {
    "return-type": "json",
    "format": "{}",
    "hide-empty-text": true,
    "exec": "~/.local/bin/yamusic-waybar-like status",
    "interval": 10,
    "signal": 12,
    "on-click": "~/.local/bin/yamusic-waybar-like toggle"
}
```

Like button CSS is in `examples/waybar-like-style.css`.

Restart Waybar after changes.

## Controls

- Left click on lyrics: play/pause
- Right click on lyrics: next track
- Middle click on lyrics: previous track
- Scroll up on lyrics: seek to next lyric line
- Scroll down on lyrics: seek to previous lyric line
- Like module left click: toggle like

## Like Module

The like button uses the same endpoints documented by `MarshalX/yandex-music-api`:

- `users/{uid}/likes/tracks/add-multiple` to like
- `users/{uid}/likes/tracks/remove` to unlike

Tracks are sent as `track_id:album_id`, so the track is added to the actual “Liked” library, not accidentally liked as a playlist/album/artist.

```jsonc
"custom/like": {
    "return-type": "json",
    "format": "{}",
    "hide-empty-text": true,
    "exec": "~/.local/bin/yamusic-waybar-like status",
    "interval": 10,
    "signal": 12,
    "on-click": "~/.local/bin/yamusic-waybar-like toggle"
}
```

## API Mapping (examples from MarshalX docs)

Below is how our raw HTTP calls map to the official `MarshalX/yandex-music-api` documentation.

### Request signing

In MarshalX the sign is generated like this:

```python
from yandex_music.utils.sign_request import get_sign_request
sign = get_sign_request(track_id="12345")
print(sign.timestamp, sign.value)
```

Our code uses the exact same algorithm (`yandex_music/utils/sign_request.py` line-by-line):

```python
import base64, datetime, hashlib, hmac
KEY = "p93jhgh689SBReK6ghtw62"
track_id = "12345"
timestamp = int(datetime.datetime.now().timestamp())
message = f"{track_id}{timestamp}".encode("utf-8")
sign = base64.b64encode(
    hmac.new(KEY.encode("utf-8"), message, hashlib.sha256).digest()
).decode("utf-8")
```

- Same key `p93jhgh689SBReK6ghtw62` from the Android app.
- Same message format: `{numeric_track_id}{timestamp}`.
- Same HMAC-SHA256 + Base64.

### Track search

MarshalX:

```python
from yandex_music import Client
client = Client().init()
result = client.search("Deftones Change")
track = result.best.result
```

Ours:

```python
params = urllib.parse.urlencode({"text": "Deftones Change", "type": "track", "page": "0"})
result = request_json(f"https://api.music.yandex.net/search?{params}")
tracks = result["tracks"]["results"]
```

### Like a track

MarshalX:

```python
from yandex_music import Client
client = Client(TOKEN).init()
client.users_likes_tracks_add("12345:67890")
```

Ours:

```python
request_json(
    f"https://api.music.yandex.net/users/{uid}/likes/tracks/add-multiple",
    {"track-ids": "12345:67890"}
)
```

The endpoint and payload (`track-ids`) are identical to the library’s internal call.

### Account status (getting uid)

MarshalX:

```python
from yandex_music import Client
client = Client(TOKEN).init()
print(client.me.account.uid)
```

Ours:

```python
result = request_json("https://api.music.yandex.net/account/status")
uid = result["account"]["uid"]
```

### Getting a token

See the official MarshalX documentation:

- https://ym.marshal.dev/token/
- https://github.com/MarshalX/yandex-music-api/blob/main/docs/source/token.md

We only read `YANDEX_TOKEN` from `os.environ` and never store or log it.

## Cache

Lyrics are cached by Yandex `track_id:album_id` in:

```text
~/.cache/yamusic-waybar-lyrics/
```

Positive cache entries are kept indefinitely. Missing-lyrics entries expire after 3 days.

Cache files contain track metadata and lyric timestamps/text only.

## Privacy

The script reads `YANDEX_TOKEN` from the process environment at runtime. It never prints it and never writes it to disk.

## Honorable Mentions

- [MarshalX/yandex-music-api](https://github.com/MarshalX/yandex-music-api) for documenting and implementing the unofficial Yandex Music API behavior this module relies on.
- [sptlrx](https://github.com/raitonoberu/sptlrx) as the inspiration/fallback path for synced lyrics in terminal/status-bar workflows.

## License

GPL-3.0-or-later.

Canonical references:

- https://www.gnu.org/licenses/gpl-3.0.txt
- https://spdx.org/licenses/GPL-3.0-or-later.html
