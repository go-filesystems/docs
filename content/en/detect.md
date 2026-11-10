---
title: "Detect"
weight: 25
---

# Detect {#detect}

[`go-filesystems/detect`](https://github.com/go-filesystems/detect) is a
pure-Go (no cgo, no libblkid) `blkid`-style prober and driver-opener registry.
It reads fixed, bounded windows of an image to identify the filesystem, then
dispatches to a registered driver. The core depends only on
[`go-filesystems/interface`](https://github.com/go-filesystems/interface) and
the standard library; drivers are wired in by blank-importing an adapter.

```go
import (
	"github.com/go-filesystems/detect"
	_ "github.com/go-filesystems/detect/fat32reg" // registers FAT12/16/32
)

t, err := detect.Detect(r, size)            // probe only, e.g. "fat32"
fs, t, err := detect.OpenFile("disk.img")   // probe and open
```

`Open` returns `ErrUnknown` when no signature matches and `ErrUnsupported`
when a type is detected but no driver is registered for it.

## Encrypted containers {#encrypted-containers}

A filesystem may sit inside a full-disk-encryption container. Containers are
wired in like drivers, so a program that never opens encrypted images links no
cryptography: the core contains none. The adapter module
`detect/fdereg` registers them with a blank import:

```go
import _ "github.com/go-filesystems/detect/fdereg"

fs, t, err := detect.OpenSecret(r, size, []byte("passphrase"))
fs, t, err := detect.OpenFileSecret("disk.img", secret)
fs, t, err := detect.OpenContainer(detect.LUKS, r, size, secret) // name the container
```

| Container | Backend | Notes |
|---|---|---|
| LUKS 1 and 2 | [go-fde/luks](https://github.com/go-fde/luks) | detected by its header |
| BitLocker | [go-fde/bitlocker](https://github.com/go-fde/bitlocker) | read-only; password, recovery password or `.BEK` |
| VeraCrypt / TrueCrypt | [go-fde/veracrypt](https://github.com/go-fde/veracrypt) | read-only; opened by name only, because its header is ciphertext |

- **Unlocking.** `OpenSecret` probes the registered containers first. On a
  match it unlocks the container and probes the plaintext for a filesystem.
  When nothing matches it opens the image as `Open` does.
- **Without a secret**, `Open` and `OpenFile` return `ErrEncrypted` wrapped
  with the container's name and do not probe the ciphertext (a BitLocker To Go
  volume starts like FAT32).
- **VeraCrypt** has no probe: `OpenContainer(detect.VeraCrypt, ...)` opens it
  by name.
- **FileVault 2** is not a container layer: APFS encrypts inside the
  filesystem and opens it with its own `OpenFDE`.

The containers' own documentation is at
[go-fde.github.io/docs](https://go-fde.github.io/docs/).
