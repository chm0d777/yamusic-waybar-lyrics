# yamusic-waybar-lyrics

[English](README.md) | [Русский](README.ru.md)

Синхронизированные тексты Яндекс Музыки для Waybar.

> Предупреждение: этот модуль был написан не полностью вручную, а с помощью AI coding agent GPT-5.5.

Модуль читает metadata/position из Firefox MPRIS, получает LRC-тексты из Яндекс Музыки, кеширует их на диске и выводит JSON для Waybar. Также можно перематывать трек к следующей или предыдущей строке текста колёсиком мыши.

## Скриншот

![Скриншот Waybar lyrics module](assets/screenshot.png)

## Анимация Загрузки

![Демо анимированных точек загрузки](assets/demo-loading.gif)

## Возможности

- Прямой backend для Yandex Music LRC.
- Позиция воспроизведения через Firefox MPRIS.
- In-memory cache во время работы процесса.
- Persistent cache в `~/.cache/yamusic-waybar-lyrics/`.
- Negative cache для треков без текста на 3 дня.
- Tooltip с текущей строкой и 5 следующими.
- Перемотка колёсиком мыши к следующей/предыдущей строке текста.
- Опциональный fallback на `sptlrx` для не-Yandex треков или случаев без direct lyrics.
- Токен не сохраняется в файлы или cache.

## Требования

- Python 3.10+
- `dbus-python`
- Waybar
- Firefox с MPRIS для Яндекс Музыки
- `playerctl` для действий по клику из примера
- Опционально: `sptlrx` для fallback

На Arch/Arch-based системах:

```sh
sudo pacman -S python-dbus playerctl waybar
```

## Установка

```sh
install -Dm755 yamusic-waybar-lyrics ~/.local/bin/yamusic-waybar-lyrics
```

Передайте OAuth token Яндекс Музыки через environment вашей shell/session:

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

Добавьте CSS из `examples/waybar-style.css` или адаптируйте его под свою тему.

После изменений перезапустите Waybar.

## Управление

- ЛКМ: play/pause
- ПКМ: следующий трек
- Клик колёсиком: предыдущий трек
- Scroll up: перейти к следующей строке текста
- Scroll down: перейти к предыдущей строке текста

## Cache

Тексты кешируются по Yandex `track_id:album_id` в:

```text
~/.cache/yamusic-waybar-lyrics/
```

Успешные cache entries хранятся бессрочно. Entries для отсутствующих lyrics истекают через 3 дня.

Cache-файлы содержат только metadata трека и timestamps/text строк. Они не содержат `YANDEX_TOKEN`.

## Privacy

Скрипт читает `YANDEX_TOKEN` из environment процесса во время runtime. Он не печатает токен и не пишет его на диск.

Значение для подписи Yandex request в исходниках является публичной compatibility-константой, которую используют открытые клиенты Яндекс Музыки. Это не account token и не приватный API key.

## Honorable Mentions

- [MarshalX/yandex-music-api](https://github.com/MarshalX/yandex-music-api) за документацию и реализацию поведения unofficial Yandex Music API, на которое опирается этот модуль.
- [sptlrx](https://github.com/raitonoberu/sptlrx) как inspiration/fallback path для synced lyrics в terminal/status-bar workflow.

## Лицензия

GPL-3.0-or-later.

Canonical references:

- https://www.gnu.org/licenses/gpl-3.0.txt
- https://spdx.org/licenses/GPL-3.0-or-later.html
