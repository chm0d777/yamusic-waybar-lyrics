# yamusic-waybar-lyrics

[English](README.en.md) | [Русский](README.md)

Синхронизированные тексты Яндекс Музыки для Waybar.

> Предупреждение: этот модуль был написан не человеком, а с помощью кодинг агента GPT-5.5. Оптимизация может быть не идеальная.

Модуль читает metadata/position из MPRIS, получает LRC-тексты из Яндекс Музыки, кеширует их на диске и выводит JSON для Waybar. Также можно перематывать трек к следующей или предыдущей строке текста колёсиком мыши.

## Превью

![Скриншот Waybar lyrics module](assets/screenshot.png)

## Анимация перед первой строкой

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
chmod +x ~/.local/bin/yamusic-waybar-lyrics
```

Если вы клонировали или скачали репозиторий, можно установить локальный файл:

```sh
install -Dm755 yamusic-waybar-lyrics ~/.local/bin/yamusic-waybar-lyrics
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
