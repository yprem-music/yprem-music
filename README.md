<div align="center">

<img src="logo/YPREM.png" alt="YPREM" width="180">

# YPREM<i>.</i>

**Музыка вне ограничений.** Десктопный плеер YouTube & SoundCloud,
который работает в РФ без VPN.

**DPI-обход и AdBlock встроены в приложение** — никаких системных правок,
служб, драйверов и UAC. Закрыл приложение — система нетронута.

[![Downloads](https://img.shields.io/github/downloads/yprem-music/yprem-music/total?style=flat-square&labelColor=0d0d0d&color=c0c0c0&logo=github)](https://github.com/yprem-music/yprem-music/releases)
[![Stars](https://img.shields.io/github/stars/yprem-music/yprem-music?style=flat-square&labelColor=0d0d0d&color=c0c0c0&logo=github)](https://github.com/yprem-music/yprem-music/stargazers)
![Views](https://komarev.com/ghpvc/?username=yprem-music&color=blue&style=flat-square&label=VIEWS)
[![Release](https://img.shields.io/github/v/release/yprem-music/yprem-music?style=flat-square&labelColor=0d0d0d&color=c0c0c0)](https://github.com/yprem-music/yprem-music/releases/latest)
[![Tests](https://img.shields.io/badge/tests-34%2F34-0d0d0d?style=flat-square&labelColor=0d0d0d&color=c0c0c0)](#сборка-и-проверки)

**Языки и технологии:**
![Rust](https://img.shields.io/badge/Rust-000000?style=flat-square&logo=rust&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=flat-square&logo=javascript&logoColor=000000)
![HTML5](https://img.shields.io/badge/HTML5-E34F26?style=flat-square&logo=html5&logoColor=white)
![CSS3](https://img.shields.io/badge/CSS3-1572B6?style=flat-square&logo=css3&logoColor=white)
![NSIS](https://img.shields.io/badge/NSIS-4A77A8?style=flat-square&logo=gnu-bash&logoColor=white)
![Bash](https://img.shields.io/badge/Shell-121011?style=flat-square&logo=gnu-bash&logoColor=white)

`Rust + wry/tao + vanilla JS · yt-dlp · встроенный обход-прокси · tray-icon`

[Скачать последнюю версию](#установка) · [Возможности](#возможности) · [Как это работает](#как-это-работает) · [Сборка](#сборка)

</div>

---

## Установка

| ОС | Файл | Действие |
|---|---|---|
| **Windows 10/11** | `YPREM-1.1.0-setup.exe` | Запустить → «Подробнее → Выполнить в любом случае» (SmartScreen) → всё «из коробки», yt-dlp входит в комплект |
| **Linux (deb)** | `yprem_1.1.0_amd64.deb` | `sudo apt install ./yprem_1.1.0_amd64.deb` |

> Скачивайте только из [Releases](https://github.com/yprem-music/yprem-music/releases) —
> контрольные суммы в [`SHA256SUMS.txt`](release/1.1.0/SHA256SUMS.txt).

## Возможности

- **Один звук — всегда.** Единый PlaybackController: прямой поток yt-dlp →
  SoundCloud-виджет → встроенный web-core (реальная страница YouTube/SoundCloud
  в том же окне). Никаких вторых окон и двойного звука.
- **Обход блокировок для приложения.** Локальный HTTP-прокси на случайном порту
  фрагментирует TLS ClientHello (5 стратегий + бенчмарк) — трафик только YPREM,
  система не затрагивается. DoH-фолбэк при подмене DNS.
- **AdBlock встроен.** Доменный блок-лист на прокси (EasyList + RU AdList +
  AdGuard RU, ~59 000 доменов) + MV3-расширение на Windows. Плеер YouTube —
  вне блок-листа, воспроизведение не ломается.
- **Поиск с фильтрами.** Чипы «Все/YouTube/SoundCloud» и «Все/Аудио/Видео»,
  дебаунс 350 мс, кэш выдачи, skeleton, параллельный поиск по двум площадкам.
- **Альбомы.** Запрос «мементо мори» находит альбом — карточка-плейлист над
  треками: «Слушать» (альбом по кругу) и «В мои плейлисты» (импорт с обложкой).
- **Плейлисты с обложками.** Смена обложки файлом или drag&drop (canvas 512×512),
  локальные плейлисты, импорт с YouTube.
- **Клипы.** Полноэкранный режим (F/Esc), автоскрытие панели через 3 секунды,
  чистое видео без «мебели» YouTube.
- **Мини-плеер.** Бар сворачивается в плавающую пилюлю; прогресс — во всплывающем
  окне над иконкой звука.
- **Shuffle и повтор.** Fisher–Yates без повторов до конца очереди; повтор
  очереди / трека / альбома; режимы живут между сессиями. Хоткеи **R** / **S**.
- **Метка «продолжить».** Возвращается к треку и секунде — на любом backend'е.
- **Закрытие в трей.** Воспроизведение продолжается; LKM по трею — показать окно;
  меню трея: Play/Pause, Следующий трек, Выход.
- **Telegram-сообщество** — карточка в настройках, [t.me/yprem_music](https://t.me/yprem_music).

## Как это работает

```
UI (vanilla JS) ──IPC──▶ Rust: PlaybackController ──▶ один из трёх backend'ов
                           (единая очередь/громкость)      │
                                                           ├─ прямой поток yt-dlp → <video>
                                                           ├─ SoundCloud Widget API
                                                           └─ web-core: дочерний WebView
                                                              со страницей трека
Трафик окна / yt-dlp / HTTP ──▶ встроенный прокси 127.0.0.1:<порт>
                                  ├─ TLS ClientHello фрагментация (DPI-обход)
                                  └─ доменный AdBlock-лист
```

- Один процесс = прокси-поток и webview умирают вместе с приложением: **сирот нет**.
- Single instance: повторный запуск exe будит существующее окно.
- Frameless-окно с кастомной шапкой, tray, Snap-раскладка — не требуется прав администратора.

## Известные ограничения

- yt-dlp зафиксирован на **2026.08.19** — не обновлять (новые версии без
  deno-рантайма ломают поиск).
- Web-core на Linux работает в X11-сессиях (Wayland — плашка «Открыть оригинал»).
- YouTube-миксы (RD\*) не импортируются — это ограничение платформы.
- iframe-превью загружаются с YouTube: на совсем медленном канале — появятся позже списка.

  
<div align="center">
<b>YPREM.</b> — музыка вне ограничений.<br>
<sub>Сделано на Rust, без Electron и без VPN.</sub>
</div>
