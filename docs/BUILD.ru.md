[English](BUILD.md) | **Русский**

# Raspberry Pi 5: установка, сборка и восстановление

## Требования

Raspberry Pi 5 Model B, Debian 13 ARM64, работающее PREEMPT_RT-ядро, четыре online CPU и проводной Gigabit Ethernet с MTU 1500. RT-ядро устанавливается отдельно. Пакет не меняет ядро и IP-адреса ОС. Wi-Fi можно оставить для Интернета и обслуживания.

## Установка runtime

Скачайте `piaoip-rpi5_2.4.3-1_arm64.deb` из [релиза 2.4.3](https://github.com/danrey-bilo/Pi5-AoIP/releases/tag/v2.4.3).

```sh
sudo apt install ./piaoip-rpi5_2.4.3-1_arm64.deb
sudo piaoip-configure --interface eth0 --peer 192.168.50.1 --restart
/usr/lib/piaoip/aoip_peer_rpi5 --check
systemctl status pi-aoip --no-pager
journalctl -u pi-aoip -b --no-pager
```

Замените `192.168.50.1` на **Ethernet-адрес ПК**. Пример прямого соединения: ПК `192.168.50.1/24`, Pi `192.168.50.2/24`, без шлюза/DNS на аудио-LAN. Подсеть не должна пересекаться с другими сетями. Для DHCP закрепите адрес ПК. До первой настройки сервис завершает запуск с подсказкой.

| Путь | Назначение |
|---|---|
| `/usr/lib/piaoip/aoip_peer_rpi5` | Сервис |
| `/etc/piaoip/peer.conf` | Интерфейс и IPv4 ПК; conffile dpkg |
| `/var/lib/piaoip/profile.txt` | Профиль, владелец `piaoip` |
| `/usr/share/doc/piaoip-rpi5` | Инструкции EN/RU и лицензия |

Профиль новой установки: `8 8 192000 32 16` (`inputs outputs rate bits capture_frames`). При обновлении профиль сохраняется. Для чтения используйте `sudo piaoip-configure --show`. Буферы Windows задаются отдельно. Pi 5 ждёт подписку ASIO и прекращает аудиотрафик при завершении сессии. Дополнительный LAN-сервис применяет политику Ethernet из пакета: [подробности](LOW-LATENCY.ru.md).

## Потоки

RX/TX — CPU0/FIFO70, control/reporter — CPU0/SCHED_OTHER. Сервис ограничен CPU0; CPU2/CPU3 свободны от AoIP. Unit задаёт `LimitRTPRIO=80`, `LimitMEMLOCK=infinity`. Ошибка применения политики сообщается явно.

```sh
systemctl show pi-aoip -p MainPID -p CPUAffinity
ps -eLo pid,tid,psr,cls,rtprio,comm | grep aoip
```

## Сборка

```sh
git clone --recurse-submodules https://github.com/danrey-bilo/Pi5-AoIP.git
cd Pi5-AoIP
sudo apt install build-essential cmake
taskset -c 0,1 cmake -S . -B build -DCMAKE_BUILD_TYPE=Release
taskset -c 0,1 cmake --build build --parallel 2
./build/bin/aoip_peer_rpi5 --version
./build/bin/aoip_peer_rpi5 --check
cmake --install build --prefix "$PWD/sdk"
```

`external/AoIP-lib` закреплён на совместимом релизе. Можно передать `-DAOIP_SOURCE_DIR=/path/to/AoIP-lib`. Без исходников зависимости используется установленный SDK `AoIP 2.4.3` через `CMAKE_PREFIX_PATH`. AoIP-lib — закрытая зависимость. Нужен разрешённый доступ к её репозиторию или SDK. Публичный архив содержит только платформенную часть, без закрытых исходников.

`--check` не запускает PCM. Сборка на другой ARM64-плате не подтверждает работу на Pi 5.

## SDK

`piaoip-rpi5-sdk_2.4.3-1_arm64.deb` содержит статические библиотеки core/peer/platform, заголовки и CMake exports для Debian 13 ARM64/GCC 14. Для работы сервиса он не нужен.

```cmake
find_package(Pi5AoIP 2.4.3 CONFIG REQUIRED)
target_link_libraries(my_service PRIVATE Pi5AoIP::platform)
```

## Обновление и удаление

```sh
sudo apt install ./piaoip-rpi5_2.4.3-1_arm64.deb
sudo apt remove piaoip-rpi5
```

Перед обновлением закройте Windows-аудиохост. Конфигурация сохраняется, жизненным циклом сервиса управляют скрипты пакета. Пользовательский локальный unit автоматически не заменяется. Известная старая установка сохраняется в `/var/lib/piaoip/legacy-2.1`: это имя каталога миграции, а не версия текущего пакета. После удаления остаются состояние и пользователь сервиса. Для отката используйте сохранённый пакет и настройки.

Пакеты Pi 4 и Pi 5 используют одинаковые пути и не устанавливаются вместе. См. [границы проверки](VALIDATION.ru.md).


Дополнительный SDK содержит заголовки закрытой AoIP-lib и распространяется для разработчиков с доступом в [закрытом релизе AoIP-lib](https://github.com/danrey-bilo/AoIP-lib/releases/tag/v2.4.3). В публичный платформенный релиз входит runtime-пакет.
