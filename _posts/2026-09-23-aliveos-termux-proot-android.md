---
permalink: /news/aliveos-on-android-termux-proot/
layout: post
title: "AliveOS on Android: Termux, PRoot, and a Phone That Thinks It's a Workstation"
date: 2026-09-23 18:00:00 +0000
author: The AliveOS Project
---

Let me tell you a story about the fastest computer most people own, and how
they never get to actually use it as one.

Your phone has 8 cores, 8 or 12 GB of RAM, NVMe storage faster than the
spinning rust most desktops still limped along on ten years ago, an always-on
internet connection, and a battery. Then Android locks it all behind a Java
sandbox, a touch launcher, and a share sheet, and tells you: this is not a
computer. You may consume. You may not compile.

I find that offensive. So we did what we always do in AliveOS: strip the idea
down to the bone and see if it survives.

The question was simple: can the AliveOS paradigm &mdash; zero bloat,
terminal-first, developer-optimized, no-nonsense tooling &mdash; live inside
Android, via Termux and PRoot? The short answer is yes. The long answer is
yes, but you have to cheat, lie to binaries about what kernel they are on,
and drag glibc kicking and screaming into a Bionic world.

This post is the long answer. Until the full proot image is ready, we have
put up a staging ground: a custom Termux APT repo with the tools we actually
use every day.

