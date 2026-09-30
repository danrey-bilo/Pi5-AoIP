**English** | [Русский](BUILD.ru.md)

# Raspberry Pi 5: installation, build and recovery

## Requirements

Raspberry Pi 5 Model B, Debian 13 ARM64, a working PREEMPT_RT kernel, four online CPUs and wired Gigabit Ethernet with MTU 1500. Install and verify the RT kernel separately. The package does not replace the kernel or configure OS IP addresses. Wi-Fi can remain available for Internet and maintenance.

## Install the runtime

Download `piaoip-rpi5_2.4.3-1_arm64.deb` from the [2.4.3 release](https://github.com/danrey-bilo/Pi5-AoIP/releases/tag/v2.4.3).

```sh
sudo apt install ./piaoip-rpi5_2.4.3-1_arm64.deb
sudo piaoip-configure --interface eth0 --peer 192.168.50.1 --restart
/usr/lib/piaoip/aoip_peer_rpi5 --check
systemctl status pi-aoip --no-pager
journalctl -u pi-aoip -b --no-pager
```

Replace `192.168.50.1` with the **PC Ethernet address**. Example dedicated link: PC `192.168.50.1/24`, Pi `192.168.50.2/24`, no Ethernet gateway or DNS. Choose a subnet that does not overlap other networks. With DHCP, reserve the PC address. Before initial configuration, startup exits with a configuration hint.

| Path | Purpose |
|---|---|
| `/usr/lib/piaoip/aoip_peer_rpi5` | Runtime binary |
| `/etc/piaoip/peer.conf` | Wired interface and PC IPv4; dpkg conffile |
| `/var/lib/piaoip/profile.txt` | Device profile owned by service user `piaoip` |
| `/usr/share/doc/piaoip-rpi5` | English/Russian instructions and license |

Fresh profile: `8 8 192000 32 16` (`inputs outputs rate bits capture_frames`). Existing profiles are preserved. Use `sudo piaoip-configure --show` to read protected state. Windows buffers are configured separately. The Pi 5 service waits for an ASIO subscription and stops audio when the session ends. Its LAN service applies the packaged Ethernet tuning policy; see [low-latency operation](LOW-LATENCY.md).

## Scheduling

RX and TX: CPU0 / FIFO70; control and reporter: CPU0 / SCHED_OTHER. All service threads are confined to CPU0; CPU2 and CPU3 remain free of AoIP. The unit grants `LimitRTPRIO=80` and `LimitMEMLOCK=infinity`. Startup reports a board/RT/affinity failure rather than silently claiming the requested policy.

Check real placement while streaming:

```sh
systemctl show pi-aoip -p MainPID -p CPUAffinity
ps -eLo pid,tid,psr,cls,rtprio,comm | grep aoip
```

## Build from source

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

`external/AoIP-lib` is pinned to the compatible release. An explicit `-DAOIP_SOURCE_DIR=/path/to/AoIP-lib` overrides it. Without either source tree, CMake can use an installed `AoIP 2.4.3` SDK through `CMAKE_PREFIX_PATH`. AoIP-lib is private. Source builds require authorized repository access or its compatible SDK. Public source archives include only this platform repository, not the private dependency.

`--check` reads the host without starting PCM. Compiling on another ARM64 board verifies a build, not operation on Pi 5.

## Optional SDK

`piaoip-rpi5-sdk_2.4.3-1_arm64.deb` contains core/peer/platform static libraries, headers and CMake exports. It is intended for a compatible Debian 13 ARM64/GCC 14 toolchain; the runtime does not need it.

```cmake
find_package(Pi5AoIP 2.4.3 CONFIG REQUIRED)
target_link_libraries(my_service PRIVATE Pi5AoIP::platform)
```

## Upgrade and remove

```sh
sudo apt install ./piaoip-rpi5_2.4.3-1_arm64.deb
sudo apt remove piaoip-rpi5
```

Close the Windows audio host before upgrading. Package scripts preserve existing peer/profile settings and handle the service lifecycle. A custom local systemd unit is not overwritten automatically. The known legacy unit is backed up under `/var/lib/piaoip/legacy-2.1`; this directory name is a migration identifier, not the installed version. State and the service user remain after removal. Restore a saved package/configuration for rollback.

Pi 4 and Pi 5 runtime packages use the same service/state paths and cannot be installed together. Read [validation](VALIDATION.md) before treating the release as qualified for a workload.


The optional SDK includes headers from the private AoIP-lib dependency and is distributed to authorized developers in the [private AoIP-lib release](https://github.com/danrey-bilo/AoIP-lib/releases/tag/v2.4.3). The public platform release contains the runtime package.
