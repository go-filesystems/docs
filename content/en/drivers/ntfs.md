---
title: "ntfs"
weight: 120
---

# ntfs {#ntfs}

<img src="../../assets/fs/go-filesystems-ntfs.png" width="110" align="right">

Windows NT filesystem.

Implements [`filesystem.Filesystem`]({{< relref "../interface.md" >}}). No cgo, no root.

{{< callout type="warning" >}}
Lightweight, test-oriented. Use a real NTFS implementation for on-disk compatibility.
{{< /callout >}}

## Status {#status}

| Read | Write | Format | Label | Symlinks | On-disk format |
|:--:|:--:|:--:|:--:|:--:|---|
| ✅ | ✅ | ✅ | ✅ | ✕ | Minimal in-image blob model — NOT the real NTFS on-disk format |

## Supported {#supported}

- Read/write files (blob storage inside the image)
- Directories, recursive delete, rename
- In-image free-list reuse
- In-image Grow / Shrink / Resize and compaction (`Compact`, `FragmentationStats`, `Layout`)
- A separate read-only reader parses genuine on-disk NTFS: LZNT1 decompression, named
  streams (ADS), `$REPARSE_POINT` symlinks/junctions, `$ATTRIBUTE_LIST`, volume label

## Not implemented {#not-implemented}

- The NTFSIMG1 in-image driver does not implement real NTFS structures (MFT, ACLs, journaling)
- The real on-disk reader is read-only and does not decode EFS-encrypted data
- `Grow`/`Shrink`/`Resize`/`Compact` operate on the NTFSIMG1 image only, not real on-disk NTFS

## Install {#install}

```bash
go get github.com/go-filesystems/ntfs
```

- Source: <https://github.com/go-filesystems/ntfs>
- API reference: <https://pkg.go.dev/github.com/go-filesystems/ntfs>

{{< callout type="info" >}}
See the module's README for full, up-to-date details.
{{< /callout >}}
