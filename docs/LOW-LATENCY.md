**English** | [Русский](LOW-LATENCY.ru.md)

# Pi 5 operation and buffer selection

Version 2.4.3 starts a fresh Pi 5 installation with 8 inputs, 8 outputs, 192 kHz and PCM32. The device's physical channel counts are configured locally:

```sh
sudo piaoip-configure --inputs 8 --outputs 8 --restart
```

Windows discovers these counts. **Device → Channels** selects existing channels; it does not reconfigure the physical device. Choose sample rate, bit depth and ASIO/LAN buffers in the Windows panel. The LAN field accepts integers from 0 to 2048 samples. Automatic tuning is not part of the release.

All Pi 5 AoIP roles use CPU0. The package's separate `pi-aoip-lan.service` selects the configured wired interface, requests zero RX/TX interrupt coalescing, disables EEE and assigns its discovered Ethernet IRQs to CPU0. It does not change Windows settings, disable Wi-Fi/USB/Bluetooth/GPIO or claim that every OS task stays off the other cores. Inspect the LAN service journal if the interface does not support a requested ethtool setting.

The service waits for an ASIO subscription. With V3 traffic reduction enabled, only selected host channels carry PCM; exact zero can suspend audio datagrams after three transition markers. A running DAW still receives ASIO callbacks. Stop unsubscribes and an abandoned session expires after three seconds.

## Measured results versus settings

Historical 2.3.1 runs on one Pi 5 / Windows Intel I225-V system passed two 180-second digital tests at ASIO64/LAN448 with zero missing/late frames and deadline misses. Their RTT p50 values were 2.695/2.696 ms, p95 2.768 ms, p99 2.788/2.785 ms and max 4.223/4.182 ms. These were synthetic digital loops, not ADC/DAC measurements. Later Windows work validated different buffers for that computer; neither result is a universal driver default.

For your computer, first choose a stable LAN buffer with ASIO fixed, then reduce ASIO while retaining that LAN value. Record profile, duration, p50/p95/p99/max, missing/late/deadline/overflow/TX expiry/errors and CPU load under the actual DAW workload. Small average RTT alone does not prove stability.

The full historical record is in [2026-09-29 validation](VALIDATION-2026-09-29.md). Current release checks are in [2.4.3 validation](VALIDATION.md). The service remains synthetic PCM; no physical ADC/DAC/I2S backend is supplied.
