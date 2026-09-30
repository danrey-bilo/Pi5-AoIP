**English** | [Русский](VALIDATION-2026-09-29.ru.md)

# Historical Pi 5 validation — 2026-09-29

These are measurements of versions 2.1–2.3.1. They are not tests of 2.4.3 and do not describe an arbitrary computer. The detailed chronological record and command context are retained in the Russian edition.

Reference system: Raspberry Pi 5 Model B 4 GB, Debian 13 ARM64, `6.18.50+rpt-rpi-v8-rt`, Gigabit Ethernet and Windows 11 / Intel I225-V. The peer generated synthetic PCM; physical ADC/DAC hardware was absent.

## 2.3.1, 8×8 / 192 kHz / PCM32

Native ARM64 and Windows suites passed 4/4 each. The Pi 5 SDK consumer verified all four AoIP roles on CPU0, with RX/TX FIFO70. In a 30-second active window, the service consumed 15.50% of CPU0; other CPUs had zero service-thread runtime, not zero whole-system load.

Each row below is a separate 180-second digital Pi → ASIO callback → Pi loop:

| ASIO/LAN frames | RTT p50 / p99 / max, ms | Missing / late frames | Deadline misses | Callback overruns |
|---|---|---:|---:|---:|
| 64/256 | 1.695 / 1.783 / 2.777 | 16 / 16 | 1 | 0 |
| 64/320 | 2.027 / 2.113 / 4.025 | 80 / 80 | 0 | 0 |
| 64/448, run 1 | 2.695 / 2.788 / 4.223 | 0 / 0 | 0 | 0 |
| 64/448, run 2 | 2.696 / 2.785 / 4.182 | 0 / 0 | 0 | 0 |
| 64/512 | 3.032 / 3.121 / 4.599 | 0 / 0 | 0 | 0 |
| 32/512 | 2.869 / 2.963 / 4.135 | 0 / 0 | 0 | 1 |

Both 64/448 runs had RTT p95 2.768 ms and minimum 2.532 ms. Combined traffic was about 4.32 million Windows RX packets, 2.16 million TX packets and 2.15 million RTT markers. Invalid/CRC, queue overflow, resync, skipped frames, expired TX, TX errors, MMCSS/ASIO-time errors and host overruns were zero. Pi lost/bad/skipped/expired counters remained zero; it accepted 10/9 reordered packets. Windows maximum RX gaps were 1.947/1.744 ms and maximum callbacks 218.7/129.7 µs.

Local synthetic fixtures covered 4×4 PCM24/96 kHz, 8×8 PCM16/48 kHz and 64×64 PCM32/192 kHz discovery/profile/masks, PCM zero transitions and legacy compatibility. They did not qualify physical 4- or 64-channel converters.

## Earlier results

In 2.1, a 30-second digital 64×64/192 kHz/PCM32 ASIO128/LAN512 loop reported RTT min 3.433, p50 3.471, p95 3.532, p99 3.697 and max 4.002 ms, with the recorded Windows/Pi error counters at zero. A separate direct UDP startup test had nonzero Windows loss and cannot be described as loss-free.

In 2.2, a 180-second 8×8 ASIO32/LAN512 loop reported RTT min 2.774, p50 2.849, p95 2.972, p99 2.993 and max 5.397 ms. Active-run missing/late/deadline/overflow/expired/TX/ASIO errors were zero; smaller LAN buffers failed in other runs. A separate five-second silent run had one callback overrun, retained in the record.

The historical 2.3.1 image passed structural/hash checks but was not separately booted from a newly written medium. Kernel/peripheral operation on the running reference Pi was verified. Analog converter latency, a physical audio backend and universal sub-millisecond reliability were not established.
