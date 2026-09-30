**English** | [Русский](RELEASE-2.4.3.ru.md)

# Pi5-AoIP 2.4.3 — Raspberry Pi 5 runtime

Development preview · `v2.4.3` · 2026-09-30

Runtime, package metadata and platform library aligned to 2.4.3 with the matching shared transport. Fresh-install profile is 8×8, 192 kHz, PCM32; existing device profiles are preserved. AoIP roles stay on CPU0, with CPU2/CPU3 free of AoIP. The Pi 5 wrapper uses leased V3 subscriptions and waits for ASIO start. Documentation, project banners and repository metadata have been refreshed in English and Russian.

## Downloads

| File | Purpose |
|---|---|
| `piaoip-rpi5_2.4.3-1_arm64.deb` | Runtime for Raspberry Pi 5 / Debian 13 ARM64 / PREEMPT_RT |
| `Pi5-AoIP-2.4.3-source.zip` | Platform source; private AoIP dependency not included |
| `SHA256SUMS.txt` | Checksums for all release files |
| `LICENSE.txt` | Project license terms |

## Installation

Install with `sudo apt install ./piaoip-rpi5_2.4.3-1_arm64.deb`, then configure the PC Ethernet address with `sudo piaoip-configure --interface eth0 --peer YOUR_PC_IP --restart`. An RT kernel and OS IP addresses must already be configured. Optional SDKs containing private core headers are in the [private AoIP-lib release](https://github.com/danrey-bilo/AoIP-lib/releases/tag/v2.4.3) for authorized developers. Runtime packages do not require access to that repository.

## Validation

Native Debian 13 ARM64 build passed four CTest checks, including actual Pi 5 affinity, FIFO policy and wired-interface checks. Runtime/SDK package metadata and contents were inspected, and an extracted SDK passed a separate exact-version CMake consumer build.

Release preparation did not install the MSI or new DEBs, restart the running Pi service or change the Windows profile. No new long-duration DAW or physical ADC/DAC test was performed. The peer remains synthetic PCM; this release does not claim guaranteed sub-millisecond or physical converter latency.

## Licenses

Personal noncommercial use is free; commercial use requires a separate written license. The ASIO component is additionally subject to Steinberg terms.