Get it here: [github.com/Twilight0/termux-repo](https://github.com/Twilight0/termux-repo/)

```bash
curl -sL https://twilight0.github.io/termux-repo/install.sh | bash
```

---

## What Termux actually is (and isn't)

Termux is not a Linux distribution. It is a very clever prefix:
`/data/data/com.termux/files/usr`. No root, no systemd, no `/sbin/init`,
SELinux watching your every move, and an Android kernel built for phones,
not servers.

Most importantly: Android uses Bionic libc, not glibc. That single sentence
explains about 80% of the pain you are about to experience. If a binary was
compiled against glibc &mdash; which is to say, if it was compiled like a
normal Linux program &mdash; it will look around Termux, fail to find
`/lib64/ld-linux-x86-64.so.2`, shrug, and die with a confusing `No such file
or directory` even when the file is right there staring at you.

Termux works around this by rebuilding the entire world against Bionic.
Which is heroic, and also why half the cool new developer tools never land
in `pkg`. Upstream does not package them, the Termux maintainers do not have
infinite time, and you are left compiling Node-based CLIs on a phone at
2 AM wondering where your life went wrong.

PRoot is the second cheat code. Short version: user-space `chroot` using
`ptrace`. It intercepts syscalls and pretends the guest filesystem is `/`.
No root required. With `proot-distro` you can run Arch, Debian, Fedora inside
Termux like a Russian doll:

```bash
pkg install proot-distro
proot-distro install arch
proot-distro login arch
```

It works. It is also slow in that special emulated way, breaks on weird
syscalls, has no systemd, no real `/proc`, no cgroups, no BTRFS snapshots,
no GPU acceleration, and thermal throttles exactly when you get excited.
In other words: perfect AliveOS material. Constraints breed discipline.

---

## Why even bother? The hardware argument, again

We keep repeating this because it keeps being true: hardware is expensive,
AI made it worse, and the best computer in most households is the phone
nobody thinks of as a computer.

An old Pixel, a retired Samsung tablet, a Xiaomi with a cracked corner but
perfect silicon &mdash; these are 8-core ARM64 machines with more RAM than
the laptops we used to ship distributions on. With a Bluetooth keyboard,
a cheap USB-C hub, and Termux-X11, they become a surprisingly honest
workstation: terminal, editor, git, ssh, AI agents, browser via proot.

You will not run Cinnamon on it. You will not run our full desktop stack.
Forget BTRFS snapshots, forget display servers, forget daemons. What does
translate is the philosophy:

1. **Zero bloat.** On a phone, every megabyte and every wake-lock matters.
2. **Curated repos over sprawl.** One small repo you trust beats three big
   ones you don't.
3. **Terminal-first workflow.** Touch is for scrolling. Real work is
   keyboard-driven, even over SSH / DeX / scrcpy.
4. **Portable tooling.** If it can't run in a prefix without systemd, it
   doesn't belong here.

That last one is why the Dory lesson mattered. We killed a whole file
manager because the portal was the real interface. Same logic on Android:
don't port the desktop, port the workflow. See
[Dory retirement](/news/retiring-dory/) if you missed that bloodbath.

---

## The glibc swamp: empirical notes from the trenches

Here is where the fun starts. Modern AI coding tools love to ship as
single-file Bun / Node / Rust binaries linked against glibc. Try running
one in vanilla Termux and you get:

- `CANNOT LINK EXECUTABLE: cannot locate symbol`
- `SIGSYS (bad system call)` from seccomp filters
- mysterious deaths inside `workerd` / V8 isolates
- `VA39` address-space complaints on older ARM kernels

Our [PACKAGE_RESEARCH.md](https://github.com/Twilight0/termux-repo/blob/main/PACKAGE_RESEARCH.md)
is the field diary. Highlights, so you don't have to bleed the same way:

- **Bun + seccomp:** Bun loves syscalls Android's seccomp profile hates.
  Fix is PRoot syscall emulation or patched builds, not prayers.
- **`workerd` stubbing:** Cloudflare's `workerd` runtime assumes a real
  server kernel. On Termux there is no local `wrangler dev`. We ship
  `wrangler` as an npm wrapper, cloud-API only. `deploy` works,
  `dev` doesn't. Documented, not hidden.
- **VA39 patching:** some prebuilts assume 39-bit virtual addressing.
  Older phone kernels say no. You either patch, wrap in `qemu-user`, or
  pick another binary.
- **glibc via `glibc-repo`:** the community `x11-repo` / `tur-repo` /
  `glibc-repo` dance (`pkg install glibc-repo`) gives you a glibc prefix
  inside Termux. It works, but now you have two libcs eyeing each other
  suspiciously. Version skew breaks things monthly.

None of this is elegant. All of it is real. If you want elegance, buy a
ThinkPad. If you want a computer in your pocket, keep reading.

---

## The repo: what we ship today

The [termux-repo](https://github.com/Twilight0/termux-repo/) is intentionally
small. No 40,000-package mirror. Just the stuff we needed and couldn't
`pkg install`:

| Package | What it is | Arch | Gotcha |
|---------|------------|------|--------|
| `wrangler` | Cloudflare Workers CLI | all | npm wrapper, no local `dev`, use `deploy` |
| `9router` | Free AI coding router with smart fallback | all | pure script, just works |
| `muse-code` | Meta's Muse Code agent, sessions/MCP | aarch64, x86_64 | needs `proot` for syscall emulation |
| `opencode` | AI coding assistant | aarch64, x86_64 | needs `glibc-repo` + `glibc` |
| `antigravity-cli` | Google Antigravity CLI | aarch64, x86_64 | needs `glibc-repo` + `glibc` |
| `oh-my-pi` | Oh-My-Pi plugin manager | aarch64, x86_64 | needs `glibc-repo` + `glibc` |

Install flow that actually works on a fresh Termux (Android 11+, ~2 GB free):

```bash
pkg update && pkg upgrade -y
pkg install -y git curl proot glibc-repo
pkg install -y glibc  # for opencode / antigravity-cli / oh-my-pi

curl -sL https://twilight0.github.io/termux-repo/install.sh | bash
pkg update
pkg install -y opencode muse-code 9router
```

On x86_64 tablets without AVX2 (yes, they exist, yes, they hurt), add
`qemu-user-x86-64` for `muse-code`. On aarch64 phones you can skip it.

CI rebuilds weekly via GitHub Actions and pushes to GitHub Pages. Push to
`main`, packages rebuild. No manual tarball juggling.

Uninstall, if you hate joy:

```bash
curl -sL https://twilight0.github.io/termux-repo/uninstall.sh | bash
```

There are also two guides worth stealing even if you never touch our repo:

- **DNS_FORWARDING.md:** AdGuard Home + dnsmasq on rooted / unrooted
  Android, with client config for LAN devices. Because ISP DNS is
  surveillance with extra steps.
- **PACKAGE_RESEARCH.md:** the VA39 / Bun / seccomp / workerd notes
  mentioned above. Future packagers, start here.

---

## What "AliveOS on Android" will actually look like

Let's be honest about scope. We are not stuffing Cinnamon into a phone.
Nobody wants to pinch-zoom a start menu. The AliveOS-on-Android target is
much more boring, and much more useful:

- A proot Arch image with AliveOS defaults: zsh/fish config, editor,
  git, ssh, TUI tools, AI agents pre-wired, no desktop.
- The Termux repo as the outer bootstrap layer (native-side tools).
- Termux-X11 / VNC only for the few graphicalescape hatches (browser
  testing, image preview), not as daily driver.
- Same zero-bloat policy: if it wakes the CPU in background, it gets
  removed. Battery is the ultimate bloat detector.
- Thermal-aware thinking: phones throttle. Our governor experience from
  [Fermi reclocking](/news/nouveau-fermi-reclock-deep-dive/) taught us to
  respect temperature curves instead of fighting them.

Think of it as AliveOS headless mode: the brain without the face. You ssh
in, or you dock the phone with DeX / scrcpy + Bluetooth keyboard, and you
get 90% of a dev laptop with 5% of the power draw.

Is PRoot slow? Yes, about 10-20% syscall overhead, worse for I/O-heavy
builds. Is it usable? Also yes. `git`, `python`, `node`, `rustc` on small
crates, AI agents streaming diffs &mdash; all fine. Chromium compile?
Don't. That's what the big machine is for. Know your battles.

---

## Try it, break it, tell us

This is still scaffolding, not a release. Rough edges, missing deps,
glibc skew, the usual Termux paper cuts. But the repo installs, the tools
run, and I wrote half of this post's research notes from inside Termux
itself, which is either proof of concept or Stockholm syndrome. You decide.

If you have:

- an ARM64 phone / tablet lying in a drawer,
- Termux from F-Droid (not Play Store, that build is ancient),
- 20 minutes and a charger,

then:

```bash
pkg install -y proot-distro
proot-distro install arch
curl -sL https://twilight0.github.io/termux-repo/install.sh | bash
```

Break it. File issues. Tell us your device, Android version, kernel
(`uname -a`), and what died. Especially interesting: Pixel vs Samsung
seccomp behavior, Xiaomi/MTK quirks, Android 12 vs 14 Bionic differences.

Hardware telemetry saved the Fermi reclocking project. Same deal here:
we cannot test every phone. You are the test lab.

Repo again: [Twilight0/termux-repo](https://github.com/Twilight0/termux-repo/).
Forum / Discussions for longer rants. And no, we won't make a touch-first
launcher. Touch is for scrolling. Real work is typing.

---

## See Also

- [Why We Are Retiring Dory](/news/retiring-dory/) &mdash;
  why we port workflows, not desktops.
- [Nouveau Fermi Reclocking deep-dive](/news/nouveau-fermi-reclock-deep-dive/) &mdash;
  what thermal and power constraints teach you, applicable from GPUs to phones.
- [Xlibre, Nouveau, and Telegram Bans](/news/xlibre-nouveau-and-telegram-bans/) &mdash;
  why open drivers for old hardware matter &mdash; phones included.
