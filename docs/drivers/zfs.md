# zfs

<img src="../../assets/fs/go-filesystems-zfs.png" width="110" align="right">

Copy-on-write pooled storage filesystem.

Implements [`filesystem.Filesystem`](../interface.md). No cgo, no root.

## Status

| Read | Write | Format | Label | Symlinks | On-disk format |
|:--:|:--:|:--:|:--:|:--:|---|
| ✅ | ✅ | ✅ | — | — | Single pool / single vdev (test-oriented subset) |

## Supported

- Read/write basic file I/O on a ZPL dataset
- Create pool image (`Format`)
- Directories, rename
- Pool info (`Info`)
- Grow (`Resizer`) — shrink unsupported

## Not implemented

- Snapshots & clones not implemented
- Compression and data-block checksums not implemented
- Single vdev only

## Install

```bash
go get github.com/go-filesystems/zfs
```

- Source: <https://github.com/go-filesystems/zfs>
- API reference: <https://pkg.go.dev/github.com/go-filesystems/zfs>

!!! note
    See the module's README for full, up-to-date details.
