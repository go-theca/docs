---
title: "gui"
weight: 40
---

# gui {#gui}

[go-theca/gui](https://github.com/go-theca/gui) is a portable desktop GUI for
Theca containers, built with [go-widgets](https://github.com/go-widgets). Release:
**v0.1.0**, the first. This page is written from the
[README](https://github.com/go-theca/gui/blob/v0.1.0/README.md) and
[CHANGELOG](https://github.com/go-theca/gui/blob/v0.1.0/CHANGELOG.md) at that tag;
where it differs from the repository, the repository wins.

```sh
go install github.com/go-theca/gui@latest   # the binary is named gui
```

It is pure Go (`CGO_ENABLED=0`): a Cocoa window on macOS, X11 or Wayland on Linux,
Win32 on Windows.

## What it does {#what}

It does what the `theca` command does, in a window: create a container, unlock it,
look at it, verify it, copy a file in and out, pair a FIDO2 security key with it,
and share the exFAT filesystem inside it.

- **Create.** File, size, policy (*any* or *2of3*) and the password twice. The
  recovery code is shown once, and the screen cannot be left until you tick *I
  have written it down*.
- **Unlock.** Password and/or recovery code, and a *Use security key* toggle.
- **Container.** Size, key slots, journal, rollback status; *Verify*, *Import*,
  *Export* (to a new file), *Enroll security key*, *Close container*.
- **Share / Stop sharing.** On the Container screen, *Share* serves the exFAT
  filesystem inside the container over SMB on a loopback address and mounts it in
  the file manager with no password dialog, through the
  [`share` package]({{< relref "share.md" >}}) of theca v0.3.0. The screen shows
  where it is shared and mounted, and *Open in file manager* shows the volume.
  The share stops, unmounting first, on the toggle, on *Close container* and on
  closing the window: always before the container is committed and closed. The
  share's account password is never shown. A container with no exFAT inside is
  refused with a message.

## Limits {#limits}

- **Share needs an exFAT volume**, and *Create new container* in v0.1.0 makes a
  raw one: make the container with `theca create --fs exfat` to share it.
- Mounting is measured on **macOS**. Linux (gvfs) and Windows
  (`MSFT_SmbMapping`) go through the same library but have not been run from this
  GUI.
- FIDO2 is macOS-only for now.
- The Linux and Windows windows are built and cross-compiled in CI, but have not
  been run by hand yet.
- There is no file picker yet: paths are typed or pasted.
- No rollback counter support.
