# xfs

<img src="../../assets/fs/go-filesystems-xfs.png" width="110" align="right">

High-performance journaling filesystem.

Implements [`filesystem.Filesystem`](../interface.md). No cgo, no root.

## Status

| Read | Write | Format | Label | Symlinks | On-disk format |
|:--:|:--:|:--:|:--:|:--:|---|
| ✅ | ✅ | ✅ | ✅ | ✅ | XFS v5 (CRC32c, ftype) |

## Supported

- Read/write files
- Directories, rename, symlinks
- MBR/GPT auto-detect
- Volume label (`Labeller`)
- Grow / Resize (whole-AG and partial last-AG growth) — shrink returns `ErrShrinkUnsupported`
- Reflink / copy-on-write clones (`FormatConfig.Reflink`, `FS.Reflink`)
- Quotas — user / group / project (`FormatConfig.Quota`), kept consistent by an inode-scan quotacheck
- Extended metadata: `ExtendedStat`, `Chown`, `Chmod`, `Chtimes`, `Truncate`, `Link` (hardlinks)
- Layered backends via `OpenFromDevice` (LUKS / qcow2 / in-memory)

## Install

```bash
go get github.com/go-filesystems/xfs
```

- Source: <https://github.com/go-filesystems/xfs>
- API reference: <https://pkg.go.dev/github.com/go-filesystems/xfs>

!!! note
    See the module's README for full, up-to-date details.
