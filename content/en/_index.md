---
title: "Overview"
weight: 10
---

# go-theca documentation {#go-theca}

go-theca holds **Theca**, an encrypted container format (a file or a block device
that holds a volume) written in pure Go with `CGO_ENABLED=0`. The goal is a format
that is better than the state of the art in composition, that is, in how known,
analysed primitives are put together, while inventing no primitive and no mode.

The project is pre-release: v0.1 is in progress and the format can still change.
Nothing is a commitment until a v0.1 tag says so. The repository has no tag yet.
Theca has a library and a command line (create, info, verify, import, export,
enroll-key); it does not mount. A container opens with a password, a recovery
code, or a FIDO2 key paired with one of them. This site was updated from the
repository's `main` branch on 2026-11-16.

## The repositories {#repositories}

| Page | Tag | What it answers |
| --- | --- | --- |
| [theca]({{< relref "repos/theca.md" >}}) | no tag | what Theca is, its factors, the command line, security notes, measured performance |

Links: the [organisation](https://github.com/go-theca) on GitHub, and the
[landing page](https://go-theca.github.io/).
