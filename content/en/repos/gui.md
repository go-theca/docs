---
title: "gui"
weight: 40
---

# gui {#gui}

[go-theca/gui](https://github.com/go-theca/gui) is a portable desktop GUI for
Theca containers, built with [go-widgets](https://github.com/go-widgets). Release:
**v0.3.1** (v0.1.0 was the first). This page is written from the
[README](https://github.com/go-theca/gui/blob/v0.3.1/README.md) and
[CHANGELOG](https://github.com/go-theca/gui/blob/v0.3.1/CHANGELOG.md) at that tag;
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

- **Create.** File, size, filesystem (*exFAT*, the default, or a *raw volume*),
  policy (*any* or *2of3*) and the password twice. The
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

## Since v0.3.0 {#v030}

- **Share on the network**, a toggle beside Share: it serves on every interface
  with message signing **and** encryption required of every client (theca
  `share.Options{Network: true}`). A warning says that anyone on the network with
  the password can connect. The screen shows where it listens and the port; the
  password stays hidden until *Show password* and is cleared at stop, lock and
  close. Live-tested on macOS: `smbutil` reports encryption and signing required.
- **Keyslots screen**, from the Container screen: it shows the recorded slots and
  which ones the password and/or recovery code you type now open, and removes the
  slot of a factor set while a **different** set to keep must still open the
  container. A removal that would leave no way in is refused and changes nothing.
  A removal only stops access to this copy.
- **The security key's state is shown on screen**: "touch your key", or "type its
  PIN" (masked, emptied on Submit, never in a message), then touch again. On
  Windows the screen says that Windows shows its own dialog. These states are
  tested with a **fake key**; a real-key run has been done by hand on macOS only.
  It uses theca v0.8.0 (FIDO2 on Linux and Windows).

v0.3.1 uses theca v0.10.2, which **fixes a second mount failing on macOS**
("authentication error (errno 80)"): every share now has its own random account
name. Live tests: network share 6 of 6, local share 3 of 3 (the failure was
intermittent before).

## Limits {#limits}

- **Share needs an exFAT volume.** Since v0.2.0, *Create* offers the filesystem:
  exFAT (the default, laid through the container by `share.Format` of theca
  v0.4.0, as `theca create --fs exfat` does; a volume too small for exFAT is
  refused and the half-made file removed) or a raw volume. A raw one cannot be
  shared. A shareable container is made from the GUI.
- Mounting is measured on **macOS**. Linux (gvfs) and Windows
  (`MSFT_SmbMapping`) go through the same library but have not been run from this
  GUI.
- FIDO2 is measured on a real key on macOS only; on Linux and Windows it is built
  but not measured.
- The Linux and Windows windows are built and cross-compiled in CI, but have not
  been run by hand yet.
- There is no file picker yet: paths are typed or pasted.
- No rollback counter support.
