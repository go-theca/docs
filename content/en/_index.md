---
title: "Overview"
weight: 10
---

# go-theca documentation {#go-theca}

go-theca holds **Theca**, an encrypted container format (a file or a block device
that holds a volume) written in pure Go with `CGO_ENABLED=0`. The goal is a format
that is better than the state of the art in composition, that is, in how known,
analysed primitives are put together, while inventing no primitive and no mode.

Theca v0.9.0 is released (v0.2.0 to v0.9.0, 2026-11-16 and 2026-11-17); see its
[CHANGELOG](https://github.com/go-theca/theca/blob/v0.9.0/CHANGELOG.md). The
format may still change before v1 (below v1, a breaking change bumps the minor
version). Theca has a library and a command line (create, info, verify, import,
export, enroll-key, serve). Since v0.2.0 it can serve the files inside a
container over SMB on loopback and mount them in the file manager; since v0.3.0
that is a library package, `share`, and the [go-theca/gui](https://github.com/go-theca/gui)
desktop window (v0.2.0) uses it. A container opens with a password, a recovery
code, or a FIDO2 key paired with one of them. This site describes theca tag
v0.9.0 and gui tag v0.2.0.

## The repositories {#repositories}

| Page | Tag | What it answers |
| --- | --- | --- |
| [theca]({{< relref "repos/theca.md" >}}) | v0.9.0 | what Theca is, its factors, the command line, `slots` (list, remove), bounded caches, security notes, measured performance |
| [serving and mounting]({{< relref "repos/serve.md" >}}) | v0.2.0 | `create --fs exfat`, `serve`, `serve --mount`, `serve --network`: the files inside, over SMB on loopback or, signed and encrypted, beyond |
| [share (library)]({{< relref "repos/share.md" >}}) | v0.5.0 | the `share` package: `Start`, `Stop`, `URL`, `MountPath`, `Done`, `Err`, `Format`, `Network` |
| [gui]({{< relref "repos/gui.md" >}}) | v0.2.0 | the desktop window, with its Share / Stop sharing toggle |

Links: the [organisation](https://github.com/go-theca) on GitHub, and the
[landing page](https://go-theca.github.io/).
