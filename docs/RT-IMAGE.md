**English** | [Русский](RT-IMAGE.ru.md)

# Pi 5 RT system and wired network

The release provides application packages, not a public SD/USB operating-system image. Existing private image backups can contain user accounts, SSH keys and network configuration and are not release assets.

## Reference RT setup

The September 2026 reference board was a Pi 5 Model B Rev 1.1 with 4 GB RAM, Raspberry Pi OS Lite 64-bit / Debian 13 and official `linux-image-rpi-v8-rt`, kernel `6.18.50+rpt-rpi-v8-rt`. This is a recorded tested configuration, not a statement about the newest available kernel.

The reference boot files are `kernel8_rt.img` and `initramfs8_rt`, with the original stock kernel retained for recovery. Back up `config.txt` and the one-line `cmdline.txt` before editing them. Do not overwrite `root=PARTUUID=...`.

```ini
[pi5]
kernel=kernel8_rt.img
initramfs initramfs8_rt followkernel
[all]
```

The reference `cmdline.txt` adds `isolcpus=domain,managed_irq,2-3 irqaffinity=0-1 kthread_cpus=0-1` to its existing single line. Systemd housekeeping uses `[Manager] CPUAffinity=0 1`; AoIP itself uses CPU0. These are system configuration choices, separate from building the platform library.

After any reboot, verify the actual kernel/RT mode, CPU placement and retained Ethernet, Wi-Fi, Bluetooth, USB and GPIO. A runtime `--check` does not replace those peripheral checks. The platform can detect `PREEMPT_RT` in `uname` even when `/sys/kernel/realtime` is absent.

## Dedicated Ethernet example

Use nonoverlapping addresses such as PC `192.168.50.1/24` and Pi `192.168.50.2/24`; omit the Ethernet gateway/DNS and retain Wi-Fi for Internet. Configure the Pi's normal network manager, then point PiAoIP at the PC address with `piaoip-configure`. This command records service configuration; it does not assign OS addresses.

Check `ip route get 192.168.50.1`, `ethtool eth0`, and the `pi-aoip`/`pi-aoip-lan` service journals. A filtered ICMP echo is not by itself proof that UDP/SSH connectivity has failed.

## Image validation boundary

The historical private 2.3.1 image was structurally checked: filesystems, retained boot partition, package metadata and raw/XZ hashes. It was not separately booted from a newly written device. Version 2.5.0 does not rename that old image or present it as a newly validated release image. Install the 2.5.0 runtime package on a suitable existing system instead.

See [installation](BUILD.md), [release validation](VALIDATION.md), and the [historical record](VALIDATION-2026-09-29.md).
