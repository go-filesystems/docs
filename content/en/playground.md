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

A filesystem at offset 0 of the file — a filesystem image, not a partitioned
disk — detected by [`detect`](https://github.com/go-filesystems/detect) and opened
by the drivers that take an `io.ReaderAt`:

| Filesystem | Viewing and downloading |
|---|---|
| [fat32]({{< relref "drivers/fat32.md" >}}), [exfat]({{< relref "drivers/exfat.md" >}}), [ext4]({{< relref "drivers/ext4.md" >}}), [iso9660]({{< relref "drivers/iso9660.md" >}}), [squashfs]({{< relref "drivers/squashfs.md" >}}) | streamed in 4 MiB pieces (`filesystem.Opener`) |
| [ntfs]({{< relref "drivers/ntfs.md" >}}), [hfsplus]({{< relref "drivers/hfsplus.md" >}}), [ufs]({{< relref "drivers/ufs.md" >}}) | read whole, files up to 64 MiB |

apfs, btrfs, xfs and zfs open a partition of a disk image by path rather than an
`io.ReaderAt`, so they are not in the playground yet.

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

Source: <https://github.com/go-filesystems/playground>
