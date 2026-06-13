# squashfs

<img src="../../assets/fs/go-filesystems-squashfs.png" width="110" align="right">

Compressed, read-only archive filesystem (SquashFS 4.0).

Implements [`filesystem.Filesystem`](../interface.md). No cgo, no root.

## Status

| Read | Write | Format | Label | Symlinks | On-disk format |
|:--:|:--:|:--:|:--:|:--:|---|
| ✅ | ✕ | ✕ | — | ✅ | SquashFS 4.0 — read-only archive; gzip blocks + fragments |

## Supported

- Superblock parse + validation (magic, version, block geometry)
- Metadata-block reader crossing 8 KiB block boundaries
- Inodes: basic + extended directory, regular file, symlink
- Multi-header directory listings
- File data: block list (compressed / uncompressed / sparse) + tail-end fragments
- Path resolution following intermediate and final symlinks
- `gzip` (zlib-stream) and uncompressed blocks

## Not implemented

- Write/format — SquashFS is a read-only format; mutators return `ErrReadOnly`
- `xz` / `lz4` / `zstd` / `lzo` / `lzma` compression (`ErrUnsupportedCompression`)
- Extended attributes (xattr table)

## Install

```bash
go get github.com/go-filesystems/squashfs
```

- Source: <https://github.com/go-filesystems/squashfs>
- API reference: <https://pkg.go.dev/github.com/go-filesystems/squashfs>

!!! note
    See the module's README for full, up-to-date details.
