# uefi

<img src="../../assets/fs/go-filesystems-uefi.png" width="110" align="right">

EFI System Partition (FAT-based).

Implements [`filesystem.Filesystem`](../interface.md). No cgo, no root.

!!! warning
    A UEFI variable store, not a POSIX filesystem.

## Status

| Read | Write | Format | Symlinks | On-disk format |
|:--:|:--:|:--:|:--:|---|
| ✅ | ✅ | — | — | OVMF/EDK2 NvVar variable store (non-authenticated) |

## Supported

- List / Get / Set / Delete UEFI variables (atomic rewrite)
- Secure Boot key enrolment (PK, KEK, db, dbx)
- Exposed via `filesystem.Filesystem` (variable = file)

## Not implemented

- Time-based authenticated writes not supported
- `MkDir` / `DeleteDir` / `ReadLink` return "not supported" (a store, not a tree)

## Install

```bash
go get github.com/go-filesystems/uefi
```

- Source: <https://github.com/go-filesystems/uefi>
- API reference: <https://pkg.go.dev/github.com/go-filesystems/uefi>

!!! note
    See the module's README for full, up-to-date details.
