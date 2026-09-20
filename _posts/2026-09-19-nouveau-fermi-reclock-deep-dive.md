---
layout: post
title: "Nouveau Fermi Reclocking: Reverse Engineering, Falcon Microcode, and a Call for Hardware Telemetry"
date: 2026-09-19 14:00:00 +0000
author: The AliveOS Project
---

This is a technical deep-dive into [nouveau-fermi-reclock-dkms](https://github.com/Twilight0/nouveau-fermi-reclock-dkms),
an out-of-tree DKMS kernel module that adds bidirectional DDR3 memory
reclocking, GPU core/shader reclocking, and a dynamic frequency governor for
NVIDIA Fermi GPUs (GF100&ndash;GF119). It is also a story about how many
things had to go wrong before a single thing went right.

If you are not interested in MMIO register maps, Falcon microcode disassembly,
or the particular joy of watching a desktop freeze because the memory
controller just hung, skip to the end where I ask for help.

This work was previously discussed in
[Xlibre, Nouveau, and Telegram Bans](/news/2026-09-15-xlibre-nouveau-and-telegram-bans/),
where we outlined the importance of a Wayland-compatible, open-source driver
for legacy hardware. This post is the technical follow-up, documenting the
reverse engineering that made it happen.

---

## The Problem

Nouveau, the open-source NVIDIA driver, has supported Fermi GPUs for years. It
can modeset, it can display, it can do basic 3D. What it cannot do is
reclock. The GPU ships with three performance states defined in the VBIOS:

| P-State | Core | Memory | Voltage | Use Case |
|---------|------|--------|---------|----------|
| `03` | 50 MHz | 135 MHz | 820 mV | Deep idle |
| `07` | 202 MHz | 324 MHz | 820 mV | Desktop |
| `0f` | 590 MHz | 900 MHz | 1030 mV | Full 3D |

In practice, nouveau boots into `07` and stays there. Forever. The memory
clock sits at 324 MHz, yielding roughly 6.48 GB/s of bandwidth on a 192-bit
DDR3 bus. The `0f` pstate offers 900 MHz and 14.4 GB/s. That is a 2.2x
bandwidth improvement that the hardware is perfectly capable of delivering, but
the driver never attempts to use.

The reason is that memory reclocking on Fermi requires executing a proprietary
PMU microcode script through the Falcon microcontroller embedded in the GPU's
PDAEMON block. The upstream nouveau driver has partial infrastructure for this
(`ramgf100.c`, `pmu/memx.c`), but it was written for Tesla-era hardware and
contains assumptions that cause Privileged Ring faults, PMU reply timeouts, and
occasionally corrupted framebuffers on Fermi silicon.

I decided to fix this. It took longer than expected.

---

## What Actually Broke

The first surprise was the PRIVRING fault loop. Register `0x10a580` is a PMU
data lock that exists on Tesla (card type < NV_C0). On Fermi and later, the
register does not exist. Writing to it generates a Privileged Ring fault. The
upstream code wrote to it unconditionally inside a polling loop, producing
roughly 33,000 faults every five seconds and hanging the calling thread. The
fix was a simple card-type guard: `if (device->card_type < NV_C0)`. This took
three hours to find because the fault was silent in most kernel log
configurations and the symptom looked like a PMU firmware issue rather than a
register access problem.

The second surprise was the PMU reply timeout. The function `gt215_pmu_send()`
used an unconditional `wait_event()` to block until the PMU Falcon responded to
a message. If the PMU encountered a delay, a missing VBlank interrupt, or
simply decided not to respond, the calling thread entered uninterruptible sleep
(D-state) forever. The fix replaced `wait_event()` with active polling and a
strict 100ms timeout window. This is the kind of fix that looks obvious in
retrospect but required tracing through four layers of PMU abstraction to
reach.

The third surprise was the GDDR5 training guard. The function
`gf100_ram_calc()` unconditionally invoked GDDR5 hardware training routines.
On DDR3 hardware, these routines do not exist. The Falcon engine hung. The fix
was a runtime check: skip GDDR5 training when `ram->base.type ==
NVKM_RAM_TYPE_DDR3`. Three lines of code, discovered after two days of
debugging what appeared to be a memory controller initialization failure.

The fourth surprise was display hub clock locking. The function
`gf100_clk_calc()` recalculated and reprogrammed the display crossbar and hub
clocks (`hubk07`, `hubk06`, `hubk01`) on every performance state change. On
Fermi laptops with high pixel-clock eDP panels (396.36 MHz for a 1920x1080 @
120 Hz display), reprogramming the hub caused FIFO underruns, display link
loss, or black screens during clock transitions. The fix omitted these clock
domains from dynamic frequency transitions. The display hub remains locked to
its stable boot frequency while core, shader, and ROP clocks scale freely.

The fifth surprise was voltage. The VBIOS VMAP speedo formula mapped the
`0f` pstate voltage ID to 862.5 mV (VID `0x04` = 870 mV) instead of the
factory 1.030 V (VID `0x01`). Under 3D load, the GPU was undervolted by 160
mV. The fix forced `cstate->voltage = 0x67` (1030000 microvolts) for the
`0f` pstate in `nvkm_pstate_new()`. Without this correction, the overclock
pstate was unstable under sustained load.

These five fixes got core and shader reclocking working. Memory reclocking
was still broken.

---

## The DDR3 Memory Reclocking Nightmare

This is where the project spent most of its time.

The proprietary NVIDIA 390.157 driver reclocks DDR3 memory through the PMU
Falcon microcontroller. It streams a bytecoded script through the
`PDAEMON.DATA` register (`0x10a1c4`), and the Falcon executes it autonomously.
The script contains register writes, timing delays, VBlank waits, and memory
controller training sequences. The upstream nouveau driver has infrastructure
for this in `pmu/memx.c`, but it was never tested on Fermi DDR3 because nobody
had the hardware or the motivation to make it work.

I had the hardware. The motivation came later.

### First Attempt: Direct MMIO Replay

The initial approach was to bypass the PMU entirely and replay the decoded
MMIO script directly from the host CPU. The proprietary driver's script was
decoded using `demmt` (part of envytools), producing two sequences: one for
324 to 900 MHz (72 register writes) and one for 900 to 324 MHz (43 register
writes).

The first test was a 324 to 324 MHz "no-op" transition to verify the script
infrastructure worked. It did not. The no-op transition executed the full
script and killed the framebuffer. PAGE_NOT_PRESENT faults on every memory
channel, PRIVRING fault at `0x13b0d4`. The system became unresponsive.

The root cause was that `gf100_ram_calc()` was Ben Skeggs' GDDR5 sequence
for the GF100 reference board. Its mode register writes (`0x10f300 =
0x0000011d / 0x0000084d`) are GDDR5 MRS encodings. Splicing DDR3 timing
values into a GDDR5 sequence does not make it a DDR3 sequence.

### Second Attempt: Correct Script, Incomplete Engine Gating

With the correct DDR3 script (decoded from the 390.157 driver trace), the
324 to 324 no-op was clean. The 324 to 900 transition executed, but PGRAPH
immediately faulted (PAGE_NOT_PRESENT on the compositor's channel), BAR
flushes started timing out, and `nvkm_intr` read `0xffffffff` from
`PMC_BOOT_0`. The memory controller had hung.

The diagnosis: disabling host IRQs stops the CPU, but PGRAPH and PFIFO kept
issuing VRAM traffic while DRAM was in self-refresh. The proprietary driver
avoids this with an FB_PAUSE handshake through the PDAEMON I/O window. The
host-side replay had no mechanism to assert FB_PAUSE because the PDAEMON I/O
block was inaccessible.

### Third Attempt: PDAEMON I/O Block Gating

This led to the central discovery: the PDAEMON I/O block (`0x10a4xx` queues,
`0x10a6xx` MEMIF, `0x10a7xx` GPIO) is gated after Falcon reset. The
proprietary driver unlocks it by performing a specific register sequence before
uploading firmware:

```
W 0x10a048 0x00000001   PDAEMON.ACCESS_EN.CHANNEL_SWITCH
W 0x10a090 0x00010040   PDAEMON.UNK090 bit16
W 0x10a47c 0x701074ef   PDAEMON.CHANNEL_SETUP (VALID | dummy channel)
W 0x10a058 0x00000002   PDAEMON.CHANNEL_TRIGGER.LOAD
```

Without this sequence, the `0x10a4xx` registers fault with PRIVRING. With
it, the registers respond normally. The upstream `gt215_pmu_init()` does
none of this on Fermi because on Kepler the block is open by default. The
fix replicated the unlock sequence in `gt215_pmu_init()` for `NV_C0` after
the scrub wait.

With the unlock in place, the PMU firmware demonstrably boots. The `H2D` and
`D2H` registers read back `0x00800270` and `0x008002f0`, which are the
`fifo_queue` and `rfifo_queue` DMEM link addresses from `host.fuc`. Only
firmware that executed `host_init` could program those values. The firmware
upload works. The Falcon boots.

But the first real message fails. `pmu reply timeout`. Two "unexpected
message" warnings consumed from stale RFIFO slots. The Falcon is found halted
(`UC_CTRL = 0x10`, STOPPED), all queue registers read zero.

### The Falcon Core Side

The selective fault pattern told the story: host reads of `FIFO_PUT` and
`RFIFO_GET` fault while writes, `GET`, `RPUT`, and DATA-window accesses pass.
The boot-time unlock opened the host side. The Falcon-core side needed a valid
loaded channel. Without it, the firmware boots (writes only), idles healthy
(never polls PUT; five-minute idle watch is clean), and halts on its first
PUT poll when the first host message wakes it.

This single mechanism explains idle health, first-message death, the silent
post-mortem (halt, not a host-visible violation), and why no host fault lines
appear at INFO time.

### The Firmware Patch

The fix required patching the Falcon firmware binary itself. Disassembly of
the `gf100` PMU image (`envydis -m falcon -V fuc3`) located the
`host_recv_wait` loop at code offset `0x559`: a triple-instruction sequence
polling `RGET`. The I/O gating also blocked the Falcon core's read of `RGET`,
so the firmware consumed the first message (`GET` advances), then halted
inside `host_recv_wait` before writing any reply.

The 29-byte wait was skipped with `bra +29` patched over its first instruction
(`f1 17 cc 04` replaced with `f4 0e 1d`). Same length, no address shifts, all
header labels remain valid. The `RPUT` bump and `INTR_TRIGGER` MMIO writes
were NOP-filled with `clear b32 $r0` (`bd 04`). The driver-side reply wait
was updated to poll all 8 reply slots directly and consume via an `RGET` write.

After this patch, the firmware boots, idles, consumes messages, and produces
replies. Memory reclocking through the normal MEMX path works.

### The Custom Falcon Executor

As a fallback and for future experimentation, a custom Falcon microcode
executor was also built. Instead of talking to the proprietary PMU firmware
through message queues, the host uploads a tiny custom Falcon program into
PDAEMON IMEM, points `ENTRY` at it, and starts execution. The custom program
walks a decoded DDR3 script from a DMEM scratch area, implementing the same
seven MEMX opcodes (`ENTER`/`LEAVE`/`WR32`/`WAIT`/`DELAY`/`VBLANK`/`TRAIN`)
so existing scripts run unmodified.

The executor uses a resident command-wait architecture: instead of halting
after script completion, it spins polling a scratch register for the next
command. This avoids the hardware power-management clock-gating that engages
when the Falcon transitions from RUNNING to STOPPED, which on Fermi triggers
fatal PRIVRING faults on any subsequent MMIO access. The transition latency
drops to approximately 5ms with zero reset overhead.

---

## The Dynamic Clock Governor

With reclocking working, the next problem was making it automatic. A static
pstate selection is useful for benchmarks but not for daily use. The
`nouveau-dynclockd` daemon implements a two-stage dynamic frequency governor:

**Stage 1 (2D/Desktop):** Smoothly toggles between `03` (50 MHz) and `07`
(202 MHz) based on GPU load, keeping memory clamped at 324 MHz. This is
100% flicker-free at 120 Hz because the memory clock does not change and the
display hub is not reprogrammed.

**Stage 2 (3D Graphics):** Instantly scales to `0f` (590 MHz core, 1180 MHz
shader, 900 MHz memory at 1.030 V) when dedicated 3D applications are
detected. The governor monitors GPU temperature and caps the highest clocks to
`07` when the throttle limit is reached (default 80 degrees C with 5 degrees
C hysteresis).

The governor communicates with the driver through sysfs and uses Wayland/EGL
load awareness for compositor detection. It is written in Python for
maintainability and ships as a systemd service.

---

## The Tooling

The repository includes several diagnostic tools that were built during the
reverse engineering process and are now available for hardware telemetry
collection:

**`nouveau-fermi-diag.py`**: Standalone hardware diagnostic tool that reads
debugfs and VBIOS nodes, producing a structured report with GPU information,
pstate tables, memory configuration, and thermal zone data. This is the
primary tool for community hardware telemetry contributions.

**`nouveau-ctrl`**: CLI management utility for status reporting, clock
locking, and governor control. Colorized thermal zones (<65 degrees C green,
65-75 degrees C yellow, >75 degrees C red). Real-time fan RPM reading via Dell
SMM platform monitor. PCIe link speed and generation telemetry.

**`nouveau-tui`**: Interactive Curses-based terminal user interface for GPU
reclocking and telemetry. Same data as `nouveau-ctrl` but with a live
updating display.

**`sniff-memory-reclock.py`**: Direct hardware BAR0 memory controller (PFB)
and PLL timing sniffer for verifying memory clock transitions.

**`run-mmiotrace.sh`**: Automated kernel MMIO tracing and `demmt` decoder
script for capturing proprietary driver behavior on different hardware.

---

## Call for Hardware Telemetry

Here is the part that matters for the project's future.

Everything described above was developed and tested on a single hardware
configuration: a Dell XPS L702X with a GeForce GT 555M (GF106M, 3072 MB
DDR3, 1920x1080 @ 120 Hz). The Fermi family spans GF100 through GF119, with
dozens of OEM variants using different VBIOS power tables, voltage regulators,
display connectors (eDP, LVDS, HDMI, DisplayPort), and memory types (DDR3,
GDDR5).

The DDR3 reclocking scripts are board-specific. The timing values, MPLL
dividers, and mode register sequences are decoded from the proprietary
driver's MMIO trace on this specific hardware. They may work on other DDR3
boards. They may not. The only way to know is to collect traces from more
hardware.

If you have a Fermi GPU (any model from GF100 to GF119) and are willing to
run diagnostics, I need your help. The process is:

1. Clone the repository:
   ```bash
   git clone https://github.com/Twilight0/nouveau-fermi-reclock-dkms.git
   cd nouveau-fermi-reclock-dkms
   ```

2. Run the diagnostic tool:
   ```bash
   sudo python3 tools/nouveau-fermi-diag.py
   ```

3. Submit the generated `nouveau_fermi_diag_report.md` via the
   [Hardware Telemetry Issue Template](https://github.com/Twilight0/nouveau-fermi-reclock-dkms/issues/new?template=hardware-telemetry.md).

For deeper involvement, the `run-mmiotrace.sh` script captures proprietary
driver behavior through kernel MMIO tracing. This produces the decoded
register sequences needed to extend reclocking support to GDDR5 hardware and
to other Fermi GPU variants. The trace capture process requires the
proprietary NVIDIA driver (390.157 or earlier) to be installed and functional
on the target hardware.

See [`CONTRIBUTING.md`](https://github.com/Twilight0/nouveau-fermi-reclock-dkms/blob/master/CONTRIBUTING.md)
for detailed instructions.

---

## Why This Matters

Hardware is expensive. AI has made it more so. The users who most need
open-source GPU driver support are the ones who cannot afford to replace their
hardware every two generations. Fermi GPUs are still in active use on laptops
and desktops that are otherwise perfectly functional. A 2.2x memory bandwidth
improvement through proper reclocking is the difference between a usable
desktop and a frustrating one.

The nouveau driver has the infrastructure. What it lacks is the hardware-specific
data and the testing coverage to make reclocking safe across the full range of
Fermi hardware. That data comes from mmiotraces. Those traces come from users.

The project repository is at
[github.com/Twilight0/nouveau-fermi-reclock-dkms](https://github.com/Twilight0/nouveau-fermi-reclock-dkms).
The AUR package is `nouveau-fermi-reclock-dkms`. Documentation is in
[`RECLOCKING_NOTES.md`](https://github.com/Twilight0/nouveau-fermi-reclock-dkms/blob/master/RECLOCKING_NOTES.md)
(everything above, with full register tables and disassembly) and
[`NVENC.md`](https://github.com/Twilight0/nouveau-fermi-reclock-dkms/blob/master/NVENC.md)
(hardware video encoding analysis).

Contributions are welcome. Hardware telemetry is critical. If you have Fermi
hardware and a few minutes to run a diagnostic script, the project needs your
data.

---

## See Also

- [Xlibre, Nouveau, and Telegram Bans](/news/2026-09-15-xlibre-nouveau-and-telegram-bans/) &mdash;
  The broader context for this work: why a Wayland-compatible, open-source
  driver matters for legacy hardware, and why making it work properly is worth
  the effort.
