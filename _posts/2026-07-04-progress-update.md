---
layout: post
title: "Progress Update: Foundations Taking Shape"
date: 2026-07-04 07:00:00 +0000
author: The AliveOS Project
---

A couple of days on from our welcome post, and things are moving. Here is a
quick look at what has come together since launch and what is currently in
flight.

## Online and reachable

The project now has a proper home at **[aliveos.org](https://aliveos.org)**,
served over HTTPS with a valid certificate on a custom domain. The landing page
lays out the manifesto and defaults; this news section is where we will publish
ongoing progress.

A couple of infrastructure bits worth noting:

- **An RSS feed** is available at [/feed.xml](/feed.xml) for anyone who wants
  updates delivered to their reader.
- **A contact address** for sponsorship and business inquiries is now live
  (you will find it in the footer of every page).

## Foundations taking shape

Behind the scenes, the project's repositories and build infrastructure are
coming together. The package repository, the icon theme, the Cinnamon
customizations, and the taskbar applet are all under active development. These
are the unglamorous pieces that make a distribution feel coherent on first
boot, and they are where most of the current effort is going.

Native utilities are progressing in parallel:

- **Dory File Manager** &mdash; the core and a set of companion integrations
  are in active development.
- **Respite Media Player** &mdash; streamlined playback, free of heavy bloat.
- **Custom Cinnamon DE** &mdash; a refined, responsive desktop workflow.

## The `aliveos-repo` package repository

One piece worth calling out specifically: [aliveos-repo](https://github.com/Twilight0/aliveos-repo),
our custom pacman repository. It compiles and hosts stable builds of AUR and
local packages for AliveOS, and it does so automatically &mdash; a GitHub
Actions pipeline runs on a weekly schedule inside a privileged Arch Linux
container, pulls packages from the AUR, normalizes them with a small
`clean_pkgbuild.py` helper, compiles them with `makepkg`, assembles the
database with `repo-add`, and deploys the resulting `*.pkg.tar.zst` files to a
`gh-pages` branch so they are instantly downloadable.

The repository already covers a fair amount of ground:

- **Dory** &mdash; the file manager itself, plus a full set of extensions
  (`dory-audio-tab`, `dory-compare`, `dory-dropbox`, `dory-emblems`,
  `dory-fileroller`, `dory-image-converter`, `dory-media-columns`,
  `dory-pastebin`, `dory-preview`, `dory-python`, `dory-repairer`,
  `dory-seahorse`, `dory-share`, `dory-terminal`).
- **cinnamon-no-nemo** &mdash; Cinnamon repackaged without the Nemo dependency,
  leaning on Dory for file handling instead.
- **XLibre stack** &mdash; `xlibre-xserver-legacyabi`, a drop-in replacement
  for the X11 display server, alongside patched legacy NVIDIA drivers
  (`nvidia-390xx-utils`, `nvidia-340xx-utils`) tuned for modern kernels and
  XLibre.
- **AliveOS core** &mdash; `aliveos-assets`, `aliveos-settings`, and
  `aliveos-cinnamon-spices`.
- **Themes &amp; utilities** &mdash; `graphite-gtk-theme-git` (including the
  black compact variant), `tela-icon-theme`, `viewmd` (a lightweight GTK
  markdown viewer), `nerd-dictation` (voice typing via Vosk),
  `xdg-desktop-portal-xapp-filepicker`, `grub-silent-ldfix`, and the legacy
  Clutter stack that backs `dory-preview`.

If you are running Arch (or an early AliveOS build), you can point pacman at the
repository right now by adding this to `/etc/pacman.conf`:

```ini
[aliveos-repo]
SigLevel = Optional TrustAll
Server = https://Twilight0.github.io/aliveos-repo/x86_64
```

Then `sudo pacman -Syu` to sync.

## What is next

The target for the initial `v0.1` release remains **Q3 2026**. Between now and
then, expect sporadic updates here as individual components reach a usable
state, the package set stabilizes, and the installer image starts coming
together.

> Zero Bloat Policy: strictly essential utilities, no office suites, no
> pre-installed bloatware. Clean by default.

Thanks for following along &mdash; the next update will be more concrete as
pieces start landing.
