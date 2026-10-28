---
title: "fat32"
weight: 80
---

# fat32 {#fat32}

<img src="../../assets/fs/go-filesystems-fat32.png" width="110" align="right">

FAT with 32-bit allocation.

Implements [`filesystem.Filesystem`]({{< relref "../interface.md" >}}). No cgo, no root.

## Status {#status}

| Read | Write | Format | Label | Symlinks | On-disk format |
|:--:|:--:|:--:|:--:|:--:|---|
| ✅ | ✅ | ✅ | ✅ | ✕ | FAT32 |

## Supported {#supported}

- Read/write files
- Directories, recursive delete, rename
- Bare images + MBR/GPT
- Volume label (`Labeller`)
- Resize (`filesystem.Resizer`)
- Format to a path (`Format`) or into any `io.ReaderAt` + `io.WriterAt`, such as
  a buffer in memory (`FormatImage`) — what the
  [playground]({{< relref "../playground.md" >}}) uses to make its sample image
  in the browser

## Not implemented {#not-implemented}

- No POSIX symlinks (not in FAT32)

## Install {#install}

```bash
go get github.com/go-filesystems/fat32
```

- Source: <https://github.com/go-filesystems/fat32>
- API reference: <https://pkg.go.dev/github.com/go-filesystems/fat32>

{{< callout type="info" >}}
See the module's README for full, up-to-date details.
{{< /callout >}}
