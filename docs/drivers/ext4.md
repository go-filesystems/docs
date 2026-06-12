# ext4

<img src="../../assets/fs/go-filesystems-ext4.png" width="110" align="right">

Linux ext4 — extents, 64-bit, journaling, metadata_csum.

Implements [`filesystem.Filesystem`](../interface.md). No cgo, no root.

## Status

| Read | Write | Format | Label | Symlinks | On-disk format |
|:--:|:--:|:--:|:--:|:--:|---|
| ✅ | ✅ | ✅ | ✅ | ✅ | ext4 — extents, 64-bit, flex_bg, dir htree, metadata_csum (CRC32c) |

## Supported

- Read/write files (extents, sparse writes)
- Directories, rename, symlinks
- metadata_csum (CRC32c) writes
- Online grow (`Grow`)
- `CheckImage` / `RepairImage` hooks
- `GetFlags`/`SetFlags` ioctls
- MBR/GPT auto-detect

## Not implemented

- Some cross-check tests require `mke2fs`
- Not every kernel ioctl / obscure feature is implemented

## Install

```bash
go get github.com/go-filesystems/ext4
```

- Source: <https://github.com/go-filesystems/ext4>
- API reference: <https://pkg.go.dev/github.com/go-filesystems/ext4>

!!! note
    See the module's README for full, up-to-date details.
