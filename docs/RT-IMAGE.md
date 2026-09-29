# Raspberry Pi 5: RT-образ и проводная сеть

Основа: Raspberry Pi OS Lite 64-bit / Debian 13 ARM64. Проверенная плата:
Raspberry Pi 5 Model B Rev 1.1, 4 ГБ. Образ создаётся на диске ПК через
выделенный Ethernet; USB-носитель Pi используется только как источник файлов.

## Ядро

Используется официальный пакет `linux-image-rpi-v8-rt`, проверенная версия
`6.18.50+rpt-rpi-v8-rt`. В архиве Trixie на дату 2026-09-29 отдельного
`linux-image-rpi-2712-rt` нет. Универсальный вариант v8 поддерживает BCM2712
и RP1: `CONFIG_PINCTRL_RP1=y`, `CONFIG_PINCTRL_BCM2712=y`, `CONFIG_MFD_RP1=y`.
Он использует страницы 4 КиБ и `CONFIG_PREEMPT_RT=y`.

Raspberry Pi перечисляет Pi5 среди устройств с поддержкой `kernel8.img`:
[официальная таблица загрузочных файлов](https://www.raspberrypi.com/documentation/computers/configuration.html).
Драйверы RP1 также входят в
[официальную конфигурацию v8](https://github.com/raspberrypi/linux/blob/rpi-6.18.y/arch/arm64/configs/bcm2711_defconfig).

Установка на чистую Lite 64-bit с настроенным Wi-Fi Интернетом:

```sh
sudo apt update
sudo apt install linux-image-rpi-v8-rt
```

Сохраните `config.txt` и `cmdline.txt` из `/boot/firmware` перед настройкой.
Штатные `kernel_2712.img` и `initramfs_2712` остаются доступны для возврата.
В конце `/boot/firmware/config.txt`:

```ini
[pi5]
kernel=kernel8_rt.img
initramfs initramfs8_rt followkernel
[all]
```

Добавьте к существующей **единственной строке** `/boot/firmware/cmdline.txt`:

```text
isolcpus=domain,managed_irq,2-3 irqaffinity=0-1 kthread_cpus=0-1
```

Не меняйте `root=PARTUUID=...` установленной системы. `nohz_full` этот
официальный RT-вариант не включает, поэтому параметр не добавляется.
Для systemd используется `/etc/systemd/system.conf.d/90-pi5-rt-housekeeping.conf`:

```ini
[Manager]
CPUAffinity=0 1
```

После перезагрузки проверяются `uname -a`, `CONFIG_PREEMPT_RT=y` в
`/boot/config-$(uname -r)` и `2-3` в `/sys/devices/system/cpu/isolated`.
Файл `/sys/kernel/realtime` у проверенной сборки отсутствует; runtime
распознаёт RT по строке `PREEMPT_RT` в `uname`.

Wi-Fi, Bluetooth, USB, Ethernet, GPIO и исходные настройки дисплея/камеры
сохраняются. Governor устанавливается в `performance`, штатное термоуправление
и вентилятор остаются активны.

## LAN без Интернета

| Узел | IPv4 | Маска | Шлюз/DNS на Ethernet |
|---|---|---|---|
| ПК | 192.168.1.1 | 255.255.255.0 | не задаются |
| Pi5 / eth0 | 192.168.1.2 | 255.255.255.0 | не задаются |

Для нового профиля NetworkManager:

```sh
sudo nmcli connection add type ethernet ifname eth0 con-name Pi5-AoIP-LAN \
  ipv4.method manual ipv4.addresses 192.168.1.2/24 ipv4.never-default yes \
  ipv4.ignore-auto-dns yes ipv6.method link-local \
  connection.autoconnect yes connection.autoconnect-priority 999
sudo nmcli connection up Pi5-AoIP-LAN
ip route get 192.168.1.1
```

Результат должен указывать `dev eth0 src 192.168.1.2`. Интернет проходит
через Wi-Fi. SSH и SFTP для передачи файлов используют `admin@192.168.1.2`.
Проверяется `1000Mb/s`, `Full`, `Link detected: yes` в `ethtool eth0`.
Отсутствие ответа ПК на ICMP само по себе не означает отказ SSH/UDP: Windows
может фильтровать ping.

## Образ на ПК

Сборочные средства находятся в локальном AoIP-debug-tool, `tools/pi5`.
NBD-сервер ПК слушает только loopback; временный обратный SSH-туннель
работает через `192.168.1.2`. Pi записывает файловые системы в `/dev/nbd0`,
за которым находится **файл на ПК**. Разделы USB-источника не форматируются.

Образ 4 ГиБ содержит MBR, FAT32 boot 512 МиБ, ext4 root, RT-ядро, firmware,
сохранённую конфигурацию и установленную службу AoIP 2.2.0. Профиль
8×8 / 192 кГц / PCM32, capture24; Ethernet coalescing 0, EEE off. Копируются
используемые файлы; swap, временные файлы и APT-кэш исключаются. PARTUUID
в boot cmdline и fstab переписываются под новую таблицу разделов.

Итоговая копия 2.2 получена на ПК из исходного образа, ранее сохранённого
через LAN. Проверенные на настоящей Pi ARM64 runtime/SDK DEB применены к
копии через `tools/pi5/update-image-on-pc.sh` в WSL. Этот шаг не запускает
ARM64 maintainer scripts на x86: их работа отдельно проверена нативной
установкой на Pi. Файлы пакетов, dpkg metadata, профиль и включение службы
LAN обновлены; FAT boot и PARTUUID сохранены без изменений. Никакой образ
на медленный USB-носитель Pi не записывается.

Проверки: `fsck.vfat -n`, `e2fsck -fn`, загрузочные файлы, service/profile
и SHA256 raw/XZ. Загрузка RT-системы на Pi5 проверена. Сам образ на отдельный
носитель в рамках этой работы не записывался; отдельная загрузка из него
ещё не проверена. После будущей записи на больший носитель файловую систему
можно расширить через `sudo raspi-config` → Advanced Options → Expand Filesystem.

Образ персональный: он сохраняет существующего пользователя и настройки
исходной системы. В этот Git-репозиторий публикуются исходники платформы;
raw/XZ образы и DEB-артефакты хранятся на ПК.
