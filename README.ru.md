![Pi5-AoIP 2.5.0](docs/assets/header.svg)

# Pi5-AoIP

[English](README.md) | **Русский**

Платформенная библиотека и сервис синтетического PCM для Raspberry Pi 5 Model B с PREEMPT_RT. Все потоки AoIP используют CPU0.

**[Скачать 2.5.0](https://github.com/danrey-bilo/Pi5-AoIP/releases/tag/v2.5.0)** · **[Описание релиза](docs/RELEASE-2.5.0.ru.md)** · **[Проверки](docs/VALIDATION.ru.md)**

[Транспорт и статус endpoints](docs/TRANSPORT-2.5.ru.md)

## Начало работы

Нужны **Raspberry Pi 5 Model B, Debian 13 ARM64, PREEMPT_RT и проводной Gigabit Ethernet**.

Скачайте [piaoip-rpi5_2.5.0-1_arm64.deb](https://github.com/danrey-bilo/Pi5-AoIP/releases/download/v2.5.0/piaoip-rpi5_2.5.0-1_arm64.deb) и выполните на Pi:

```sh
sudo apt install ./piaoip-rpi5_2.5.0-1_arm64.deb
sudo piaoip-configure --interface eth0 --peer 192.168.50.1 --restart
/usr/lib/piaoip/aoip_peer_rpi5 --check
systemctl status pi-aoip --no-pager
```

`192.168.50.1` — пример адреса **ПК**. Сначала настройте доступные друг другу IPv4-адреса ПК и Pi. Пакет не назначает IP и не устанавливает RT-ядро. Для разработки приложений доступен дополнительный `piaoip-rpi5-sdk_2.5.0-1_arm64.deb`: заголовки, статические библиотеки и CMake-пакеты. Для запуска сервиса SDK не требуется.

Дополнительный SDK содержит заголовки закрытой AoIP-lib и распространяется для разработчиков с доступом в [закрытом релизе AoIP-lib](https://github.com/danrey-bilo/AoIP-lib/releases/tag/v2.5.0). В публичный платформенный релиз входит runtime-пакет.

Профиль новой установки: **8×8 / 192 kHz / PCM24 / capture32**. При обновлении существующий профиль устройства сохраняется. Wi-Fi остаётся для обслуживания, аудио использует Ethernet. Платформенная библиотека не отключает USB, Bluetooth и GPIO.

## Возможности

| Параметр | Поддержка |
|---|---|
| Плата | Raspberry Pi 5 Model B |
| Система | Debian 13 ARM64 / PREEMPT_RT |
| Потоки AoIP | CPU0; RX/TX — FIFO 70 |
| Профиль | `/var/lib/piaoip/profile.txt` |
| Сеть | `/etc/piaoip/peer.conf` |
| Аудиопрофили | До 64 каналов в направлении; 44,1–192 кГц; PCM16/24/32 |
| Транспорт | PiAoIP UDP/IPv4 через Ethernet; не AES67 и не Dante |

## Документация

[Установка и сборка](docs/BUILD.ru.md) · [API платформы](docs/API.ru.md) · [Архитектура](docs/ARCHITECTURE.ru.md) · [Проверки](docs/VALIDATION.ru.md)

Основной язык документации — английский; у актуальных руководств есть русские версии. Версия 2.5.0 — **предварительный выпуск**. Проверка сборки и пакетов не подтверждает физическую работу ADC/DAC или гарантированную задержку. Сервис Pi генерирует и проверяет синтетический PCM; для физической звуковой карты нужен аппаратный аудиобэкенд.

## Компоненты проекта

| Репозиторий | Назначение |
|---|---|
| [AoIP-lib](https://github.com/danrey-bilo/AoIP-lib) | Протокол и переносимые библиотеки |
| [Win11-asio-AoIP](https://github.com/danrey-bilo/Win11-asio-AoIP) | Драйвер ASIO и панель Windows |
| [Pi4-AoIP](https://github.com/danrey-bilo/Pi4-AoIP) | Сервис Raspberry Pi 4 / PREEMPT_RT |
| [Pi5-AoIP](https://github.com/danrey-bilo/Pi5-AoIP) | Сервис Raspberry Pi 5 / PREEMPT_RT |

## Лицензия

Личное некоммерческое использование бесплатно. Для коммерческого использования требуется отдельная платная письменная лицензия. См. [LICENSE](LICENSE) и [русское пояснение](docs/LICENSE-RU.md).
