# yamusic-waybar-lyrics

[English](README.en.md) | [Русский](README.md)

Синхронизированные тексты Яндекс Музыки для Waybar.

> Предупреждение: этот модуль был написан не человеком, а с помощью кодинг агента GPT-5.5. Оптимизация может быть не идеальная.

Модуль читает metadata/position из MPRIS, получает LRC-тексты из Яндекс Музыки, кеширует их на диске и выводит JSON для Waybar. Также можно перематывать трек к следующей или предыдущей строке текста колёсиком мыши.

## Превью

![Скриншот Waybar lyrics module](assets/screenshot.png)

## Анимация Перед Первой Строкой

<img src="assets/demo-loading.gif" width="218" alt="Демо анимированных точек загрузки">

## Возможности

- Прямой backend для Yandex Music LRC.
- Позиция воспроизведения через любой совместимый MPRIS player/browser.
- In-memory cache во время работы процесса.
- Persistent cache в `~/.cache/yamusic-waybar-lyrics/`.
- Negative cache для треков без текста на 3 дня.
- Tooltip с текущей строкой и 5 следующими.
- Перемотка колёсиком мыши к следующей/предыдущей строке текста.
- По умолчанию используется Firefox, но player target настраивается через `MPRIS_PLAYER`.
- Опциональный fallback на `sptlrx` для случаев без direct lyrics.
- Отдельный Waybar-модуль для лайка/дизлайка текущего трека.
- Токен не сохраняется в файлы или cache.

## Требования

- Python 3.10+
- `dbus-python`
- Waybar
- Любой player/browser, который отдаёт metadata и position через MPRIS. Firefox используется по умолчанию.
- OAuth token Яндекс Музыки для direct LRC lookup. Текущий трек не обязательно должен играть именно в веб-интерфейсе Яндекс Музыки, но для direct lyrics он должен находиться через Yandex Music API.
- `playerctl` для действий по клику из примера
- Опционально: `sptlrx` для fallback

На Arch/Arch-based системах:

```sh
sudo pacman -S python-dbus playerctl waybar
```

## Установка

```sh
mkdir -p ~/.local/bin
curl -fsSL https://raw.githubusercontent.com/chm0d777/yamusic-waybar-lyrics/main/yamusic-waybar-lyrics -o ~/.local/bin/yamusic-waybar-lyrics
curl -fsSL https://raw.githubusercontent.com/chm0d777/yamusic-waybar-lyrics/main/yamusic-waybar-like -o ~/.local/bin/yamusic-waybar-like
chmod +x ~/.local/bin/yamusic-waybar-lyrics
chmod +x ~/.local/bin/yamusic-waybar-like
```

Если вы клонировали или скачали репозиторий, можно установить локальный файл:

```sh
install -Dm755 yamusic-waybar-lyrics ~/.local/bin/yamusic-waybar-lyrics
install -Dm755 yamusic-waybar-like ~/.local/bin/yamusic-waybar-like
```

`install -Dm755` копирует локальный файл `yamusic-waybar-lyrics`, создаёт родительские директории при необходимости и делает файл исполняемым.

Передайте OAuth token Яндекс Музыки через environment вашей shell/session:

Как получить токен:

- https://ym.marshal.dev/token/

```sh
export YANDEX_TOKEN='your-token-here'
```

Не коммитьте токены. Не кладите реальные токены в этот репозиторий.

## Waybar

Добавьте `custom/lyrics` в modules Waybar и используйте snippet из `examples/waybar-config.jsonc`.

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

Для MPRIS player не из Firefox добавьте `MPRIS_PLAYER` перед командой. Пример:

```jsonc
"exec": "MPRIS_PLAYER=chromium ~/.local/bin/yamusic-waybar-lyrics"
```

Добавьте CSS из `examples/waybar-style.css` или адаптируйте его под свою тему.

Для отдельной кнопки лайка добавьте `custom/like` в modules Waybar и используйте snippet из `examples/waybar-like-config.jsonc`.

