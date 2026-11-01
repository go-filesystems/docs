---
title: "xfs"
weight: 160
---

# xfs {#xfs}

<img src="../../assets/fs/go-filesystems-xfs.png" width="110" align="right">

High-performance journaling filesystem.

Implements [`filesystem.Filesystem`]({{< relref "../interface.md" >}}). No cgo, no root.

## Status {#status}

| Read | Write | Format | Label | Symlinks | On-disk format |
|:--:|:--:|:--:|:--:|:--:|---|
| ✅ | ✅ | ✅ | ✅ | ✅ | XFS v5 (CRC32c, ftype) |

## Supported {#supported}

- Read/write files
- Directories, rename, symlinks
- MBR/GPT auto-detect
- Volume label (`Labeller`)
- Grow / Resize (whole-AG and partial last-AG growth) — shrink returns `ErrShrinkUnsupported`
- Reflink / copy-on-write clones (`FormatConfig.Reflink`, `FS.Reflink`)
- Quotas — user / group / project (`FormatConfig.Quota`), kept consistent by an inode-scan quotacheck
- Extended metadata: `ExtendedStat`, `Chown`, `Chmod`, `Chtimes`, `Truncate`, `Link` (hardlinks)
- Layered backends via `OpenFromDevice` (LUKS / qcow2 / in-memory)

## Untrusted input {#untrusted-input}

Every allocation and walk sized from a value on the image is checked against
what the image can hold before it happens. `ReadFile` and remote symlinks are
capped at `DefaultMaxReadSize` (1 GiB, changed with `SetMaxReadSize`) and refused
with `ErrTooLarge`; bmbt extent records that overlap are refused.

## Install {#install}

```bash
go get github.com/go-filesystems/xfs
```

- Source: <https://github.com/go-filesystems/xfs>
- API reference: <https://pkg.go.dev/github.com/go-filesystems/xfs>

{{< callout type="info" >}}
See the module's README for full, up-to-date details.
{{< /callout >}}
