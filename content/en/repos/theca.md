---
title: "theca"
weight: 10
---

# theca {#theca}

Theca is an encrypted container format in pure Go, with a library and a command
line. The library gives an `io.ReaderAt` and `io.WriterAt` over the logical
volume, and the command line creates, inspects, verifies, imports and exports
containers. Since v0.2.0 it also serves the files inside over SMB and mounts
them: see [Serving and mounting]({{< relref "serve.md" >}}).

The same logic is a library package, `share`, since v0.3.0: see
[Library: share]({{< relref "share.md" >}}).

Release: **v0.7.0** (tag `v0.7.0` of
[go-theca/theca](https://github.com/go-theca/theca); install with
`go install github.com/go-theca/theca/cmd/theca@v0.7.0`). What changed is in the
[CHANGELOG at v0.7.0](https://github.com/go-theca/theca/blob/v0.7.0/CHANGELOG.md).
The format may still change before v1. This page is written from that tag; where
it differs from the repository, the repository wins. Its links to the
repository's design documents still point at tag v0.1.0, and the performance
tables below were measured on the v0.1.0 commit.

## What it is {#what}

From the [README](https://github.com/go-theca/theca/blob/v0.1.0/README.md):

- authenticated sectors: XChaCha20 xor AES-256-CTR, with HMAC-SHA-256;
- a Merkle tree over the sector tags, and a write-ahead journal;
- no magic bytes;
- keyslots opened by a password (Argon2id), a recovery code, or both;
- an optional hardware rollback counter.

The design, its threat model, the sources it rests on and the decisions taken
are in [docs/DESIGN.md](https://github.com/go-theca/theca/blob/v0.1.0/docs/DESIGN.md);
section 0 is a one-table summary.

## Factors {#factors}

A container is opened by one of three factors:

- a **password**, stretched with Argon2id;
- a **recovery code**, printed once by `create` and never stored;
- a **FIDO2 security key**, which is never a factor alone: it always pairs
  with the password or with the recovery code.

The policy is chosen at creation. `--policy any` (the default) lets the password
alone, or the recovery code alone, open the container. `--policy 2of3` needs two
of the three: password and recovery code from the start, and a FIDO2 key
(`enroll-key`) then pairs with each.

## Command line {#cli}

```sh
go install github.com/go-theca/theca/cmd/theca@latest

theca create vault.theca --size 1G            # asks the password twice, prints the recovery code once
theca info   vault.theca                      # unlock and describe
theca verify vault.theca                      # authenticate every sector and tree node
theca import vault.theca --password-file pw --offset 0 < disk.img
theca export vault.theca --password-file pw > disk.img

theca enroll-key vault.theca --recovery-file rc   # pair the key with the password and the recovery code
theca info vault.theca --fido2                    # password + key
theca info vault.theca --recovery-file rc --fido2 # recovery code + key
```

- **Secrets never go in arguments or environment variables.** The password is
  read from the terminal with echo off, or from `--password-file`. The recovery
  code is read from `--recovery-file`. Both files must be regular files that only
  their owner can read (mode `0600` or stricter); anything else is refused.
- `create` prints the recovery code once, on standard output. If you redirect it
  to a file, that file gets your umask (often `0644`): restrict it, or better,
  write the code down on paper.
- `import` reads its data from standard input, so it needs `--password-file` or
  `--recovery-file`. It is not one atomic transaction: on an error, the bytes
  written before the error are kept.
- `enroll-key` makes one discoverable credential on the key (relying party
  `theca`, `credProtect` 3, `hmac-secret`) and pairs it with the password, and
  with the recovery code too when `--recovery-file` names it. The key is touched
  twice: to register, then to confirm.
- `--fido2` adds the attached key to an unlock. A key without a fingerprint asks
  for its PIN, read from `--pin-file` (mode `0600` or stricter) or from the
  terminal, never from an argument.

`theca create --fs exfat` and `theca serve` (with `--mount`) are described in
[Serving and mounting]({{< relref "serve.md" >}}).

## Slots and caches (v0.6.0) {#slots}

- `theca slots list` and `theca slots remove` list the key slots and remove one
  by the factors that open it
  ([go-theca/theca#5](https://github.com/go-theca/theca/issues/5)). In the
  library: `Container.RemoveFactors(remove, keep)`, `SlotsOf`, `Slots` and
  `ErrLastWay`. A removal that would leave the `keep` set with no other slot is
  refused before any write.
- The metadata block cache and the Merkle node cache of an open container are
  bounded
  ([#6](https://github.com/go-theca/theca/issues/6)):
  `OpenOptions.CacheBytes` and `CreateOptions.CacheBytes`, default 64 MiB. Clean
  entries are evicted least-recently-used and wiped; dirty ones are pinned until
  commit; an evicted entry is authenticated again when read. `Close` wipes the
  caches.

## Later releases {#later}

- **v0.6.1**: go-filesystems/smb v0.16.1, with per-source connection caps (32
  per source, 16 pre-auth; loopback is not exempt, and the limit is far above one
  Finder client) and a zone-id fix.
- **v0.7.0**: AES-256-CTR runs through
  [go-encryptions/ctr](https://github.com/go-encryptions/ctr) (round keys expanded
  once, one kernel call per sector, no allocation; `Zeroize` wipes the round
  keys), and Argon2id through
  [go-encryptions/argon2](https://github.com/go-encryptions/argon2), the
  `x/crypto/argon2` code with a NEON kernel on arm64. See the performance
  section below.

Not in this version: FIDO2 on Linux and Windows, and the TPM counter.

## Security notes {#security}

The threat model is
[DESIGN section 1](https://github.com/go-theca/theca/blob/v0.1.0/docs/DESIGN.md).
The places where the code differs from, or fixes the details of, the design are
in its [section 19, the implementation log](https://github.com/go-theca/theca/blob/v0.1.0/docs/DESIGN.md);
the FIDO2 design and its trade-offs are in
[HARDWARE-KEYS.md section 9, "H2 revised"](https://github.com/go-theca/theca/blob/v0.1.0/docs/HARDWARE-KEYS.md).

Known limits, as the repository states them:

- **No rollback counter in the command line yet.** Every command that opens a
  container says whether it is protected against rollback; in this version it
  never is, so an older copy of a container would be accepted.
- **FIDO2 works on macOS only** for now (through go-macos/fido). It was tested
  on a YubiKey 5, firmware 5.7.4.
- **A reader of the FIDO2 salt can test password guesses.** The salt that a key
  is asked for is derived from the partner factor, so for whoever reads it (a
  forged authenticator, a leaked log) it is a password verifier, at Argon2id cost
  per guess. The salts are wiped after each call and never logged (DESIGN
  section 19, I21).
- **Revoked factors:** older copies of a container still open with a factor that
  was removed (DESIGN section 19, I15).
- **Tree area:** a container at rest is indistinguishable from random except the
  Merkle tree area, which shows how sparse the container is (DESIGN section 19,
  I11).

## Performance {#performance}

Theca is measured against C references on one core with
[go-scikit/refbench](https://github.com/go-scikit/refbench): libsodium for
XChaCha20 and Argon2id, OpenSSL for AES-256-CTR and HMAC-SHA-256. The procedure
and the cases are in
[bench/README.md](https://github.com/go-theca/theca/blob/v0.1.0/bench/README.md).
Ratio is Go time divided by reference time; below 1 is faster. Both runs are on
commit `6d515c5` with go1.27.4, libsodium 1.0.22 and OpenSSL 4.0.2, from
2026-11-15. Medians in ns/call; the full tables, with quartiles and allocation
counts, are
[docs/perf-table.md](https://github.com/go-theca/theca/blob/v0.1.0/docs/perf-table.md) and
[docs/perf-zen4-table.md](https://github.com/go-theca/theca/blob/v0.1.0/docs/perf-zen4-table.md).

| case | Apple M4 Max (Go / ref, ratio) | AMD Zen 4 (Go / ref, ratio) |
| --- | --- | --- |
| Sector/XChaCha20 | 2286 / 4143, 0.55 faster | 4016 / 9195, 0.44 faster (see caveat) |
| Sector/AES-256-CTR | 459.4 / 308.9, 1.49 slower | 432.7 / 357.1, 1.21 slower |
| Sector/Tag (HMAC-SHA-256) | 1493 / 1428, 1.05 slower | 1691 / 1736, 0.97 faster |
| Sector/Seal | 4290 / 5935, 0.72 faster | 6140 / 11330, 0.54 faster |
| Sector/Open | 4296 / 5992, 0.72 faster | 6154 / 11380, 0.54 faster |
| Argon2id t3-m64MiB-p1 | 9.424e+07 / 6.306e+07, 1.49 slower | 6.616e+07 / 2.662e+08, 0.25 faster (see caveat) |

The Apple M4 Max run was at a 1-minute load of 7.22 to 7.59 (gate 8.00); the
Zen 4 machine is an AMD Ryzen 9 7950X pinned to one CPU, at a load of 0.16 to
0.35. Both runs are marked valid. A gap in a row includes Theca's own per-sector
setup cost, which is a cost of the design and not of the language.

**Since v0.7.0** (the table above is the v0.1.0 run), per the
[CHANGELOG](https://github.com/go-theca/theca/blob/v0.7.0/CHANGELOG.md):
Argon2id on the Apple M4 Max went from 1.54 to **0.73 of libsodium**, and
AES-256-CTR is 1.03x and the sector tag 1.04x OpenSSL there (#8 and #9 stay
open). On Zen 4 (the AMD Ryzen 9 7950X, v0.7.0): AES-256-CTR is **0.97 of
OpenSSL**, the sector tag 0.98, and Argon2id **0.25 of libsodium**. On amd64 the
Argon2id code is x/crypto's unchanged, and libsodium's 265 ms there was
confirmed by a C control program on the same core, so that row stands as a
measurement of libsodium on this machine. The control is recorded in the
[bench README on main](https://github.com/go-theca/theca/blob/main/bench/README.md):
a C program calling `crypto_pwhash` (Argon2id, t=3, m=64 MiB, one lane) pinned
to the same core gives 265 ms with the registry build and 265 to 271 ms with the
legacy pkgx build of libsodium 1.0.22 (2026-11-17). That README has no ratio
table; its XChaCha20 row has no such control yet. The M4 Max tables are in
`docs/perf-table.md` at that tag.

Rows of the v0.1.0 run that were slower, each tracked as an issue (the
changelog says #8 and #9 stay open after v0.7.0, and #10 is the Argon2id one):

- [#8](https://github.com/go-theca/theca/issues/8): Sector/AES-256-CTR is slower than OpenSSL.
- [#9](https://github.com/go-theca/theca/issues/9): Sector/Tag (HMAC-SHA-256) is slower than OpenSSL.
- [#10](https://github.com/go-theca/theca/issues/10): Argon2id is slower than libsodium on arm64 (no NEON kernel in x/crypto/argon2).

**Caveat on the Zen 4 libsodium rows.** On that machine libsodium's XChaCha20 and
Argon2id run far below their usual AVX2 speed (about 440 MB/s, and 4 times slower
than Go's Argon2id). The registry's libsodium may be built without its SIMD code.
Until a control with another libsodium build settles this, the "faster" verdicts
in those two rows are **not** claims.

Source: <https://github.com/go-theca/theca>.
