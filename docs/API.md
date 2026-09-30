**English** | [Русский](API.ru.md)

# Pi 5 platform API

Include `<aoip/rpi5.hpp>` and link `Pi5AoIP::platform`. The platform library depends on `AoIP::peer`. Build-tree alias `PiAoIP::rpi5` remains available.

| Function | Contract |
|---|---|
| `inspect_host()` | Reads board model, kernel/RT status and isolated CPU list |
| `initialize(error)` | Validates Pi 5/PREEMPT_RT and installs the process/thread CPU policy before workers start |
| `configure_thread(role)` | Applies the role's affinity and scheduler; returns false on failure |
| `find_wired_endpoint(interface, endpoint, error)` | Selects an up/running physical wired IPv4 interface; rejects Wi-Fi and ambiguous automatic selection |

`HostStatus` contains model, kernel, isolated CPUs and the board/RT booleans. `Endpoint` contains interface name and IPv4. An empty interface requests automatic selection of the sole eligible wired endpoint. When multiple endpoints exist, select the interface explicitly and use one IPv4 address.

RX and TX: CPU0 / FIFO70; control and reporter: CPU0 / SCHED_OTHER. The library does not configure IRQ affinity, the governor or peripherals. The service package may include additional policy, documented separately. Call `initialize()` before starting `run_service()`; do not change the thread setup callback while workers are active.

The runtime wrapper's `--check` is read-only. Normal startup performs board/RT checks and waits for an eligible wired endpoint. `--version` prints the product version without requiring the target board.

These APIs provide scheduling and endpoint policy, not an ADC/DAC driver. Use a separate hardware backend for physical audio.
