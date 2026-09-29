# Raspberry Pi 5: сборка, установка и настройка

## Требования

Raspberry Pi 5 Model B, Raspberry Pi OS Lite 64-bit / Debian 13 ARM64,
PREEMPT_RT, четыре online CPU и проводной Gigabit Ethernet с MTU 1500.
Пакет проверен с ядром `6.18.50+rpt-rpi-v8-rt`; само ядро устанавливается отдельно.
Wi-Fi может обеспечивать Интернет независимо от аудио-LAN.

## Зависимость AoIP-lib

`external/AoIP-lib` — git submodule, закреплённый конкретным commit. После обычного
clone выполните `git submodule update --init --recursive`. Для приватных
репозиториев Git должен иметь доступ и к основному проекту, и к AoIP-lib.
Токены и пароли в CMake или `.gitmodules` не сохраняются.

Вместо submodule можно передать `-DAOIP_SOURCE_DIR=/path/to/AoIP-lib` либо
установить SDK AoIP-lib и передать `-DCMAKE_PREFIX_PATH=/path/to/sdk`.
Приоритет: явный source path → submodule → установленный пакет.
Чтобы явно использовать установленный SDK, не инициализируйте submodule.
Обновление зависимости: выберите проверенный commit внутри submodule и
закоммитьте новый gitlink в родительском репозитории.

## Сборка библиотеки и службы

```sh
sudo apt install build-essential cmake dpkg-dev
taskset -c 0 cmake -S . -B build -DCMAKE_BUILD_TYPE=Release
taskset -c 0 cmake --build build --parallel 2
./build/bin/aoip_peer_rpi5 --check
```

`--check` только читает окружение; обычный запуск включает политику CPU0.
Сборка платформы на другом Linux может проверить компиляцию, но не доказывает
работу Pi5/RT. Для SDK: `cmake --install build --prefix "$PWD/sdk"`.

## DEB

Скрипты создания пакетов находятся в закрытом
[AoIP-debug-tool](https://github.com/danrey-bilo/AoIP-debug-tool/blob/main/docs/BUILD.md).
Основная CMake-сборка не зависит от них. Для установки скопируйте полученный DEB
в текущий каталог Pi:

```sh
sudo apt install ./piaoip-rpi5_2.3.1-1_arm64.deb
sudo piaoip-configure --interface eth0 --peer 192.168.1.1 --restart
/usr/lib/piaoip/aoip_peer_rpi5 --check
systemctl status pi-aoip.service --no-pager
```

Замените `192.168.1.1` на Ethernet IPv4 компьютера. Проверенная прямая LAN:
Pi5 — `192.168.1.2/24`, ПК — `192.168.1.1/24`, маска `255.255.255.0`.
Эти адреса команда настройки не назначает ОС. На аудио-LAN шлюз/DNS не нужны;
Интернет можно оставить на Wi-Fi. Для DHCP закрепите адрес компьютера.
До `piaoip-configure` служба заканчивает запуск с кодом 2 и понятной подсказкой.
Автоматический выбор IP компьютера в Linux не поддерживается этим выпуском.

| Путь | Назначение |
|---|---|
| `/usr/lib/piaoip/aoip_peer_rpi5` | Исполняемый файл |
| `/etc/piaoip/peer.conf` | Interface и IPv4 компьютера, conffile dpkg |
| `/var/lib/piaoip/profile.txt` | Сохранённый аудиопрофиль, владелец piaoip |
| `journalctl -u pi-aoip.service -b` | Журнал текущей загрузки |

Число физических входов/выходов задаётся локально на каждой Pi. Для устройства
8×8: `sudo piaoip-configure --inputs 8 --outputs 8 --restart`.
Windows получает эти числа через discovery; изменить их через панель, INI или
сетевой запрос нельзя. Пункты «Каналы» включают/отключают использование уже
существующих каналов, сохраняя их физические номера и число каналов ASIO.

Профиль по умолчанию: 8×8, 192 кГц, PCM32, до 16 кадров в сетевом пакете.
Служба ждёт подписку ASIO и не отправляет фоновый PCM. v3 поддерживает маски
каналов, точный цифровой ноль, штатную отписку и 3-секундную аренду сессии.
Другие устройства до 64×64 настраиваются на самой Pi; Windows-драйвер общий.
Частота, PCM и буферы настраиваются в панели Windows. См. [режим 8×8](LOW-LATENCY.md).
`pi-aoip-lan.service` устанавливает rx-usecs/tx-usecs=0 и отключает EEE на
выбранном Ethernet-интерфейсе. Wi-Fi для Интернета не затрагивается.
IRQ выбранного Ethernet также закрепляется за CPU0.
Размер ASIO-буфера и guard настраиваются на Windows отдельно.
RX/TX: CPU0/FIFO70; control/reporter: CPU0/SCHED_OTHER.
`LimitRTPRIO=80`, `LimitMEMLOCK=infinity`, `CPUAffinity=0` задаёт systemd unit.
При невозможности применить ограничения процесс завершается, а не запускается
на произвольных CPU. CPU1/CPU2/CPU3 не используются сетевой службой.

## Обновление и удаление

```sh
sudo apt install ./piaoip-rpi5_2.3.1-1_arm64.deb
sudo apt remove piaoip-rpi5
```

Активная служба перезапускается при обновлении; вручную остановленная остаётся
остановленной. Конфигурация сохраняется по правилам dpkg. `/var/lib/piaoip`
и пользователь piaoip сохраняются после remove/purge. Для чтения профиля при
необходимости используйте `sudo`: файл намеренно доступен только владельцу.

Миграция старой установки с известным `/home/admin/aoip_peer` сохраняет unit
в `/var/lib/piaoip/legacy-2.1`. Произвольный локальный unit автоматически не заменяется.
Для ручного возврата после удаления пакета восстановите сохранённый unit,
выполните `systemctl daemon-reload` и `systemctl enable --now pi-aoip.service`.
Исходные бинарник и профиль старой установки должны оставаться на месте.

## Своя программа

```cmake
find_package(Pi5AoIP 2.3.1 CONFIG REQUIRED)
target_link_libraries(my_service PRIVATE Pi5AoIP::platform)
```

Пакет `piaoip-rpi5-sdk` необязателен для работающей службы. Он содержит архивы общего
AoIP и платформы, заголовки, CMake exports и лицензии. Используйте совместимый
Debian 13 ARM64 toolchain. [Публичный API](API.md).

## Проверки разработчика

Тесты SDK, жизненного цикла пакета и утилита `piaoip-doctor` перенесены в
[AoIP-debug-tool](https://github.com/danrey-bilo/AoIP-debug-tool).
Они не входят в библиотеку или пользовательский DEB.
Результаты проверки конкретной Pi5: [VALIDATION.md](VALIDATION.md).
