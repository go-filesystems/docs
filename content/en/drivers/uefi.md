---
title: "uefi"
weight: 140
---

# uefi {#uefi}

<img src="../../assets/fs/go-filesystems-uefi.png" width="110" align="right">

EFI System Partition (FAT-based).

Implements [`filesystem.Filesystem`]({{< relref "../interface.md" >}}). No cgo, no root.

{{< callout type="warning" >}}
A UEFI variable store, not a POSIX filesystem.
{{< /callout >}}

## Status {#status}

| Read | Write | Format | Label | Symlinks | On-disk format |
|:--:|:--:|:--:|:--:|:--:|---|
| ✅ | ✅ | ✅ | — | — | OVMF/EDK2 NvVar variable store (non-authenticated) |

## Supported {#supported}

- Create a fresh store (`Format` / `FormatOVMF`)
- List / Get / Set / Delete UEFI variables (atomic rewrite)
- Secure Boot key enrolment (PK, KEK, db, dbx)
- Exposed via `filesystem.Filesystem` (variable = file)

## Not implemented {#not-implemented}

- Time-based authenticated writes not supported
- `MkDir` / `DeleteDir` / `ReadLink` return "not supported" (a store, not a tree)

## Untrusted input {#untrusted-input}

A store file is bounded before it is read: `OpenLimits` takes `MaxFileSize`
(default `DefaultMaxFileSize`, 256 MiB) and `MaxVariables` (default 65 536), and
an excess returns an error wrapping `safeio.ErrTooLarge`.

## Install {#install}

```bash
go get github.com/go-filesystems/uefi
```

- Source: <https://github.com/go-filesystems/uefi>
- API reference: <https://pkg.go.dev/github.com/go-filesystems/uefi>

{{< callout type="info" >}}
See the module's README for full, up-to-date details.
{{< /callout >}}
