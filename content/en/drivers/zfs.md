---
title: "zfs"
weight: 170
---

# zfs {#zfs}

<img src="../../assets/fs/go-filesystems-zfs.png" width="110" align="right">

Copy-on-write pooled storage filesystem.

Implements [`filesystem.Filesystem`]({{< relref "../interface.md" >}}). No cgo, no root.

## Status {#status}

| Read | Write | Format | Label | Symlinks | On-disk format |
|:--:|:--:|:--:|:--:|:--:|---|
| ✅ | ✅ | ✅ | — | ✅ | Single pool, single-vdev writer (test-oriented subset); multi-vdev/RAID-Z read via `OpenFromDevices` |

## Supported {#supported}

- Read/write basic file I/O on a ZPL dataset
- Create pool image (`Format`)
- Directories, rename
- Pool info (`Info`)
- Grow / Shrink / Resize, including `ShrinkWithMode` (Rebuild / InPlace / Auto)
- Snapshots (`Snapshot`, `OpenSnapshot`, `DestroySnapshot`) and clones (`Clone`, `Origin`)
- Transparent decompress on read (lz4 / gzip / zstd / lzjb / zle)
- Native encryption read (`OpenFromDeviceDatasetWithKey`, AES-CCM/GCM)
- Symlinks and hardlinks; chmod/chown/chtimes (type-assert the optional interfaces)
- Multi-vdev (mirror / RAID-Z) pools opened read via `OpenFromDevices`

## Not implemented {#not-implemented}

- The writer only targets a single dataset in a single-vdev pool (the reader
  navigates nested datasets and multi-vdev/RAID-Z pools)
- The writer does not compress or encrypt data blocks it produces
- Snapshots/clones are eager block copies, not O(1) copy-on-write; no ACLs
- A missing RAID-Z data leg is not yet reconstructed from parity

## Untrusted input {#untrusted-input}

Every allocation and walk sized from a value on the image is checked against
what the image can hold before it happens, with bounds taken from the OpenZFS
sources: block pointer sizes are at most `SPA_MAXBLOCKSIZE` (16 MiB), and
`ReadFile` / `ReadLink` are capped at `DefaultMaxReadSize` (1 GiB, changed with
`SetMaxReadSize`) and refused with `ErrTooLarge` before any buffer is sized.

## Install {#install}

```bash
go get github.com/go-filesystems/zfs
```

- Source: <https://github.com/go-filesystems/zfs>
- API reference: <https://pkg.go.dev/github.com/go-filesystems/zfs>

{{< callout type="info" >}}
See the module's README for full, up-to-date details.
{{< /callout >}}
