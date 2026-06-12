# btrfs

<img src="../../assets/fs/go-filesystems-btrfs.png" width="110" align="right">

Copy-on-write Linux fs with snapshots and subvolumes.

Implements [`filesystem.Filesystem`](../interface.md). No cgo, no root.

## Status

| Read | Write | Format | Label | Symlinks | On-disk format |
|:--:|:--:|:--:|:--:|:--:|---|
| ✅ | ✅ | ✅ | ✅ | ✅ | Single-device, CRC32c (btrfs-progs ≥ 5.x) |

## Supported

- Read/write files
- Directories, rename, symlinks
- MBR/GPT auto-detect

## Not implemented

- Snapshots, send/receive, multi-device/RAID, quotas and reflink are not implemented

## Install

```bash
go get github.com/go-filesystems/btrfs
```

- Source: <https://github.com/go-filesystems/btrfs>
- API reference: <https://pkg.go.dev/github.com/go-filesystems/btrfs>

!!! note
    See the module's README for full, up-to-date details.
