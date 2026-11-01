---
title: "fat32"
weight: 80
---

# fat32 {#fat32}

<img src="../../assets/fs/go-filesystems-fat32.png" width="110" align="right">

FAT with 32-bit allocation. Since v0.6.0 it also **reads FAT12 and FAT16**
volumes: what `newfs_msdos -F 12|16` and `mkfs.vfat` produce below about 33 MiB.

Implements [`filesystem.Filesystem`]({{< relref "../interface.md" >}}). No cgo, no root.

## Status {#status}

| Read | Write | Format | Label | Symlinks | On-disk format |
|:--:|:--:|:--:|:--:|:--:|---|
| ✅ | ✅ | ✅ | ✅ | ✕ | FAT32 (read/write); FAT12, FAT16 (read-only) |

## FAT12 and FAT16 {#fat12-16}

The variant is decided by the count of data clusters, as the Microsoft FAT
specification says (below 4085 is FAT12, below 65525 FAT16), never by the
`FAT12`/`FAT16` label. The root directory of these two is a fixed region after
the FATs, and a FAT12 entry is 12 bits packed, so one can straddle a sector
boundary. Both are handled; `Info.Type` says which one a volume is.

**They are read-only in v0.6.0**: every write returns `ErrReadOnlyFAT` and
leaves the image untouched. Together with
[`detect`](https://github.com/go-filesystems/detect) v0.4.0, which now reports `fat12` and
`fat16`, such an image opens exactly like a FAT32 one.

## Supported {#supported}

- Read/write files
- Directories, recursive delete, rename
- Bare images + MBR/GPT
- Volume label (`Labeller`)
- Resize (`filesystem.Resizer`)
- fsck-clean images (v0.5.1): `fsck_msdos -n` and `fsck.vfat -n` exit 0 on what
  the driver writes. The `..` entry of a directory under the root holds
  cluster 0, and the FSInfo free count and next-free hint are exact at format
  and kept current while the image is written
- Format to a path (`Format`) or into any `io.ReaderAt` + `io.WriterAt`, such as
  a buffer in memory (`FormatImage`) — what the
  [playground]({{< relref "../playground.md" >}}) uses to make its sample image
  in the browser

## Not implemented {#not-implemented}

- No POSIX symlinks (not in FAT32)
- Writing to a FAT12 or FAT16 volume (read-only for now)

## Untrusted input {#untrusted-input}

Every allocation a value in the image can size is bounded before it happens: a
cluster buffer is allocated only after the cluster's last byte has been read, the
sector size is 512..4096, and a directory is at most 2 MiB. A hostile image gets
an error, not an out-of-memory kill.

## Install {#install}

```bash
go get github.com/go-filesystems/fat32
```

- Source: <https://github.com/go-filesystems/fat32>
- API reference: <https://pkg.go.dev/github.com/go-filesystems/fat32>

{{< callout type="info" >}}
See the module's README for full, up-to-date details.
{{< /callout >}}
