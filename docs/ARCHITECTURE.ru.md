[English](ARCHITECTURE.md) | **Русский**

# Архитектура платформы Raspberry Pi 5

```mermaid
sequenceDiagram
  participant S as systemd
  participant P as Pi5 main
  participant B as Pi5AoIP platform
  participant R as AoIP peer runtime
  S->>P: User piaoip / CPU0 / RT limits
  P->>B: inspect + initialize
  B-->>P: Pi5 / RT / CPU policy ready
  P->>B: find wired IPv4
  P->>R: run_service(bind_ipv4)
  R->>B: configure each thread role
  R->>R: TX + RX + control + reporter
```

В `src/rpi5.cpp` нет собственной копии PCM, CRC, UDP control или TX-loop. Эти части
приходят из закреплённой AoIP-lib. `apps/main.cpp` проверяет окружение, ждёт Ethernet
и передаёт выбранный IPv4 общему сервису. Системная адресация не изменяется.

## CPU и права

`initialize()` ограничивает главный поток CPU0 до создания остальных потоков.
RX и TX закрепляются за CPU0/FIFO70, control/reporter за CPU0/SCHED_OTHER.
Systemd задаёт общую маску и разрешение RT. CPU1/CPU2/CPU3 не используются даже при сбое
настройки: демон завершается при отказе применить thread policy.

Пакетная служба `pi-aoip-lan` закрепляет IRQ выбранного Ethernet за CPU0.
Остальные IRQ остаются политикой ОС. Изоляция эффектов, RT-ядро, Ethernet full duplex и
достаточное охлаждение входят в подготовку системы, а не устанавливаются библиотекой.
На общей памяти/шине возможна конкуренция с эффектами даже при разных CPU.

## Жизненный цикл пакета

DEB содержит исполняемый файл, unit, conffile, средства настройки и документацию.
Пользователь piaoip владеет состоянием. Обновление не теряет настройки и не запускает
вручную остановленную службу. Удаление сохраняет состояние для восстановления.
SDK-пакет отдельный: для работы демона компилятор и заголовки не нужны.

## Расширение аудиобэкенда

Следующий уровень интеграции — ADC/DAC и обмен с эффектами через ограниченные очереди
с явным владением памятью, clock/drift control и xrun-диагностикой. Сейчас сервис
синтетический; этот репозиторий предоставляет платформенную основу, а не ALSA-драйвер.
