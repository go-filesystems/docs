# ffs

<img src="../../assets/fs/go-filesystems-ffs.png" width="110" align="right">

NetBSD/OpenBSD Berkeley Fast File System (FFS) — a thin re-export of
[`ufs`](ufs.md), since FFSv1/FFSv2 are the same on-disk format as UFS1/UFS2.

Implements [`filesystem.Filesystem`](../interface.md). No cgo, no root.

## Status

| Read | Write | Format | Label | Symlinks | On-disk format |
|:--:|:--:|:--:|:--:|:--:|---|
| ✅ | ✅ | ✅ | — | ✅ | NetBSD/OpenBSD FFSv1/FFSv2 (= UFS1/UFS2), via the `ufs` driver |

## Supported

Everything the `ufs` driver provides, under an import path that matches how
NetBSD/OpenBSD callers think of the format (`ffs.OpenFile`, `ffs.Open`,
`ffs.OpenRW` — thin aliases of the `ufs` entry points).

## Install

```bash
go get github.com/go-filesystems/ffs
```

- Source: <https://github.com/go-filesystems/ffs>
- API reference: <https://pkg.go.dev/github.com/go-filesystems/ffs>

!!! note
    See [`ufs`](ufs.md) and the module's README for full, up-to-date details.