```jsonc
"custom/like": {
    "return-type": "json",
    "format": "{}",
    "hide-empty-text": true,
    "exec": "~/.local/bin/yamusic-waybar-like status",
    "interval": 10,
    "signal": 12,
    "on-click": "~/.local/bin/yamusic-waybar-like toggle",
    "on-click-right": "~/.local/bin/yamusic-waybar-like dislike",
    "on-click-middle": "~/.local/bin/yamusic-waybar-like undislike"
}
```

CSS для кнопки лайка лежит в `examples/waybar-like-style.css`.

После изменений перезапустите Waybar.

## Управление

- ЛКМ на lyrics: play/pause
- ПКМ на lyrics: следующий трек
- Клик колёсиком на lyrics: предыдущий трек
- Scroll up на lyrics: перейти к следующей строке текста
- Scroll down на lyrics: перейти к предыдущей строке текста
- Like module ЛКМ: поставить/снять лайк

## Like Module

Кнопка лайка использует те же endpoints, что описаны в `MarshalX/yandex-music-api`:

- `users/{uid}/likes/tracks/add-multiple` для лайка
- `users/{uid}/likes/tracks/remove` для снятия лайка

Для треков передаётся `track_id:album_id`, поэтому трек попадает именно в библиотеку “Мне нравится”, а не лайкается как плейлист/альбом/артист.

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

## API Mapping (примеры из документации MarshalX)

Ниже показано, как наши raw-HTTP вызовы соответствуют официальной документации `MarshalX/yandex-music-api`.

### Подпись запросов

В MarshalX подпись генерируется так:

```python
from yandex_music.utils.sign_request import get_sign_request
sign = get_sign_request(track_id="12345")
print(sign.timestamp, sign.value)
```

У нас — тот же алгоритм (`yandex_music/utils/sign_request.py` line-by-line):

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

- Тот же ключ `p93jhgh689SBReK6ghtw62` из Android-приложения.
- Тот же формат сообщения: `{numeric_track_id}{timestamp}`.
- Тот же HMAC-SHA256 + Base64.

### Поиск трека

MarshalX:

```python
from yandex_music import Client
client = Client().init()
result = client.search("Deftones Change")
track = result.best.result
```

У нас:

```python
params = urllib.parse.urlencode({"text": "Deftones Change", "type": "track", "page": "0"})
result = request_json(f"https://api.music.yandex.net/search?{params}")
tracks = result["tracks"]["results"]
```

### Лайк трека

MarshalX:

```python
from yandex_music import Client
client = Client(TOKEN).init()
client.users_likes_tracks_add("12345:67890")
```

У нас:

```python
request_json(
    f"https://api.music.yandex.net/users/{uid}/likes/tracks/add-multiple",
    {"track-ids": "12345:67890"}
)
```

Endpoint и payload (`track-ids`) идентичны внутреннему вызову библиотеки.

### Account status (получение uid)

MarshalX:

```python
from yandex_music import Client
client = Client(TOKEN).init()
print(client.me.account.uid)
```

У нас:

```python
result = request_json("https://api.music.yandex.net/account/status")
uid = result["account"]["uid"]
```

### Получение токена

См. официальную документацию MarshalX:

- https://ym.marshal.dev/token/
- https://github.com/MarshalX/yandex-music-api/blob/main/docs/source/token.md

Мы читаем `YANDEX_TOKEN` только из `os.environ`, не сохраняем и не логируем.

## Cache

Тексты кешируются по Yandex `track_id:album_id` в:

```text
~/.cache/yamusic-waybar-lyrics/
```

Успешные cache entries хранятся бессрочно. Entries для отсутствующих lyrics истекают через 3 дня.

Cache-файлы содержат только metadata трека и timestamps/text строк.

## Privacy

Скрипт читает `YANDEX_TOKEN` из environment процесса во время runtime. Он не печатает токен и не пишет его на диск.

## Honorable Mentions

- [MarshalX/yandex-music-api](https://github.com/MarshalX/yandex-music-api) за документацию и реализацию поведения unofficial Yandex Music API, на которое опирается этот модуль.
- [sptlrx](https://github.com/raitonoberu/sptlrx) как inspiration/fallback path для synced lyrics в terminal/status-bar workflow.

## Лицензия

GPL-3.0-or-later.

Canonical references:

- https://www.gnu.org/licenses/gpl-3.0.txt
- https://spdx.org/licenses/GPL-3.0-or-later.html
