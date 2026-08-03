---
layout: post
title: "Dory: Why We Forked Nemo and What It Took to Build a Proper File Chooser Portal"
date: 2026-08-01 10:00:00 +0000
author: The AliveOS Project
---

## The Spark That Started AliveOS

Every distribution has a catalyst. For AliveOS, it was a file picker dialog.

I was running Cinnamon on Arch, and every time a Flatpak app needed to open or save a file, the portal dialog was either missing, broken, or looked like it belonged in a different desktop environment. The `xdg-desktop-portal-gtk` backend worked, but it was generic — no custom actions, no proper sidebar integration, no native feel.

I tried configuring `xdg-desktop-portal-xapp-filepicker`. It worked better, but Nemo's portal mode had quirks: it wouldn't stay focused, multiselect was flaky, and the save dialog didn't handle overwrites gracefully.

So I asked myself: what if I just forked Nemo and made it do exactly what I needed?

That question became Dory. And Dory became the reason AliveOS exists.

## What Dory Actually Is

Dory is a **standalone file chooser portal backend** and file manager, forked from [Nemo](https://github.com/linuxmint/nemo) (the Cinnamon file manager). But it's not Nemo with a different name. Every namespace — binaries, D-Bus services, GSettings schemas, desktop files, library paths — has been renamed from `nemo` to `dory`.

This means Dory can be installed **side-by-side with Nemo** without conflicts. It doesn't steal your file manager associations. It doesn't overwrite your schemas. It just quietly handles portal file dialogs when called by `xdg-desktop-portal-xapp-filepicker`.

## The Technical Challenge: Portal Dialogs Are Hard

File chooser portals are deceptively complex. Here's what the D-Bus interface (`org.Dory.FileChooser`) needs to handle:

```c
// OpenFile — standard file/folder selection
void open_file(
    string parent_window,
    string title,
    string[] accept_label,
    string[] options  // filters, multiselect, directory mode, etc.
);

// SaveFile — single file save with overwrite confirmation
void save_file(
    string parent_window,
    string title,
    string accept_label,
    string options  // suggested_name, current_folder, etc.
);

// SaveFiles — multi-file save (for GIMP layers, etc.)
void save_files(
    string parent_window,
    string title,
    string accept_label,
    string[] files,  // list of suggested filenames
    string options
);
```

Each of these needs to:
1. Spawn a dialog window
2. Track focus and lifecycle via GApplication holds
3. Handle the case where the user cancels
4. Return URIs or throw `org.freedesktop.DBus.Error.NoResponse`

The **GApplication lifecycle** was the first major roadblock.

## Roadblock #1: The Daemon That Wouldn't Stay Alive

When a Flatpak app requests a file, the portal backend spawns, shows the dialog, waits for user input, then returns the result. Simple in theory. In practice, the D-Bus service activation would spawn the process, the dialog would appear, but the process would exit before the user could click anything.

Root cause: GApplication was releasing its hold when the D-Bus method call completed, not when the dialog closed.

The fix: **restructure the D-Bus service activation to maintain GApplication holds throughout the dialog lifecycle**:

```c
// Before: hold released immediately after method dispatch
g_application_release(app);

// After: hold maintained until dialog response
g_application_hold(app);
// ... show dialog ...
// In dialog response handler:
g_application_release(app);
```

This kept the process alive until the user actually selected or cancelled.

## Roadblock #2: Focus Stealing and Dialog Z-Order

Portal dialogs need to appear above the requesting application. But X11 window managers (and Wayland compositors) have their own ideas about focus. The dialog would appear behind the parent window, or the parent window would steal focus back.

The fix was a combination of `gtk_window_present()` with `gdk_window_raise()` and `gdk_window_focus()` calls, timed after the dialog realized:

```c
static void on_dialog_realized(GtkWidget *dialog, gpointer user_data) {
    GdkWindow *gdk_window = gtk_widget_get_window(dialog);
    gdk_window_raise(gdk_window);
    gdk_window_focus(gdk_window, GDK_CURRENT_TIME);
}
```

Even then, some window managers fight you. We added a GSettings persistence layer to remember the last-used directory and dialog state across invocations.

## Roadblock #3: Multiselect URI Reconstruction

The portal passes selected files as a list of URIs. But when multiselect is enabled, the D-Bus response needs to reconstruct the full URI list from the internal selection model.

The bug: `get_selected_uris()` was only returning the first selected file when multiple files were chosen.

Root cause: the selection model was being rebuilt during the response dispatch, losing the intermediate state.

Fix: **rebuild the URI list from all selected paths before the dialog closes**:

```c
// Rebuild selected_uris from all selected paths
GList *selected_paths = gtk_icon_view_get_selected_items(view);
for (GList *l = selected_paths; l != NULL; l = l->next) {
    GtkTreePath *path = l->data;
    // ... convert to URI and add to list ...
}
```

## Roadblock #4: Save Dialog Overwrite Confirmation

The standard portal save dialog doesn't handle overwrites — it just returns the path. But users expect a "file already exists, overwrite?" confirmation.

We implemented an **intelligent overwrite modal**: when the user selects a path that already exists, the dialog intercepts the selection, shows a confirmation dialog, and keeps the picker open if the user declines:

```c
if (g_file_exists(selected_file)) {
    // Show overwrite confirmation dialog
    int response = gtk_dialog_run(GTK_DIALOG(overwrite_dialog));
    if (response == GTK_RESPONSE_NO) {
        // Keep picker open, let user choose different path
        return;
    }
}
// Proceed with save
```

This required careful coordination between the dialog's response handlers and the GApplication lifecycle.

## Roadblock #5: The libglycin Thumbnail Minefield

Dory generates thumbnails for files in the portal dialog. This uses `libglycin` for image loading. Here's the problem: **libglycin is loaded into the Cinnamon process** (the desktop shell). A crash in libglycin doesn't just crash Dory — it segfaults Cinnamon and freezes the entire desktop.

The AGENTS.md in the Dory repo spells this out clearly:

> 1. Never let invalid/untrusted data reach libglycin unchecked.
> 2. Catch and handle all libglycin errors gracefully.
> 3. Avoid resizing images in-place on the main thread without bounds checks.
> 4. Test thumbnail code under stress — large images, corrupt files, zero-byte files, symlink loops.

We added **validation layers** before any image data reaches libglycin: dimension checks, format verification, and buffer size validation. The thumbnail generation runs in a separate thread with error boundaries that catch and log failures without propagating them.

## Recent Development: Post-Fork Commits

Here's what's been shipping since the initial fork:

| Version | Key Changes |
|---------|-------------|
| **6.7.6** | Wayland monitor removal crash fix, DnD search result protection, interactive label cleanup |
| **6.7.5** | Robust initial/last directory resolution, non-directory save prevention |
| **6.7.4** | Action layout editor renamed from nemo to dory, focus force fix, multiselect rebuild |

The commit history tells the story of iterative hardening:

```
7c76df75 nemo-desktop: Don't crash/quit in Wayland when the monitor is removed
2e7fe76d dnd: Don't allow drops into a search result view
77f49f79 chooser: robustly resolve initial/last directory and prevent saving non-directories
5fcebdde refactor: rename action layout editor from nemo to dory
9eecb1c8 fix: force focus on file chooser dialog with gdk_window_raise + gdk_window_focus
67fee9c6 fix: rebuild selected_uris from all selected paths for proper multiselect support
66302aa3 fix: raise file chooser dialog to foreground with gtk_window_present
0ccffa54 fix: save-mode folder selection + add SaveFiles D-Bus method
539b8a9e Properties: Add Stop button for folder disk usage scanning
```

## Current Status: Stable and Shipping

Dory is now at **v6.7.6** and ships as the default file chooser portal backend in AliveOS. It's available in the AUR as `dory-git` and declared as providing/conflicting with `nemo` to satisfy Cinnamon's dependencies.

The extensions ecosystem is also growing:
- `dory-terminal` — embedded terminal in file manager
- `dory-share` — Samba/network sharing integration
- `dory-preview` — enhanced file previews
- `dory-python` — Python scripting support
- `dory-compare` — file comparison tools

## What Dory Taught Me

Building Dory taught me three things about file managers:

1. **Portal dialogs are first-class citizens.** In a world of Flatpaks and sandboxed apps, the file picker isn't just a UI convenience — it's a system service.

2. **Namespace isolation matters.** If you want your fork to coexist with the original, you need to rename everything. Every binary, every D-Bus name, every schema path.

3. **Thumbnail loading is a trust boundary.** When your file manager loads arbitrary images for previews, you're one malformed JPEG away from crashing the entire desktop.

Dory is the piece that made me realize: if I wanted a file manager that did exactly what I needed, I'd have to build it myself. And if I was building a file manager, I might as well build a distribution around it.

---

## Acknowledgments

Dory wouldn't exist without the incredible work of the **Cinnamon and Linux Mint development team**. Nemo is a polished, well-engineered file manager — forking it gave us a solid foundation that would have taken years to build from scratch.

Special thanks to:
- **Clement Lefebvre** and the Linux Mint team for maintaining Nemo and the broader Cinnamon ecosystem
- **Michael Webster** and other Nemo contributors for the extension framework and D-Bus interfaces that Dory builds upon
- The **XApp** team for the toolkit components that make GTK-based desktop apps feel native across Cinnamon, MATE, and XFCE

We stand on the shoulders of giants. Dory is our attempt to give back — a specialized tool for a specific use case, built on the excellent foundation the Mint team created.

---

*Dory is available at [github.com/Twilight0/dory](https://github.com/Twilight0/dory). Extensions are at [github.com/Twilight0/dory-extensions](https://github.com/Twilight0/dory-extensions). Join the discussion on the [AliveOS Forum](https://aliveos.org/forum).*
