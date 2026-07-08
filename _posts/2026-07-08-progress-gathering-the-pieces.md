---
layout: post
title: "Progress Update: Gathering the Pieces"
date: 2026-07-08 10:00:00 +0000
author: The AliveOS Project
---

There is a particular stage in any project where you are not quite building the
thing yet, but you are definitely *doing things*. The pieces are scattered
across the table, some of them work, some of them are held together with duct
tape and optimism, and you are fairly certain that last commit was a mistake but
it is 2 a.m. and you are not undoing it now. That is where AliveOS is at the
moment, and honestly, it is a rather good place to be.

Here is what has been happening.

---

## Dory File Manager: Still Getting Sharper

![Dory](/assets/icons/dory.svg)
{: .float-right }

Dory, the file manager, continues to mature. It was one of the first pieces
built specifically for this project, and it remains the one I reach for most
often. The core is solid, the companion extensions are filling in nicely, and
the occasional bug fix lands when something behaves in a way that is
*technically correct* but *practically annoying*.

The philosophy behind Dory has not changed: do the file management things well,
skip the things nobody actually uses, and do not ship a 400-megabyte dependency
tree to open a folder. If it sounds simple, that is because it is. Getting it
right, however, is a different conversation entirely.

---

## Valuate: Because Numbers Need a Home Too

![Valuate](/assets/icons/valuate.svg)
{: .float-right }

Meet **Valuate** &mdash; the new calculator app for AliveOS. It started as a
"wouldn't it be nice" and ended up as a "why does this not exist yet."

Most Linux calculator apps fall into one of two categories: the one that comes
with your desktop environment and does barely enough, or the one that tries to
be a full-blown computer algebra system and weighs about as much. Valuate sits
squarely in the middle: a clean, fast, GTK3-native calculator that does the
math you actually need without making you feel like you need a PhD to use it.

It is early days, but it works, and it does not pull in half of GNOME to do it.
More on this one soon.

---

## A Custom Cinnamon Session

![Cinnamon](/assets/icons/cinnamon.svg)
{: .float-right }

The desktop is getting its own session. Not a full fork of Cinnamon &mdash;
let us not be dramatic &mdash; but a custom session that wraps the pieces
AliveOS cares about into a coherent, predictable experience.

This means a tailored default configuration, sensible keybindings that do not
require a cheat sheet, and a panel layout that stays out of your way while
remaining exactly where you expect it. Think of it as Cinnamon with the rough
edges sanded off and the bits you never use quietly removed. The kind of desktop
that disappears while you work, which is the highest compliment a desktop can
receive.

---

## The Website

![Developer](/assets/icons/developer.svg)
{: .float-right }

You are reading this, so clearly the website works. `aliveos.org` is live,
served over HTTPS, with a landing page that lays out the manifesto and a news
section where these updates go. It is built with Jekyll, deployed via GitHub
Pages, and costs exactly nothing to run. Which is how a website should be.

The RSS feed is there for the three people who still use RSS feeds (you have
excellent taste, by the way). The contact address for sponsorship and business
inquiries is in the footer. The whole thing is clean, fast, and does not phone
home to seventeen analytics trackers. Because it is a website, not a
surveillance apparatus.

---

## Bug Fixes and Testing: The Glamorous Part

Let us be honest about something: most of the work that happens at this stage
is not the kind that makes for exciting blog posts. It is the kind where you
spend three hours tracking down why a particular font renders strangely on
one specific monitor, or why the file picker opens two pixels to the left of
where it should, or why a certain package conflicts with another package in a
way that only manifests on Tuesdays.

This is the unglamorous grind that separates a distribution that *works* from a
distribution that *feels right*. AliveOS is firmly in the "feels right" camp,
and that means a lot of small, invisible fixes that nobody will ever notice
because they will just work. Which is the point.

---

## XConnect: Your Phone, Actually Connected

![XConnect](/assets/icons/xconnect.svg)
{: .float-right }

This is the big one. **XConnect** is AliveOS's answer to Android desktop
connectivity &mdash; a full-featured suite that covers almost everything
[KDE Connect](https://kdeconnect.kde.org/) does, but built on **XApp/GTK3**
to fit naturally into the AliveOS desktop.

What does it do? The usual useful things:

- **Notification syncing** &mdash; your phone's notifications, on your desktop,
  where you can actually read them without picking up the device.
- **File transfer** &mdash; drag, drop, done. No cloud services, no email
  yourself the file like it is 2009.
- **Clipboard sharing** &mdash; copy on the phone, paste on the desktop. Or the
  other way around. The future is bidirectional.
- **Media control** &mdash; pause, play, skip from your desktop while your
  phone streams music. Because reaching for your phone to skip a track is a
  civilisation-level inconvenience.
- **SMS messaging** &mdash; read and reply to texts from the desktop. Yes, it
  still works, and no, it does not require sending your messages through a
  third-party server.
- **Remote input** &mdash; use your phone as a touchpad or keyboard. Handy
  for presentations, couch computing, or when the cat is sitting on your
  keyboard and you cannot be bothered to move her.

The key difference from KDE Connect is the toolkit dependency. XConnect depends
on **XApp and GTK3**, which means it integrates cleanly with the Cinnamon
desktop and the broader XApp ecosystem without dragging in Qt libraries or
additional framework dependencies. It is native, it is light, and it belongs
here.

This one is still under active development, but it is far enough along to talk
about. Expect a dedicated post when the feature set stabilises.

---

## The Custom Repository

![Repos](/assets/icons/repos.svg)
{: .float-right }

None of this matters if you cannot actually install it. The **aliveos-repo**
custom pacman repository is the plumbing that ties everything together. It
compiles and hosts stable builds of AUR and local packages, and it does so
automatically &mdash; a GitHub Actions pipeline pulls, normalises, compiles,
assembles, and deploys on a weekly schedule.

The repository already covers Dory and its extensions, the Cinnamon
repackaging, the XLibre display server stack, AliveOS core packages, themes,
and a growing collection of utilities. If you are running Arch, you can point
pacman at it right now. If you are running AliveOS, it is already there.

---

## Where Things Stand

The pieces are gathering. Some are polished, some are rough, some are still
being hammered into shape on a Friday evening with questionable coffee. The
target for `v0.1` remains **Q3 2026**, and the path to get there is clearer
every week.

The important thing is that every piece here exists for a reason. Not because
it was popular, not because someone else shipped it, but because it solves a
specific problem in a way that fits the rest of the system. That is what a
curated distribution looks like. Not a bigger pile of packages &mdash; a
*coherent* one.

More updates as pieces land.

> Zero Bloat Policy reminder: every component mentioned above earned its place.
> Nothing is included because "someone might want it." It is included because
> it works, it fits, and it is needed. If that changes, it goes.
