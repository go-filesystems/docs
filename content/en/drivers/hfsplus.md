---
title: "hfsplus"
weight: 100
---

# hfsplus {#hfsplus}

<img src="../../assets/fs/go-filesystems-hfsplus.png" width="110" align="right">

Apple HFS+ (Mac OS Extended) and its HFSX (case-sensitive) variant.

Implements [`filesystem.Filesystem`]({{< relref "../interface.md" >}}). No cgo, no root.

## Status {#status}

| Read | Write | Format | Label | Symlinks | On-disk format |
|:--:|:--:|:--:|:--:|:--:|---|
| ✅ | ✅ | ✅ | ✅ | ✅ | HFS+ / HFSX, big-endian catalog + extents-overflow B-trees |

## Supported {#supported}

- Read/write files
- Directories, rename
- `SetLabel` / `Symlink` / `Truncate` (optional `Labeller` / `Symlinker` / `Truncater` capabilities)
- MBR/GPT auto-detect
- Times (`filesystem.TimesReader`, v0.7.0): creation, content modification, attribute
  modification (Mac OS X's change time) and access from the catalog record, to 1 s,
  GMT seconds since 1904-01-01 (Apple TN1150). A date of 0 is "not recorded", and so is
  the root folder's. Checked against an image written by macOS `newfs_hfs`

## Journaled volumes {#journal}

Since v0.9.0 the write path knows about the HFS+ journal (Apple TN1150). It
does not write transactions itself, and macOS replays a journal that holds some
over whatever was written around it, so:

- A **journaled volume with an empty journal** (start == end, what a clean
  unmount leaves) is written directly. The journal info block, the journal and
  the journaled attribute are left as they were, so macOS replays nothing at
  the next mount.
- A **journal with transactions in it**, or one that cannot be verified (a bad
  checksum, a journal outside the volume or on another device), is refused by
  `OpenWritable` and `OpenFileWritable` with the typed error `ErrJournaled`.
  Nothing is written, and `OpenFileWritable` closes the file. Test it with
  `errors.Is(err, hfsplus.ErrJournaled)`.
- Read-only `Open` is unchanged: it reads a journaled volume whatever its
  journal holds. A caller that gets `ErrJournaled` can fall back to it.
- Volumes made by `Format` and `Mkfs` are not journaled.

{{< callout type="warning" >}}
**Do not write an image that is mounted.** Neither the driver nor the system
that mounted it knows about the other's changes. Write an image nothing else
has open.
{{< /callout >}}

## Untrusted input {#untrusted-input}

Every size in an image is a claim. A fork's logical size is bounded by the image
size and every extent it uses must lie inside the image before the buffer is
made; with the size unknown the buffer grows with the bytes the reader delivers.
The B-tree leaf-chain walk is bounded by image size / node size.

## Install {#install}

```bash
go get github.com/go-filesystems/hfsplus
```

- Source: <https://github.com/go-filesystems/hfsplus>
- API reference: <https://pkg.go.dev/github.com/go-filesystems/hfsplus>

{{< callout type="info" >}}
See the module's README for full, up-to-date details.
{{< /callout >}}
