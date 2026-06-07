# Project Knowledge: yamusic-waybar-lyrics

## Что это
Модуль Waybar для синхронизированных текстов Яндекс Музыки + отдельный модуль лайка/дизлайка.

## Архитектура

### `yamusic-waybar-lyrics`
- **MPRIS**: читает `PlaybackStatus`, `Metadata`, `Position` напрямую через `dbus-python`, без `playerctl`.
- **Yandex Music API**: raw HTTP к `api.music.yandex.net`.
  - Поиск: `/search?text=...&type=track&page=0`
  - Тексты: `/tracks/{id}/lyrics?format=LRC&timeStamp={ts}&sign={sign}`
  - Подпись: HMAC-SHA256 ключом `p93jhgh689SBReK6ghtw62`, сообщение `{numeric_track_id}{timestamp}`, base64. Идентично `MarshalX/yandex-music-api/yandex_music/utils/sign_request.py`.
- **Кэш**:
  - In-memory (`yandex_cache`) на время работы процесса.
  - Persistent в `~/.cache/yamusic-waybar-lyrics/` (JSON, atomic write `.tmp` → `os.replace`).
  - Negative cache (нет текста) — 3 дня (`NEGATIVE_CACHE_TTL`).
  - Кэш не содержит `YANDEX_TOKEN`.
- **Fallback**: `sptlrx` pipe, если direct lyrics не найдены.
- **Position smoothing**: `POSITION_JITTER_MS = 900`. Между опросами D-Bus позиция дорассчитывается монотонически.
- **Seek**: Waybar scroll up/down → `SetPosition` к next/prev строке LRC через state file `/tmp/yamusic-waybar-lyrics-{uid}.json`.
- **Tooltip**: текущая строка + 5 следующих.
- **Анимация**: точки `...` перед первой строкой.

### `yamusic-waybar-like`
- One-shot JSON для Waybar (`status`) + команды (`toggle/like/unlike/dislike/undislike`).
- Те же endpoints, что в MarshalX docs:
  - `users/{uid}/likes/tracks/add-multiple`
  - `users/{uid}/likes/tracks/remove`
  - `users/{uid}/dislikes/tracks/add-multiple`
  - `users/{uid}/dislikes/tracks/remove`
- Payload: `track-ids={track_id}:{album_id}`.
- **Кэш in-memory**:
  - `uid` — 1 час.
  - `library_ids(likes/dislikes)` — 5 минут.
  - `track_search` — 1 день.
  - Инвалидация после like/unlike/dislike.

## Env
- `YANDEX_TOKEN` — только из `os.environ`, никаких `.env`-файлов, не пишется в кэш, не логируется.
- `MPRIS_PLAYER` — default `firefox`.

## Зависимости
- Python 3.10+
- `dbus-python`
- Waybar
- `playerctl` (для кликов в примере)
- Опционально: `sptlrx`

## Лицензия
GPL-3.0-or-later

## Статус
Основной функционал lyrics + like-module готов. Оптимизации (кэш, jitter, D-Bus direct) применены.
