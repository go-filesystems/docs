---
title: "ufs"
weight: 150
---

# ufs {#ufs}

<img src="../../assets/fs/go-filesystems-ufs.png" width="110" align="right">

Unix File System (BSD).

Implements [`filesystem.Filesystem`]({{< relref "../interface.md" >}}). No cgo, no root.

## Status {#status}

| Read | Write | Format | Label | Symlinks | On-disk format |
|:--:|:--:|:--:|:--:|:--:|---|
| ✅ | ✅ | ✅ | — | ✅ | UFS2 (FreeBSD 14.x) read+write; UFS1 read |

## Supported {#supported}

- Read + write (`OpenRW`) and fresh format (`Mkfs`)
- Superblock/inode decode, directory walk
- Direct + single/double indirect blocks
- Inline & spilled symlinks, sparse holes

## Not implemented {#not-implemented}

- Triple-indirect blocks unsupported (`ErrUnsupportedIndirect`)

## Untrusted input {#untrusted-input}

A file is read block by block into a buffer that grows with what was read. Since
a sparse file legitimately reads back bigger than the image, a read has a
ceiling: `DefaultMaxFileBytes` (1 GiB), changed with `(*FS).SetMaxFileBytes`. A
read over it fails with an error wrapping `ufs.ErrTooLarge`, before anything is
allocated.

## Install {#install}

```bash
go get github.com/go-filesystems/ufs
```

- Source: <https://github.com/go-filesystems/ufs>
- API reference: <https://pkg.go.dev/github.com/go-filesystems/ufs>

{{< callout type="info" >}}
See the module's README for full, up-to-date details.
{{< /callout >}}
