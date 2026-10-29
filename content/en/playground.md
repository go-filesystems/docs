---
title: "Playground"
weight: 15
---

# Playground {#playground}

**<https://go-filesystems.github.io/playground/>** — open a disk image in your
browser. The page says which filesystem it is, lets you browse the tree, view
small files and download any file. Everything runs in WebAssembly in your
browser: **nothing is uploaded**.

No image at hand? **Make a sample FAT32 image**: the page formats one in memory
with `fat32.FormatImage`, writes a few files into it, opens it again through
`detect`, and lets you download it.

## What it opens {#what-it-opens}

| Filesystem | Found | Viewing and downloading |
|---|---|---|
| [fat32]({{< relref "drivers/fat32.md" >}}), [exfat]({{< relref "drivers/exfat.md" >}}), [ext4]({{< relref "drivers/ext4.md" >}}), [iso9660]({{< relref "drivers/iso9660.md" >}}), [squashfs]({{< relref "drivers/squashfs.md" >}}) | at offset 0, by [`detect`](https://github.com/go-filesystems/detect) | streamed in 4 MiB pieces (`filesystem.Opener`) |
| [ntfs]({{< relref "drivers/ntfs.md" >}}), [hfsplus]({{< relref "drivers/hfsplus.md" >}}), [ufs]({{< relref "drivers/ufs.md" >}}) | at offset 0, by `detect` | read whole, files up to 64 MiB |
| [apfs]({{< relref "drivers/apfs.md" >}}), [btrfs]({{< relref "drivers/btrfs.md" >}}), [xfs]({{< relref "drivers/xfs.md" >}}), [zfs]({{< relref "drivers/zfs.md" >}}) | in a disk image's GPT or MBR table, through a read-only block device | read whole, files up to 64 MiB |

**The Filesystem list** chooses the driver. *Detect it*, the default, asks
`detect`; when nothing opens at offset 0, the four disk-image drivers are tried in
turn. Choosing a filesystem forces that driver, for an image `detect` gets wrong
or cannot recognise.

### Encrypted images {#encrypted}

A **LUKS1 or LUKS2** container ([go-fde/luks](https://github.com/go-fde/luks)) is
recognised by its header, and the page asks for its password. The engine unlocks
it and opens the filesystem in the payload: detected, or the one chosen in the
list. The facts then read e.g. `ext4 in LUKS`. A wrong password is refused, and
the form stays to try again.

APFS FileVault containers are not supported yet: go-filesystems/apfs cannot read
one through its decrypted device so far, and that is being fixed there first.

### Open in the explorer {#explorer}

Once an image is open, **open in the explorer** loads the
[go-fileshare explorer](https://github.com/go-fileshare/explorer) in a same-origin
frame. It is fetched only then (4.1 MB compressed), and browses the image the way it
browses a fileshare server, over WebDAV. None of its requests reach a network. The
page hands each one to the engine, which answers it with
[go-filesystems/webdav](https://github.com/go-filesystems/webdav) over the open
image, read-only. Closing the explorer unloads it.

## How it works {#how-it-works}

- The Go engine runs in a **Web Worker**. Its `io.ReaderAt` reads the dropped
  file with `FileReaderSync.readAsArrayBuffer(file.slice(off, end))` — a
  synchronous read that the [File API](https://w3c.github.io/FileAPI/) allows only
  in workers — so the drivers' synchronous reads never block the page, and the
  image is never loaded whole.
- A **block cache** (64 KiB blocks, LRU, 16 MiB) turns a driver's many small
  metadata reads into a few block reads; file contents bypass it. A checkbox on
  the page turns it off for comparison.
- The page is plain HTML, CSS and a few KiB of JavaScript, talking to the worker
  with three coarse requests: open, list a directory, read part of a file.

## Safety {#safety}

The image is untrusted input: every answer is bounded (entries per listing, bytes
per read), a driver panic becomes an error, a request that takes more than 30 s
restarts the engine, and names from the image are shown as text, never as HTML.

A LUKS header is untrusted too: it says how much key derivation unlocking costs.
The engine refuses a header that asks for more than 2²² PBKDF2 iterations, or for
Argon2 beyond 16 passes or 1 GiB, before deriving anything (go-fde/luks
`OpenFromLimited`). An unlock gets 180 s.

The password goes to the engine in the tab for one open. It is not stored and
never sent anywhere. The page empties the field, and the engine clears its copy
once the container is unlocked or refused.

Source: <https://github.com/go-filesystems/playground>
