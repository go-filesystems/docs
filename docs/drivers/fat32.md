# fat32

<img src="../../assets/fs/go-filesystems-fat32.png" width="110" align="right">

FAT with 32-bit allocation.

Implements [`filesystem.Filesystem`](../interface.md). No cgo, no root.

## Status

| Read | Write | Format | Label | Symlinks | On-disk format |
|:--:|:--:|:--:|:--:|:--:|---|
| ✅ | ✅ | ✅ | ✅ | ✕ | FAT32 |

## Supported

- Read/write files
- Directories, recursive delete, rename
- Bare images + MBR/GPT
- Volume label (`Labeller`)
- Resize (`filesystem.Resizer`)

## Not implemented

- No POSIX symlinks (not in FAT32)

## Install

```bash
go get github.com/go-filesystems/fat32
```

- Source: <https://github.com/go-filesystems/fat32>
- API reference: <https://pkg.go.dev/github.com/go-filesystems/fat32>

!!! note
    See the module's README for full, up-to-date details.
