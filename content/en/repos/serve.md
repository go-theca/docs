---
title: "Serving and mounting"
weight: 20
---

# Serving the files inside {#serve}

Since [v0.2.0](https://github.com/go-theca/theca/releases/tag/v0.2.0), Theca can
hold an exFAT filesystem and serve it over SMB, so a container is used like a
disk without ever writing a decrypted copy. This page is written from the
[README at tag v0.2.0](https://github.com/go-theca/theca/blob/v0.2.0/README.md)
(section "Serving the files inside") and its
[CHANGELOG](https://github.com/go-theca/theca/blob/v0.2.0/CHANGELOG.md); where it
differs from the repository, the repository wins.

```sh
theca create vault.theca --size 4G --fs exfat      # a container holding an exFAT filesystem
theca serve vault.theca --mount                    # appears in the Finder; Ctrl-C unmounts and stops
# serving smb://127.0.0.1:58738/theca; Ctrl-C to stop
# mounted at /Volumes/theca
theca serve vault.theca --credentials-out creds.json   # or: hand the account to another client
```

## What each command does {#commands}

- `create --fs exfat` formats the volume through the container itself
  (go-filesystems/exfat `FormatImage`). No decrypted copy is ever written to
  disk.
- `serve` unlocks the container and serves its exFAT over SMB (2.1 to 3.1.1,
  signed) using [go-filesystems/smb](https://github.com/go-filesystems/smb)
  v0.15.0.
- `serve --mount` mounts the share in the file manager (Finder, GNOME Files,
  Explorer) with **no password dialog**, through
  [go-fileshare/automount](https://github.com/go-fileshare/automount): the
  password goes to the operating system's mount service as a value. At stop it
  unmounts first, so the client flushes through the server, then commits and
  closes the container. `--credentials-out` is no longer required when `--mount`
  is given.
- `--credentials-out` writes the URL, user and password to a file (mode `0600`,
  created only if absent, removed at exit), for another client. One of
  `--mount` and `--credentials-out` is required.
- Writes are committed to the container every `--sync` (default 5 s) and when
  the server stops.

## Security {#security}

- **Two modes.** The data served is decrypted, so `serve` has two modes:
  - **Loopback (the default).** `serve` listens on a loopback address only. A
    non-loopback `--addr` is refused, and the refusal names `--network`.
  - **`--network`** (`share.Options{Network: true}`, since theca v0.5.0). `serve`
    listens beyond loopback only with message signing **and** encryption required
    together (of every session and of the share), never one without the other: a
    client that cannot do both (SMB 2.1, or one that refuses encryption) is
    refused. This is go-filesystems/smb v0.15.0 `RequireSigning`, `EncryptData`
    and `RequireEncryption`. A warning goes to standard error. It does not make
    serving safe on a hostile network: anyone who can reach the address and has the
    password can read and write the container's files, and the password is in
    `--credentials-out`.
- With `--network --mount` and an unspecified address (`0.0.0.0`, `::`), the local
  mount uses `127.0.0.1` on the same port; another machine uses this machine's
  address with that port.
- The [GUI]({{< relref "gui.md" >}}) has no network option: it always serves
  loopback.
- **One random account per share**, never printed or passed as an argument. Since
  v0.10.2 the account **name** is random too: `theca-` and 8 random base32
  characters (`Share.User()`, `share.UserPrefix`), written in `--credentials-out`.
  Before, every share used the name `theca`, and on macOS a second mount in the
  same session failed ("authentication error (errno 80)"), because macOS replays a
  stale credential for a fixed account name on 127.0.0.1. A fixed name failed 12
  of 12 repeats; a random name 0 of 120 (the smb author's harness).

## Measured, and where {#measured}

Measured on **macOS 27 only** (2026-11-16): `--mount` lands in `/Volumes/theca`
over SMB 3.1.1 signed with AES-128-GMAC, a file written through it survives a
stop and a remount, and `verify` passes. Linux (GNOME Files, through gvfs) and
Windows (Explorer, a drive letter through `MSFT_SmbMapping`) use the same
command; **neither has been measured with theca yet**.

`--network` was also measured on macOS: `smbutil` reports AES-128-GCM, GMAC, and
`ENCRYPTION_REQUIRED` and `SIGNING_REQUIRED` both TRUE.

## A limit {#limit}

A copy of a file written through the share is one transaction per journal
budget, not one per file (DESIGN section 19, I12). A crash in the middle of a
large copy can leave the exFAT metadata of that copy half-written, as on a disk.
