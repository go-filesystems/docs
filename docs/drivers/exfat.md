# exfat

<img src="../../assets/fs/go-filesystems-exfat.png" width="110" align="right">

Extended FAT for large media.

Implements [`filesystem.Filesystem`](../interface.md). No cgo, no root.

## Status

| Read | Write | Format | Label | Symlinks | On-disk format |
|:--:|:--:|:--:|:--:|:--:|---|
| ✅ | ✅ | ✅ | ✅ | ✕ | exFAT |

## Supported

- Read/write files
- Directories, recursive delete, rename
- Bare images + MBR/GPT

## Not implemented

- No POSIX symlinks, permissions or ACLs (not in exFAT)

## Install

```bash
go get github.com/go-filesystems/exfat
```

- Source: <https://github.com/go-filesystems/exfat>
- API reference: <https://pkg.go.dev/github.com/go-filesystems/exfat>

!!! note
    See the module's README for full, up-to-date details.
