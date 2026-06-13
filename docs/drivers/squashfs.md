# squashfs

<img src="../../assets/fs/go-filesystems-squashfs.png" width="110" align="right">

Compressed, read-only archive filesystem (SquashFS 4.0).

Implements [`filesystem.Filesystem`](../interface.md). No cgo, no root.

## Status

| Read | Write | Format | Label | Symlinks | On-disk format |
|:--:|:--:|:--:|:--:|:--:|---|
| ✅ | ✕ | ✅ | — | ✅ | SquashFS 4.0 read-only archive; gzip/xz/zstd/lzo/lz4 blocks + fragments |

## Supported

- Superblock parse + validation (magic, version, block geometry)
- Metadata-block reader crossing 8 KiB block boundaries
- Inodes: basic + extended directory, regular file, symlink
- Multi-header directory listings
- File data: block list (compressed / uncompressed / sparse) + tail-end fragments
- Path resolution following intermediate and final symlinks
- Compression: `gzip` (zlib), `xz` (LZMA2), `zstd`, `lzo` (LZO1X), `lz4`, and uncompressed blocks
- **Image creation** — `BuildFromDir` writes a SquashFS 4.0 image from a tree (gzip or uncompressed), readable by `unsquashfs`

## Not implemented

- In-place writes — the archive is immutable once written; mutators return `ErrReadOnly`. Writer omits tail-end fragment packing, xattrs, and non-zero uid/gid.
- Legacy standalone `lzma` compression (`ErrUnsupportedCompression`); xz with BCJ filters
- Extended attributes (xattr table)

## Install

```bash
go get github.com/go-filesystems/squashfs
```

- Source: <https://github.com/go-filesystems/squashfs>
- API reference: <https://pkg.go.dev/github.com/go-filesystems/squashfs>

!!! note
    See the module's README for full, up-to-date details.
