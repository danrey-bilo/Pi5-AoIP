![Pi5-AoIP 2.4.3](docs/assets/header.svg)

# Pi5-AoIP

**English** | [Русский](README.ru.md)

A Raspberry Pi 5 Model B platform library and synthetic PCM service for PREEMPT_RT Linux. All AoIP roles stay on CPU0.

**[Download 2.4.3](https://github.com/danrey-bilo/Pi5-AoIP/releases/tag/v2.4.3)** · **[Release notes](docs/RELEASE-2.4.3.md)** · **[Validation](docs/VALIDATION.md)**

## Get started

Requires **Raspberry Pi 5 Model B, Debian 13 ARM64, PREEMPT_RT and wired Gigabit Ethernet**.

Download [piaoip-rpi5_2.4.3-1_arm64.deb](https://github.com/danrey-bilo/Pi5-AoIP/releases/download/v2.4.3/piaoip-rpi5_2.4.3-1_arm64.deb), then run on the Pi:

```sh
sudo apt install ./piaoip-rpi5_2.4.3-1_arm64.deb
sudo piaoip-configure --interface eth0 --peer 192.168.50.1 --restart
/usr/lib/piaoip/aoip_peer_rpi5 --check
systemctl status pi-aoip --no-pager
```

`192.168.50.1` is an example **PC** address. Configure reachable IPv4 addresses on the PC and Pi first. The package does not assign IP addresses or install an RT kernel. For application development, the optional `piaoip-rpi5-sdk_2.4.3-1_arm64.deb` adds headers, static libraries and CMake packages; it is not needed to run the service.

The optional SDK includes headers from the private AoIP-lib dependency and is distributed to authorized developers in the [private AoIP-lib release](https://github.com/danrey-bilo/AoIP-lib/releases/tag/v2.4.3). The public platform release contains the runtime package.

Fresh-install profile: **8×8 / 192 kHz / PCM32**. Existing device profiles are preserved on upgrade. Wi-Fi remains available for maintenance; audio uses Ethernet. USB, Bluetooth and GPIO are not disabled by the platform library.

## At a glance

| Item | Support |
|---|---|
| Board | Raspberry Pi 5 Model B |
| System | Debian 13 ARM64 / PREEMPT_RT |
| AoIP placement | CPU0; RX/TX use FIFO 70 |
| State | `/var/lib/piaoip/profile.txt` |
| Network configuration | `/etc/piaoip/peer.conf` |
| Audio profiles | Up to 64 channels per direction; 44.1–192 kHz; PCM16/24/32 |
| Transport | PiAoIP UDP/IPv4 over Ethernet; not AES67 or Dante |

## Documentation

[Installation and build](docs/BUILD.md) · [Platform API](docs/API.md) · [Architecture](docs/ARCHITECTURE.md) · [Validation](docs/VALIDATION.md)

English is the primary documentation language. Each maintained guide links to its Russian edition. Version 2.4.3 is a **development preview**: build and package checks do not establish physical ADC/DAC support or a guaranteed latency. The supplied Pi service generates and checks synthetic PCM; a hardware audio backend is still required for a physical sound card.

## Project components

| Repository | Purpose |
|---|---|
| [AoIP-lib](https://github.com/danrey-bilo/AoIP-lib) | Protocol and portable libraries |
| [Win11-asio-AoIP](https://github.com/danrey-bilo/Win11-asio-AoIP) | Windows ASIO driver and settings |
| [Pi4-AoIP](https://github.com/danrey-bilo/Pi4-AoIP) | Raspberry Pi 4 / PREEMPT_RT service |
| [Pi5-AoIP](https://github.com/danrey-bilo/Pi5-AoIP) | Raspberry Pi 5 / PREEMPT_RT service |

## License

Personal, noncommercial use is free. Commercial use requires a separate paid written license. See [LICENSE](LICENSE) and the [Russian explanation](docs/LICENSE-RU.md).
