---
layout: post
title: "xconnect: A Deep Dive into Building a KDE Connect Alternative from Scratch"
date: 2026-07-31 14:00:00 +0000
author: The AliveOS Project
---

## Why We Built xconnect

Let's cut to the chase: KDE Connect is a fantastic project, but it comes with a dependency chain that makes it unsuitable for lightweight, GTK-based desktop environments. If you're running Cinnamon, MATE, or XFCE on Arch, you shouldn't need to pull in half of KDE's infrastructure just to get clipboard sync and notification mirroring working.

So we built **xconnect** — a clean-room implementation of the KDE Connect protocol in Vala/C with a GTK3/XApp GUI. No KDE dependencies. Just pure functionality.

## Technical Architecture

xconnect consists of three components:

1. **xconnect daemon** — The core service handling protocol negotiation, device discovery, TLS channel management, and packet routing via D-Bus (`org.xconnect`).

2. **xconnectctl** — A CLI client that exposes every daemon capability as a command-line interface. This isn't just a toy wrapper; it's a fully-featured automation tool.

3. **xconnect-app** — A GTK3 system tray application with device management, notification history, MPRIS media controls, and inline configuration editing.

The daemon runs as a systemd user unit, listening on UDP 1716 for incoming device discovery and broadcasting identity packets to UDP 1714 every 5 seconds. Device connections are established over TCP 1714, with TLS negotiation handled via GnuTLS.

## Features Implemented

Here's what's working as of v2.3.0:

| Category | Capabilities |
|----------|--------------|
| **Device Management** | Discovery, pairing (with SHA-256 verification keys), unpairing, persistent short IDs |
| **Notifications** | Full mirroring, action buttons, inline reply (Cinnamon native), dismissal |
| **File Sharing** | File, URL, and text snippet transfer via `kdeconnect.share.request` |
| **Clipboard** | Bidirectional sync via xclip watcher |
| **Remote Input** | Touchpad/mouse control (libxtst), presentation pointer |
| **Media Control** | MPRIS integration, system volume control |
| **Telephony** | SMS sending, call mute, find my phone |
| **System** | Battery telemetry, cellular connectivity reports, screensaver inhibit, device lock |

## The Nasty Bits: Roadblocks and Fixes

### Dual-Bus D-Bus Nightmare

The most insidious bug we encountered was the **dual-bus problem**. Here's what happened:

- The xconnect daemon registers on dbus-broker (`/run/user/1000/bus`) — the modern, high-performance D-Bus implementation used by systemd.
- Terminals and the GUI were connecting to the legacy dbus-daemon.
- Result: `xconnectctl list-devices` would show nothing, while the GUI worked fine.

The fix required probing the systemd user bus first, then falling back to the session bus:

```vala
// In xconnectctl — try systemd user bus first
string bus_address = Environment.get_variable("DBUS_SESSION_BUS_ADDRESS");
if (bus_address == null || !bus_address.contains("/run/user/")) {
    bus_address = "unix:path=/run/user/" + UserId.to_string() + "/bus";
}
```

We also removed the D-Bus activation file (`extra/org.xconnect.service`) from installation — it was causing the legacy dbus-daemon to auto-spawn a second daemon instance with no TCP listener. Classic.

### Packet Queuing During TLS Handshake

Devices would occasionally fail to receive packets during the initial connection phase. Root cause: the daemon was trying to send packets before the TLS handshake completed.

The solution was a **packet queue** in `Device.send()`:

```vala
public void send(Json.Object pkt) {
    if (this._channel == null || !this._is_active) {
        this._pending_packets.add(pkt);  // Queue silently
        return;
    }
    // ... send immediately
}
```

Packets queue silently when the channel isn't ready, then flush automatically after `secure_incoming_channel()` completes. No more lost packets.

### Messenger Reaction Emoji Fallback

Here's a fun one. Android's background activity launch restrictions prevent `PendingIntent.getActivity()` from working when the phone screen is off. This means Messenger's "Like" reaction button fails silently on the phone.

Our workaround: a **Smart Emoji Sniffer** that intercepts blocked action buttons and routes reaction emojis (❤️, 🔥, 👍, 💞) via `RemoteInput` reply instead:

```vala
// notification.vala — intercept blocked actions
if (pkg == "com.facebook.orca" && key == "Like") {
    // Route via RemoteInput instead of PendingIntent
    send_emoji_fallback(device, id, emoji);
}
```

It's a hack, but it works. And it's configurable per-package.

### Cinnamon Inline Reply Integration

Cinnamon's notification applet supports inline reply via `org.freedesktop.Notifications.NotificationReplied` D-Bus signal. We integrated this natively:

1. Daemon emits `notification_received` signal with `has_reply: true` and `reply_label` metadata.
2. GUI shows an inline reply entry at the bottom of the device detail page.
3. User types reply → `SendReply()` D-Bus call → daemon sends `kdeconnect.notification.reply` packet with `requestReplyId`.

No zenity popups. No separate dialogs. Just native desktop integration.

## The CLI: Power in Simplicity

Here's where xconnect really shines for automation. Every D-Bus method is exposed as a CLI command with **smart device targeting**:

```bash
# Target by short ID
xconnectctl share-file 80eeceab /path/to/photo.jpg

# Target by name (partial match)
xconnectctl send-sms Galaxy "+1234567890" "Meeting at 5 PM"

# Target by index
xconnectctl notifications 0

# Target by full D-Bus path
xconnectctl ping /org/xconnect/device/80eeceab "Hello from terminal"
```

This means you can script workflows:

```bash
#!/bin/bash
# Daily morning routine
xconnectctl show-battery Galaxy
xconnectctl show-connectivity Galaxy
xconnectctl share-url Galaxy "https://calendar.example.com/today"
```

Or integrate with other tools:

```bash
# Share clipboard content with phone
xconnectctl share-text 0 "$(xclip -selection clipboard -o)"

# Forward notification to Telegram bot
xconnectctl notifications 0 | grep -i "urgent" | curl -s -X POST "https://api.telegram.org/bot$TOKEN/sendMessage" -d chat_id=$CHAT_ID -d text="$(cat)"
```

The CLI isn't an afterthought — it's a first-class citizen designed for power users and automation engineers.

## What's Next

The roadmap (see [FEATURES.md](https://github.com/Twilight0/xconnect/blob/main/FEATURES.md)) includes:

- **Priority 1**: SFTP remote filesystem access — mount phone storage via `sshfs`/`gio mount`
- **Priority 2**: SMS conversation threads, MMS support, configurable app reaction fallback engine
- **Priority 3**: Contacts sync, digitizer/stylus support, share input devices

## Try It Out

```bash
# Arch Linux
git clone https://github.com/Twilight0/xconnect
cd xconnect
makepkg -si

# Enable and start
systemctl --user enable --now xconnect

# List devices
xconnectctl list-devices
```

No KDE dependencies. Just Vala, GTK3, and a clean D-Bus interface.

---

*Got feedback? Found a bug? Open an issue on [GitHub](https://github.com/Twilight0/xconnect) or join the discussion on the [AliveOS Forum](https://aliveos.org/forum).*
