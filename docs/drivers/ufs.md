# ufs

<img src="../../assets/fs/go-filesystems-ufs.png" width="110" align="right">

Unix File System (BSD).

Implements [`filesystem.Filesystem`](../interface.md). No cgo, no root.

## Status

| Read | Write | Format | Label | Symlinks | On-disk format |
|:--:|:--:|:--:|:--:|:--:|---|
| ✅ | ✅ | ✅ | — | ✅ | UFS2 (FreeBSD 14.x); UFS1 not supported |

## Supported

- Read + write (`OpenRW`) and fresh format (`Mkfs`)
- Superblock/inode decode, directory walk
- Direct + single/double indirect blocks
- Inline & spilled symlinks, sparse holes

## Not implemented

- Triple-indirect blocks unsupported (`ErrUnsupportedIndirect`)

## Install

```bash
go get github.com/go-filesystems/ufs
```

- Source: <https://github.com/go-filesystems/ufs>
- API reference: <https://pkg.go.dev/github.com/go-filesystems/ufs>

!!! note
    See the module's README for full, up-to-date details.
