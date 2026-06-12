# ntfs

<img src="../../assets/fs/go-filesystems-ntfs.png" width="110" align="right">

Windows NT filesystem.

Implements [`filesystem.Filesystem`](../interface.md). No cgo, no root.

!!! warning
    Lightweight, test-oriented. Use a real NTFS implementation for on-disk compatibility.

## Status

| Read | Write | Format | Symlinks | On-disk format |
|:--:|:--:|:--:|:--:|---|
| ✅ | ✅ | ✅ | ✕ | Minimal in-image blob model — NOT the real NTFS on-disk format |

## Supported

- Read/write files (blob storage inside the image)
- Directories, recursive delete, rename
- In-image free-list reuse

## Not implemented

- Does not implement real NTFS structures (MFT, ACLs, ADS, journaling)
- No storage compaction

## Install

```bash
go get github.com/go-filesystems/ntfs
```

- Source: <https://github.com/go-filesystems/ntfs>
- API reference: <https://pkg.go.dev/github.com/go-filesystems/ntfs>

!!! note
    See the module's README for full, up-to-date details.
