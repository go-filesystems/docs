---
title: "squashfs"
weight: 130
---

# squashfs {#squashfs}

<img src="../../assets/fs/go-filesystems-squashfs.png" width="110" align="right">

Compressed, read-only archive filesystem (SquashFS 4.0).

Implements [`filesystem.Filesystem`]({{< relref "../interface.md" >}}). No cgo, no root.

## Status {#status}

| Read | Write | Format | Label | Symlinks | On-disk format |
|:--:|:--:|:--:|:--:|:--:|---|
| ✅ | ✕ | ✅ | — | ✅ | SquashFS 4.0 read-only archive; gzip/xz/zstd/lzo/lz4 blocks + fragments |

## Supported {#supported}

- Superblock parse + validation (magic, version, block geometry)
- Metadata-block reader crossing 8 KiB block boundaries
- Inodes: basic + extended directory, regular file, symlink
- Multi-header directory listings
- File data: block list (compressed / uncompressed / sparse) + tail-end fragments
- Path resolution following intermediate and final symlinks
- Compression: `gzip` (zlib), `xz` (LZMA2), `zstd`, `lzo` (LZO1X), `lz4` (lzo and lz4 decode through the shared `go-compressions` decoders, v0.7.0), and uncompressed blocks
- **Image creation** — `BuildFromDir` writes a SquashFS 4.0 image from a tree (gzip or uncompressed), readable by `unsquashfs`

## Not implemented {#not-implemented}

- In-place writes — the archive is immutable once written; mutators return `ErrReadOnly`. Writer omits tail-end fragment packing, xattrs, and non-zero uid/gid.
- Legacy standalone `lzma` compression (`ErrUnsupportedCompression`); xz with BCJ filters
- Extended attributes (xattr table)

## Untrusted input {#untrusted-input}

Every input-driven allocation is bounded before it is made: `block_size` is at
most 1 MiB, a directory at most 4 Mi entries, and `ReadFile` refuses more than
`DefaultMaxReadFile` (1 GiB) with `ErrTooLarge`; `ReadFileLimit(path, max)`
picks another ceiling, and `OpenFile` + `ReadAt` stream any size.

## Parallel reads {#parallel-reads}

Since v0.8.0 the data blocks of a sequential read are decompressed in parallel
(`inflight.Ordered`: bounded, file order kept; `SetReadParallelism(n)`, default
`min(GOMAXPROCS, 8)`, 1 = serial). Sequential read of a 256 MiB text-like file
against the Linux kernel (7.0, 4 vCPU VM), MiB/s:

| compressor | kernel 1 CPU | ours 1 CPU | kernel 4 CPUs | ours 4 CPUs |
|---|---:|---:|---:|---:|
| xz | 48.3 | 39.5 | 47.7 | 146.0 |
| gzip | 283.6 | 231.1 | 261.2 | 718.8 |
| zstd | 492.1 | 457.4 | 514.6 | 1527.1 |
| lz4 | 1232.3 | 1909.3 | 977.4 | 3136.5 |
| lzo | 685.5 | 623.3 | 588.0 | 1649.3 |

Single-core xz, gzip, zstd and lzo are still behind the kernel.

## Random reads {#random-reads}

Since v0.9.0 the decoded-data cache adapts to random access. It starts at 64
blocks (8 MiB at 128 KiB) and grows, by 1/16 at a time, only when a block it
evicted is requested again: a random read of a file larger than the cache does,
one sequential pass does not. The growth stops at `DefaultMaxDataCacheBytes`
(256 MiB of decoded blocks); `(*FS).SetMaxDataCache(bytes)` sets another bound
at any time, and a cache already above it shrinks. Memory therefore reaches
256 MiB only for a workload that has shown it needs it.

20000 random 4 KiB reads over a 256 MiB file, against the Linux kernel (7.0,
4 vCPU VM), reads per second. Cold: caches dropped before the pass. Warm: the
same offsets again.

| compressor | cold: kernel | cold: v0.8.1 | cold: v0.9.0 | warm: kernel | warm: v0.9.0 |
|---|---:|---:|---:|---:|---:|
| xz | 3798 | 325 | 2951 | 1573429 | 3034088 |
| gzip | 21875 | 2030 | 16326 | 1513661 | 2853186 |
| zstd | 51286 | 6737 | 45869 | 1596586 | 2184032 |
| lzo | 60764 | 5242 | 52636 | 1676534 | 2978674 |
| lz4 | 84994 | 21881 | 109138 | 1566188 | 2675033 |

Before, the 8 MiB cache held 3 % of the file and 20000 reads decoded about 19400
blocks where the kernel's page cache decoded 2048. Cold, ours is now at 75 to
89 % of the kernel for xz, gzip, lzo and zstd and 128 % for lz4.

## Install {#install}

```bash
go get github.com/go-filesystems/squashfs
```

- Source: <https://github.com/go-filesystems/squashfs>
- API reference: <https://pkg.go.dev/github.com/go-filesystems/squashfs>

{{< callout type="info" >}}
See the module's README for full, up-to-date details.
{{< /callout >}}
