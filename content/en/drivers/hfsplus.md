---
title: "hfsplus"
weight: 100
---

# hfsplus {#hfsplus}

<img src="../../assets/fs/go-filesystems-hfsplus.png" width="110" align="right">

Apple HFS+ (Mac OS Extended) and its HFSX (case-sensitive) variant.

Implements [`filesystem.Filesystem`]({{< relref "../interface.md" >}}). No cgo, no root.

## Status {#status}

| Read | Write | Format | Label | Symlinks | On-disk format |
|:--:|:--:|:--:|:--:|:--:|---|
| ✅ | ✅ | ✅ | ✅ | ✅ | HFS+ / HFSX, big-endian catalog + extents-overflow B-trees |

## Supported {#supported}

- Read/write files
- Directories, rename
- `SetLabel` / `Symlink` / `Truncate` (optional `Labeller` / `Symlinker` / `Truncater` capabilities)
- MBR/GPT auto-detect

## Install {#install}

```bash
go get github.com/go-filesystems/hfsplus
```

- Source: <https://github.com/go-filesystems/hfsplus>
- API reference: <https://pkg.go.dev/github.com/go-filesystems/hfsplus>

{{< callout type="info" >}}
See the module's README for full, up-to-date details.
{{< /callout >}}
