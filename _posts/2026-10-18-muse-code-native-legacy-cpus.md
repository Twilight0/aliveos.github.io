---
permalink: /news/muse-code-native-legacy-cpus/
layout: post
title: "Muse Code Goes Native on Legacy CPUs: 1.4.10, Opt-in QEMU, and Our Endorsed AI Agent"
date: 2026-10-18 12:00:00 +0000
author: The AliveOS Project
---

The fastest way to make old hardware feel new is to stop emulating things
that never needed emulating.

When we first packaged Meta's Muse Code for AliveOS, upstream's x86_64
binary demanded AVX2. Anything pre-Haswell &mdash; Sandy Bridge, Ivy
Bridge, early Atoms and Celerons, the machines AliveOS exists to keep
alive &mdash; died on launch with `Illegal instruction`. So we did the
pragmatic thing: detect the missing flag in `/proc/cpuinfo` and silently
reroute through `qemu-x86_64 -cpu max`. It worked. It was also a tax on
every single invocation, on machines that could least afford it.

This week that tax got repealed. Upstream killed the AVX2 baseline, and we
rebuilt the packaging around that fact. Here is what changed, where it
comes from, and why Muse Code is now our endorsed AI coding agent.

---

## What changed

**1. Upstream 1.4.10, verified native on pre-AVX2 silicon.**
Our repo carried 1.4.2; we jumped straight to `1.4.10-R6427.1`. And we did
not take "runs everywhere" on faith: our own build machine has AVX but no
AVX2, and the new binary runs on it directly &mdash; `--version`, real
`exec` workloads, exit zero, no emulator in the process tree. The original
crash-at-startup failure mode is gone.

**2. QEMU is now opt-in, not automatic.**
The wrapper executes natively by default. Emulation is still there for edge
cases, but you have to ask for it explicitly:

```bash
MUSE_QEMU=1 muse
# or
muse --qemu
```

Why opt-in instead of auto-detect? Auto-detect was the right call when the
binary *could not run* without emulation. Now it can, and automatic
emulation would mean silently paying 2-5x startup overhead forever on
exactly the hardware that should be spared. Defaults should cost nothing;
exotic fallbacks should require a signature. If you ever do hit a real
`Illegal instruction` on ancient silicon, install `qemu-user` and opt in.
Otherwise forget QEMU exists.

**3. AUR first, AliveOS second &mdash; one pipeline.**
`muse-code-bin` on the [AUR](https://aur.archlinux.org/packages/muse-code-bin/)
is now the single source of truth: it gets the version bump and the wrapper
changes first. The [aliveos-repo](https://github.com/Twilight0/aliveos-repo)
build pulls from the AUR instead of tracking a separate upstream clone, and
CI rebuilds and republishes the `latest` release from there. One packaging,
reviewed once, shipped everywhere: AliveOS (`sudo pacman -S muse-code`
&mdash; no `-bin` suffix needed, the repo package provides that name), AUR
(`yay -S muse-code-bin`), Termux (`aarch64` deb).

A correction to our own [Android post](/news/aliveos-on-android-termux-proot/):
it told owners of x86_64 tablets without AVX2 to add `qemu-user-x86-64`.
That advice is retired &mdash; current releases run natively there too.
(`proot` on Android is still required; that is a seccomp problem, not a CPU
problem, and it is staying.)

---

## Upstream references and docs

- Muse Code documentation (the real docs, not the portal front page):
  [dev.meta.ai/docs/muse-code](https://dev.meta.ai/docs/muse-code)
- Meta's published changelog &mdash; what each release actually changed:
  [dev.meta.ai/docs/muse-code/changelog](https://dev.meta.ai/docs/muse-code/changelog)
- Meta's Muse portal: [dev.meta.ai](https://dev.meta.ai)
- Our packaging, wrappers, session manager, MCP bridge:
  [github.com/Twilight0/muse-code](https://github.com/Twilight0/muse-code)
- AUR package: [muse-code-bin](https://aur.archlinux.org/packages/muse-code-bin/)
- Prebuilt AliveOS repo: [github.com/Twilight0/aliveos-repo](https://github.com/Twilight0/aliveos-repo)

---

## Why we endorse it: cheap inference, zero-friction setup

AliveOS does not endorse tools lightly &mdash; endorsement here means "this
is the default answer when someone asks what AI agent to run." Muse Code
earned it on two axes that matter more than benchmark scores:

**Cheap inference.** The model lives behind an API, not behind a GPU you
have to buy, cool, and feed. For the old-hardware crowd this is the whole
game: a Sandy Bridge box with 8 GB of RAM cannot run a serious local model,
but it can stream diffs from Muse Spark all day. Metered inference turns AI
assistance from a hardware purchase into a utility bill &mdash; and a small
one. No VRAM, no quantized-model roulette, no watching your machine swap
itself to death.

**Easy setup.** This is where our packaging earns its keep:

```bash
yay -S muse-code-bin      # Arch / AUR
sudo pacman -S muse-code  # AliveOS repo (no -bin suffix needed)
muse "fix the failing tests"   # ...that is the whole onboarding
```

Zero configuration for the cases that used to need forum threads: Termux
`proot` wrapping on Android, TLS paths, working-directory handling. Plus
two tools upstream does not ship &mdash; `muse sessions`, a full curses TUI
for finding and resuming past work, and `muse mcp import`, which vacuums up
MCP servers from Claude Code, Codex CLI, OpenCode, and Antigravity configs
you already have. Switching agents should not mean rewiring your tools.

Terminal-first, no daemon, no Electron, no background wake-locks. It respects
the battery, which on AliveOS is a moral issue, not a technical one.

---

## Try it, on your worst machine

That is the actual test. Dig out the oldest x86_64 box you have, install
from the repo, and run `muse --version`. If it prints a version instead of
dumping core, the AVX2 era is officially behind us. If it doesn't &mdash;
file an issue with your `/proc/cpuinfo` flags and we will look at it
personally. Old hardware is not an edge case here. It is the mission.
