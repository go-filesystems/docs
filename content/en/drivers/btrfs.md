---
title: "btrfs"
weight: 50
---

# btrfs {#btrfs}

<img src="../../assets/fs/go-filesystems-btrfs.png" width="110" align="right">

Copy-on-write Linux fs with snapshots and subvolumes.

Implements [`filesystem.Filesystem`]({{< relref "../interface.md" >}}). No cgo, no root.

## Status {#status}

| Read | Write | Format | Label | Symlinks | On-disk format |
|:--:|:--:|:--:|:--:|:--:|---|
| ✅ | ✅ | ✅ | ✅ | ✅ | Single-device, CRC32c (btrfs-progs ≥ 5.x) |

## Supported {#supported}

- Read/write files
- Directories, rename, symlinks
- MBR/GPT auto-detect

## Not implemented {#not-implemented}

- Snapshots, send/receive, multi-device/RAID, quotas and reflink are not implemented

## Untrusted input {#untrusted-input}

Every allocation, walk and recursion sized from a value on the image is checked
against what the format can produce before it happens: a compressed extent is at
most 128 KiB (the kernel's `BTRFS_MAX_UNCOMPRESSED`), a zstd window 128 KiB,
overlapping file extent items are refused, and a chunk `stripe_len` must be
exactly 64 KiB.

## Install {#install}

```bash
go get github.com/go-filesystems/btrfs
```

- Source: <https://github.com/go-filesystems/btrfs>
- API reference: <https://pkg.go.dev/github.com/go-filesystems/btrfs>

{{< callout type="info" >}}
See the module's README for full, up-to-date details.
{{< /callout >}}
