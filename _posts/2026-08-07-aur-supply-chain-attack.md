---
layout: post
title: "AUR Supply Chain Attack: What Happened and How AliveOS Users Are Protected"
date: 2026-08-07 09:00:00 +0000
author: The AliveOS Project
---

## The Incident

If you run Arch Linux or any Arch-based distribution, you've likely heard about the **"Atomic Arch"** campaign — one of the largest supply chain attacks in the AUR's history.

Since mid-June 2026, attackers have systematically compromised over **400 AUR packages** by exploiting the package adoption process to take over orphaned packages and inject malicious build scripts.

Arch Linux has now **disabled AUR package adoption** entirely while they work on a permanent fix.

## Timeline

| Date | Event |
|------|-------|
| **Late May 2026** | First reports of suspicious npm hooks in AUR packages |
| **June 10, 2026** | Malicious npm package `atomic-lockfile@1.4.2` published |
| **June 11, 2026** | Community discovers the attack; consolidated report thread opened |
| **June 12, 2026** | Arch Linux issues official security announcement |
| **June 17, 2026** | AUR signups suspended; package adoption disabled |
| **July 30, 2026** | Arch disables package adoption feature entirely |
| **August 1, 2026** | Continued monitoring; community cleanup ongoing |

## How the Attack Worked

The attack chain was elegantly simple:

1. **Target orphaned packages** — packages with install counts but no active maintainer
2. **Adopt the package** through AUR's standard adoption process
3. **Modify the PKGBUILD** to add a malicious npm dependency
4. **Execute during install** — the npm package's `preinstall` script runs a native Linux binary

The malicious npm packages (`atomic-lockfile`, `js-digest`) contained a multi-phase infostealer that targeted:

- Browser-stored credentials and cookies
- SSH keys and certificates
- System environment variables
- Cryptocurrency wallet data
- Session tokens

The payload included rootkit-like capabilities with anti-debugging measures and eBPF-related code for persistence.

## Why It Worked

The AUR's strength — community trust and low barrier to entry — became its weakness:

- **No central security review** for every PKGBUILD commit
- **Familiar package names** with established install histories
- **Trust based on maintainer reputation** that could be impersonated
- **Malicious logic hidden in install hooks** rather than the PKGBUILD itself

Users running `yay -S some-package` would execute arbitrary maintainer code during installation, often without reviewing the changes.

## How AliveOS Users Are Protected

AliveOS was designed with exactly this scenario in mind. Here's why our users are safe:

### 1. Custom Repository Model

AliveOS maintains its own package repository at `https://github.com/Twilight0/aliveos-repo`. Packages are:

- **Built from verified sources** — our build script clones from AUR and builds deterministically
- **Hosted on GitHub Releases** — not directly from AUR's git repositories
- **Signed with repo-add** — the pacman database is generated locally, not fetched from AUR

### 2. Upstream Sources from Developers Themselves

For packages we control, we pull directly from the developers' own GitHub repositories — not from AUR. This eliminates the middleman entirely:

```bash
PACKAGES=(
  "upstream|https://github.com/Twilight0/dory.git|dory|"
  "upstream|https://github.com/Twilight0/cinnamon-aliveos.git|cinnamon-aliveos|"
  "upstream|https://github.com/httptoolkit/httptoolkit-desktop.git|httptoolkit|"
  # ... more upstream sources
)
```

This means packages like **dory**, **cinnamon-aliveos**, **aliveos-settings**, **xconnect**, and **httptoolkit** are built directly from the source repositories of their respective developers — no AUR intermediary, no possibility of package adoption attacks.

### 3. Curated AUR Packages

Our build script (`build-packages.sh`) explicitly lists every AUR package we build. There are no automatic pulls or unreviewed additions:

```bash
PACKAGES=(
  "aur|xlibre-video-ati|"
  "aur|cogl|"
  "aur|clutter|"
  # ... explicit list continues
)
```

### 4. Automated PKGBUILD Security Scanner

Every PKGBUILD — whether from AUR or upstream — passes through our automated security scanner before building. The scanner checks for:

- **Known malicious npm packages** — detects `atomic-lockfile`, `js-digest`, and similar payloads
- **Suspicious curl/wget pipes** — catches `curl | sh` style attacks
- **Base64 obfuscation** — flags encoded executable content
- **Reverse shell indicators** — detects `/dev/tcp` and other network exfiltration patterns
- **npm install in non-JS packages** — catches the exact pattern used in the Atomic Arch attack
- **Suspicious post_install hooks** — scans for `curl`, `exec`, `eval`, and other dangerous commands in install scripts

If any issues are detected, the package is **skipped entirely** and the build fails with a clear warning. This provides an automated safety net beyond manual review.

### 5. Build-Time Isolation

Even if an AUR package were compromised, the attack window is limited:

- Packages are built in a **CI/CD environment** (GitHub Actions)
- Built packages are uploaded to GitHub Releases, not installed directly
- Users install from our repo, not from AUR directly

### 6. No AUR Helper Dependency

AliveOS doesn't require users to run `yay`, `paru`, or other AUR helpers. Our custom repo provides pre-built packages through standard `pacman`:

```bash
sudo pacman -S dory xconnect respite
```

No PKGBUILD execution at install time. No arbitrary code from AUR maintainers.

## What You Should Do

### If you're running plain Arch Linux:

1. **Check for compromised packages:**
   ```bash
   # Search for packages with recent ownership changes
   pacman -Qii | grep -i "Name\|Packager\|Install Date"
   ```

2. **Review any AUR packages you have installed:**
   ```bash
   yay -Qm  # List AUR packages
   ```

3. **Look for red flags:**
   - New `npm` dependencies in non-JavaScript packages
   - Post-install scripts invoking `npm install atomic-lockfile`
   - Maintainer email changes without clear upstream handoff

4. **Monitor the arch-general mailing list** for updates

### If you're running AliveOS:

Your system uses our custom repo. As long as you're installing packages from our repository (not directly from AUR), you're protected by our build pipeline.

However, if you've manually installed AUR packages outside our repo, audit them.

## The Bigger Picture

This incident highlights why AliveOS exists. The AUR is a fantastic resource, but its trust model has inherent risks. By maintaining a curated, built-from-source repository, we provide:

- **Reproducibility** — same sources, same build process, same output
- **Auditability** — every package in our repo is explicitly listed and reviewed
- **Isolation** — users don't need to execute arbitrary PKGBUILD code

We're not replacing the AUR — we're adding a layer of curation on top of it.

## References

- [Arch Linux Official Announcement](https://archlinux.org/news/active-aur-malicious-packages-incident/)
- [BreachHistory: Atomic Arch Campaign](https://breachhistory.com/blog/arch-linux-aur-atomic-arch-supply-chain-june-2026)
- [Sonatype: Atomic Arch Analysis](https://www.sonatype.com/blog/atomic-arch-npm-campaign-adds-malicious-dependency)
- [EndeavourOS Forum Discussion](https://forum.endeavouros.com/t/malicious-aur-packages/80095)

---

*Questions? Join the discussion on the [AliveOS Forum](https://aliveos.org/forum).*
