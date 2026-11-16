---
title: "Library: share"
weight: 30
---

# The `share` package {#share}

Since [v0.3.0](https://github.com/go-theca/theca/releases/tag/v0.3.0), the logic
of [`theca serve`]({{< relref "serve.md" >}}) is a library package,
`github.com/go-theca/theca/share`, so that a program other than the command line
(the [GUI]({{< relref "gui.md" >}}) imports it) can share a container. This page
is written from the
[README at tag v0.3.0](https://github.com/go-theca/theca/blob/v0.3.0/README.md)
(section "Sharing the files inside") and its
[CHANGELOG](https://github.com/go-theca/theca/blob/v0.3.0/CHANGELOG.md); where it
differs from the repository, the repository wins.

```go
s, err := share.Start(ctx, c, share.Options{
	Addr: "127.0.0.1:0", Name: "theca", Sync: 5 * time.Second, Mount: true,
})
if err != nil { // share.ErrNoFilesystem: no exFAT inside
	return err
}
fmt.Println(s.URL(), s.MountPath()) // smb://127.0.0.1:58738/theca /Volumes/theca
// <-s.Done() fires when a commit or the server fails; s.Err() says why.
err = s.Stop() // unmount, close the server, close the filesystem (idempotent)
c.Close()      // the caller closes the container, after Stop
```

## API {#api}

- `share.Start(ctx, c, Options{Addr, Name, Sync, Mount})` serves the exFAT inside
  an unlocked container over SMB on loopback and optionally mounts it.
- `URL`, `User`, `Password` and `MountPath` describe the share. `Password` exists
  to hand the account to another client; it is a secret, never to be shown or
  logged.
- `Done` and `Err`: a failed commit or server ends the share, and `Err` says why.
- `Stop` is idempotent: it unmounts, closes the server and closes the filesystem.
  It never closes the container; the caller does, after `Stop`.
- A container with no exFAT inside is refused with `share.ErrNoFilesystem`.
- Only a loopback address is served.

`theca serve` is a thin front over this package. Its behaviour is unchanged,
except that the address error now reads `address X: only a loopback address is
served` and a failed mount no longer prints the `serving` line first.
