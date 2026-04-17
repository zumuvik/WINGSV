<h1 align="center">WINGS V</h1>
<p align="center">
  <a href="https://t.me/+KrgCVOtwL980ZDky">
    <img src="https://img.shields.io/badge/Telegram-Чат-26A5E4?style=for-the-badge&logo=telegram&logoColor=white" alt="Telegram Чат">
  </a>
</p>

Клиент Xray, vk-turn-proxy, WireGuard, AmneziaWG в интерфейсе Samsung One UI

## Скриншоты
|                                                                                           |                                                                                           |
| ----------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------- |
![Главная](readme-res/screenshots/main.jpg) | ![Роутинг приложений](readme-res/screenshots/appsrouting.jpg)
![VPN Hotspot](readme-res/screenshots/vpnhotspot.jpg) | ![Настройки](readme-res/screenshots/settings.jpg)

## Что умеет

- работать через `Xray (VLESS)`, `VK TURN + WireGuard или AmneziaWG`, обычные `WireGuard/AmneziaWG`
- показывать статус подключения, IP, страну, провайдера и сетевую статистику
- работать в обычном VPN режиме через `VpnService`
- работать в root режиме для `VK TURN + WireGuard`
- настраивать маршрутизацию по приложениям
- раздавать VPN через Wi‑Fi, USB, Bluetooth и Ethernet
- показывать отдельные логи `vk-turn-proxy`, `Xray` и runtime приложения
- импортировать и экспортировать конфигурации через `wingsv://`
- импортировать `vless://` и raw `awg-quick` конфиги
- работать с `Xray` профилями и подписками
- переключаться через launcher actions, внешние intents и Quick Settings tiles

## Архитектура

```
WINGS V (Android VPN клиент)
├── app/                     # Main Android приложение
│   └── src/main/java/wings/v/
│       ├── core/            # Бизнес-логика (50 файлов)
│       │   ├── AppPrefs.java           # Preferences хранилище
│       │   ├── WingsImportParser.java  # Импорт конфигов
│       │   ├── AutoSearchManager.java  # Автопоиск серверов
│       │   ├── XrayStore.java           # Xray профили и подписки
│       │   └── ...
│       ├── service/        # VPN сервисы (12 файлов)
│       │   ├── ProxyTunnelService.java # Главный VPN сервис (278KB)
│       │   ├── XrayVpnService.java      # Xray VPN интеграция
│       │   └── ...
│       ├── ui/             # UI фрагменты (18 файлов)
│       │   ├── MainActivity.java       # Точка входа
│       │   ├── HomeFragment.java       # Главный экран
│       │   ├── ProfilesFragment.java    # Управление профилями
│       │   └── ...
│       ├── activity/       # Activities
│       ├── xray/           # Xray интеграция
│       ├── xposed/         # Xposed модуль
│       ├── qs/             # Quick Settings tiles
│       └── receiver/       # Broadcast receivers
│
└── external/               # Submodules (8 штук)
    ├── vk-turn-proxy       # VK TURN клиент (Go, нативный)
    ├── libXray             # Xray Go bindings (gomobile)
    ├── Xray-core           # Xray core
    ├── amneziawg-android   # AmneziaWG tunnel
    ├── VPNHotspot          # Root tethering
    ├── byedpi              # DPI обход
    └── librustoreparser    # RuStore парсер
```

### Ключевые модули

| Модуль | Размер | Описание |
|--------|--------|----------|
| **ProxyTunnelService** | 278KB | Главный VPN сервис |
| **WingsImportParser** | 68KB | Импорт конфигов (wingsv://, vless://) |
| **AutoSearchManager** | 80KB | Автопоиск серверов |
| **AppPrefs** | 56KB | Preferences хранилище |
| **XrayStore** | 29KB | Xray профили и подписки |

### Backend типы

| Тип | Описание |
|-----|----------|
| **VK TURN + WireGuard** | Обычный VPN режим через VpnService |
| **Xray/VLESS** | Прокси протокол (VLESS, VMESS, Trojan) |
| **VK TURN + AmneziaWG** | WireGuard-совместимый протокол |
| **Root mode** | Раздача через Wi-Fi/USB/BT/Ethernet |

### Зависимости

- **Java 17**, Android SDK 36, NDK
- **OneUI/SESL 8** — Samsung интерфейс
- **WireGuard Android** tunnel library
- **Go 1.25-1.26** — сборка нативных библиотек
- **Protobuf** — сериализация конфигов

## Панель 3x-ui с встроенным vk-turn-proxy
В качестве сервера, вы можете использовать эту [панель 3x-ui](https://github.com/WINGS-N/3x-ui), в которой уже вшит vk-turn-proxy как inbound

## `wingsv://` ссылки
- Основной формат: `wingsv://{base64(deflate(protobuf_data))}`
- Основной импорт работает через protobuf+deflate
- Внутри могут храниться:
  - `VK TURN + WireGuard` настройки
  - `Xray` профили и подписки
  - `VK TURN + AmneziaWG` конфиг
- Старый JSON вариант - legacy

## Что используется

- Java для основного приложения
- OneUI / SESL 8 для интерфейса
- `com.wireguard.android:tunnel` для WireGuard
- `external/vk-turn-proxy` для native `VK TURN` клиента
- `external/libXray` + `external/Xray-core` для `Xray`
- `external/amneziawg-android` для `VK TURN + AmneziaWG`
- `external/VPNHotspot` для root раздачи

## Сборка

Нужно задать credentials для SESL GitHub Packages вне репозитория:

- `seslUser`
- `seslToken`

```bash
# Сразу склонить с submodules
git clone --recurse-submodules https://github.com/WINGS-N/WINGSV.git

# Или после простого clone
# Инициализация submodules
git submodule update --init --recursive
```

Локальная сборка:

```bash
# debug сборка
./gradlew :app:assembleDebug

# релизная сборка
./gradlew :app:assembleRelease
```

Для локальной сборки также нужны:

- Android SDK
- Android NDK
- `protoc`
- `go`
- `gomobile`

## Release

GitHub Actions собирают:

- CI debug build на `main`
- release APK по тегам `v*`

## Special thanks to

- [XTLS](https://github.com/XTLS)
- [cacggghp](https://github.com/cacggghp)
- [Samsung](https://samsung.com)
- [tribalfs](https://github.com/tribalfs)
- [Mygod](https://github.com/Mygod)
- [zx2c4](https://github.com/zx2c4)
- [Amnezia VPN](https://github.com/amnezia-vpn)
