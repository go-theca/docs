---
title: "Overview"
weight: 10
---

# go-theca documentation {#go-theca}

go-theca holds **Theca**, an encrypted container format (a file or a block device
that holds a volume) written in pure Go with `CGO_ENABLED=0`. The goal is a format
that is better than the state of the art in composition, that is, in how known,
analysed primitives are put together, while inventing no primitive and no mode.

Theca v0.1.0 is released; see its
[CHANGELOG](https://github.com/go-theca/theca/blob/v0.1.0/CHANGELOG.md). The
repository's README still says the format can still change. Theca has a library and a command line (create, info, verify, import, export,
enroll-key); it does not mount. A container opens with a password, a recovery
code, or a FIDO2 key paired with one of them. This site describes tag v0.1.0.

## The repositories {#repositories}

| Page | Tag | What it answers |
| --- | --- | --- |
| [theca]({{< relref "repos/theca.md" >}}) | v0.1.0 | what Theca is, its factors, the command line, security notes, measured performance |

Links: the [organisation](https://github.com/go-theca) on GitHub, and the
[landing page](https://go-theca.github.io/).
