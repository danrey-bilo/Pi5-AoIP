# Проверка Pi5 AoIP — 2026-09-29

Плата: Raspberry Pi 5 Model B Rev 1.1, 4 ГБ. Raspberry Pi OS Lite 64-bit,
Debian 13, ядро `6.18.50+rpt-rpi-v8-rt`, `CONFIG_PREEMPT_RT=y`, страницы 4 КиБ.
AoIP-lib закреплён на `8a787513bc2d0af68102dbafd086325398346c50`.

## Сборка и система

- Нативная ARM64-сборка GCC 14.2.0, Release, `-Wall -Wextra -Werror` прошла.
- Четыре CTest: core, example_core, example_peer, pi5_platform_policy — прошли.
- Тест платформы проверил настоящую affinity и FIFO-приоритет каждой роли,
  модель/RT/изоляцию, отказ выбирать Wi-Fi и несуществующий интерфейс.
- Runtime DEB установлен и служба запущена от пользователя `piaoip`.
- TX: CPU1/FIFO70; RX: CPU0/FIFO70; control/reporter: CPU1/SCHED_OTHER.
- SDK проверен отдельной программой через `find_package(Pi5AoIP 2.1 CONFIG)`
  и экспортированный target `PiAoIP::rpi5`.
- Ethernet 1 Гбит/с full duplex, Wi-Fi, Bluetooth, USB и GPIO доступны после
  загрузки RT; `throttled=0x0`.

30 секунд cyclictest, один поток CPU0, FIFO80, период 1 мс: min 1 мкс,
avg 2 мкс, max 47 мкс. Часть окна включала нативную сборку на CPU0/CPU1.
Это задержка пробуждения планировщика, а не задержка аудио.

## Прямой UDP peer

30 секунд, 64×64, 192 кГц, PCM32, 5 кадров/пакет.

| Метрика | Pi5 | Windows |
|---|---:|---:|
| RX пакеты | 1 152 000 | 1 155 592 |
| Потерянные пакеты | 0 | 1 857 |
| Bad/CRC | 0 | 0 |
| Late TX | 0 | 1 660 |
| Max TX late | 17,6 мкс | 169,4 мкс |
| Max RX gap | 910,5 мкс | 1 752,4 мкс |
| Skipped source frames | 0 | 0 |

Windows счётчик lost был 1857 уже в первой секундной записи и далее не рос.
Он включает запуск при уже идущем потоке Pi. Строгой гарантии отсутствия
потерь по этому тесту нет. RX totals измерены в несовпадающих временных окнах.

Нагрузка Pi5 за 30 секунд: CPU0 среднее 51,3% / максимум секунды 65,0%;
CPU1 23,5% / 25,6%; CPU2 и CPU3 среднее и максимум 0,0%.

## Windows ASIO с Pi5

Проверен существующий Windows ASIO 2.1.0 без изменения кода общего протокола.
Отдельный тестовый INI: Pi `192.168.1.2`, 64 входа + 64 выхода, 192 кГц,
PCM32, ASIO buffer128, safety512. Синтетический поток возвращался через
ASIO-хост; маркеры передавались по каналу 32.

| Цифровой RTT | Значение |
|---|---:|
| Маркеры/пакеты измерения | 1 152 086 |
| min | 3,433 мс |
| p50 | 3,471 мс |
| p95 | 3,532 мс |
| p99 | 3,697 мс |
| max | 4,002 мс |

30 секунд: RX 1152214, TX 1152086, callbacks 45004. Нулевые счётчики:
invalid, RX/TX queue overflow, late/missing frames, resync, deadline misses,
skipped frames, TX expired/errors, MMCSS failures, host overruns,
expired output frames, TX retries и ASIO time errors.

Максимумы: callback 113,5 мкс, wake late 245,1 мкс, RX gap 391,8 мкс.
Pi за ASIO-сеанс не добавил lost/bad/skipped_source_frames/tx_expired.

Этот короткий тест подтверждает работу транспорта и совместимость с ASIO
на указанном профиле. Он не заменяет длительную квалификацию, проверку других
буферов, DAW и аналогового ADC/DAC. Peer пока синтетический; физический
ADC/DAC backend, эффекты, PTP/AES67/Dante не реализованы.
