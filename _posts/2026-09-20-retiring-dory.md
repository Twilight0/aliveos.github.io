---
layout: post
title: "Why We Are Retiring Dory: A File Manager That Became a Portal Backend"
date: 2026-09-20 14:00:00 +0000
author: The AliveOS Project
---

Six weeks ago, we wrote about [Dory](/news/2026-08-03-dory-file-manager-deep-dive/) as
the cornerstone of AliveOS &mdash; a standalone file chooser portal backend
forked from Nemo, purpose-built for sandboxed application integration. Today
we are retiring it. Not because it failed, but because it succeeded at the
wrong thing.

This post explains the technical reasoning behind the decision, what we
learned about portal architecture, and where the work ended up.

---

## The Original Thesis

Dory started with a clean idea: fork Nemo, rename every namespace, and ship a
standalone file chooser portal that could install side-by-side with the
system file manager without conflicts. The file manager and the portal would
be the same binary, serving both roles through D-Bus activation.

The thesis was that a dedicated portal backend &mdash; one that did exactly
what AliveOS needed and nothing else &mdash; would be simpler to maintain than
configuring the generic `xdg-desktop-portal-gtk` or fighting with Nemo's
built-in portal mode.

For a while, this held up.

---

## Where It Broke Down

### The Rebase Tax

Nemo is actively maintained. The Cinnamon team ships point releases with bug
fixes, security patches, and occasional API changes. Every Nemo release
potentially affected Dory, because Dory inherited the full Nemo codebase and
selectively disabled components it did not need.

In practice, this meant a rebase cycle every two to four weeks. Each rebase
required:

1. Merging upstream Nemo changes into the Dory fork
2. Re-applying Dory-specific patches (namespace renames, portal D-Bus
   interface, GApplication lifecycle fixes)
3. Testing that the portal backend still worked correctly under Flatpak file
   chooser requests
4. Verifying that the file manager mode had not regressed

The rebase was not conceptually hard, but it was constant. And each cycle
carried risk: a subtle upstream change in Nemo's selection model or D-Bus
interface handling could break Dory's portal behavior in ways that only
manifested under specific application conditions (multiselect in GIMP, save
overwrites in VS Code, directory selection in Chromium).

This is the maintenance tax of forking a large, actively maintained codebase
for a narrow use case.

### The Indirection Problem

The second issue was architectural. Most applications do not call the file
manager directly. They call `org.freedesktop.portal.FileChooser` through
D-Bus, which routes to whatever portal backend is registered for the
desktop. The file manager itself is irrelevant &mdash; only the portal
interface matters.

Dory was a full file manager that happened to also be a portal backend. But
nobody was using the file manager part. They were using the portal. The file
manager code &mdash; the icon view, the list view, the sidebar, the
extension framework, the thumbnail generation &mdash; was dead weight from
the portal's perspective.

We were maintaining a complete file manager fork to ship a single D-Bus
interface with three methods (`OpenFile`, `SaveFile`, `SaveFiles`).

### The Portal Is Simpler

The third realization was that modifying the desktop portal directly was
easier than maintaining a fork of the file manager that hosted it.

`xdg-desktop-portal-xapp-filepicker` (the upstream portal backend for
XApp-based desktops) is a much smaller codebase than Nemo. It implements the
same `org.freedesktop.portal.FileChooser` interface, but without the file
manager baggage. It is updated less frequently than Nemo, which means fewer
rebases. It is designed to be a portal backend, not a file manager that
doubles as one.

The changes we made to Dory &mdash; GApplication lifecycle fixes, dialog
focus handling, multiselect URI reconstruction, overwrite confirmation &mdash;
are all portal-specific concerns. They apply equally to the standalone portal
backend. There is no reason to carry a full file manager fork to deliver them.

---

## What Happened to the Code

All of Dory's portal-specific improvements have been ported to
[xdg-desktop-portal-aliveos](https://github.com/Twilight0/xdg-desktop-portal-aliveos),
a dedicated portal backend for AliveOS. This repository contains the
accumulated fixes from the Dory experiment:

- **GApplication hold management** &mdash; the process stays alive throughout
  the dialog lifecycle, not just during the D-Bus method dispatch
- **Dialog focus forcing** &mdash; `gtk_window_present()` with
  `gdk_window_raise()` and `gdk_window_focus()` timed after realization
- **Multiselect URI reconstruction** &mdash; rebuilding the full URI list
  from all selected paths before the dialog closes
- **Save dialog overwrite confirmation** &mdash; intercepting selection of
  existing files and showing a confirmation modal
- **Non-directory save prevention** &mdash; blocking save operations when the
  target is a directory

The file manager component of Dory is being retired. The repository will
remain archived for reference, but no further development is planned.

---

## Why This Was the Right Call

Dory was never released as a stable product. It was a research project &mdash;
a proof of concept for a portable, standalone file chooser portal that could
work across desktop environments. The idea was sound: decouple the portal
from the file manager, make it installable anywhere, and let it handle file
dialogs for any sandboxed application regardless of the underlying desktop.

The execution revealed the problem: a portal backend is not a portable
solution in the way we originally envisioned. Portal backends are desktop-
specific by design. The XApp portal backend works for Cinnamon, MATE, and
XFCE because they share the same toolkit assumptions. A truly portable portal
would need to abstract away those assumptions, which means reimplementing the
dialog layer from scratch &mdash; at which point you are building a new
toolkit, not forking an existing file manager.

The decision to retire Dory was not taken lightly. It was the result of
recognizing that the maintenance burden, the architectural indirection, and
the original portability thesis did not align with the practical reality of
how portal backends work.

---

## Moving Forward

AliveOS will use `xdg-desktop-portal-aliveos` as its default file chooser
portal backend. It provides the same D-Bus interface that Dory implemented,
with all the same fixes, but without the file manager overhead.

The file manager slot in the desktop will be filled by the system default
(Cinnamon's Nemo), configured to work alongside the portal without conflicts.
Users who want a lighter file manager can install one &mdash; the portal does
not care which file manager is installed, only that the D-Bus interface is
correct.

This is the pragmatic outcome. Dory taught us what a portal backend needs to
do. Now we ship the portal backend without the file manager attached to it.

---

## See Also

- [Dory: Why We Forked Nemo and What It Took to Build a Proper File Chooser Portal](/news/2026-08-03-dory-file-manager-deep-dive/) &mdash;
  The original technical deep-dive into Dory's architecture and the problems
  it solved.
- [xdg-desktop-portal-aliveos](https://github.com/Twilight0/xdg-desktop-portal-aliveos) &mdash;
  The replacement portal backend containing all of Dory's portal-specific
  improvements.
