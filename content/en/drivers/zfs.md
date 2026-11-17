---
title: "zfs"
weight: 170
---

# zfs {#zfs}

<img src="../../assets/fs/go-filesystems-zfs.png" width="110" align="right">

Copy-on-write pooled storage filesystem.

Implements [`filesystem.Filesystem`]({{< relref "../interface.md" >}}). No cgo, no root.

## Status {#status}

| Read | Write | Format | Label | Symlinks | On-disk format |
|:--:|:--:|:--:|:--:|:--:|---|
| ✅ | ✅ | ✅ | — | ✅ | Single pool, single-vdev writer (test-oriented subset); multi-vdev/RAID-Z read via `OpenFromDevices` |

## Supported {#supported}

- Read/write basic file I/O on a ZPL dataset
- Create pool image (`Format`)
- Directories, rename
- Pool info (`Info`)
- Grow / Shrink / Resize, including `ShrinkWithMode` (Rebuild / InPlace / Auto)
- Snapshots (`Snapshot`, `OpenSnapshot`, `DestroySnapshot`) and clones (`Clone`, `Origin`)
- Transparent decompress on read (lz4 / gzip / zstd / lzjb / zle); lz4 uses the shared `go-compressions/lz4` `DecodeBlock`
- Block checksums verified on read: fletcher2/4, SHA-256, SHA-512/256 (v0.6.0), and BLAKE3, Skein and Edon-R keyed with the pool checksum salt, plain and encrypted (v0.7.0, checked against pools written by the OpenZFS 2.3.4 kernel)
- Native encryption read (`OpenFromDeviceDatasetWithKey`, AES-CCM/GCM)
- Symlinks and hardlinks; chmod/chown/chtimes (type-assert the optional interfaces)
- Multi-vdev (mirror / RAID-Z) pools opened read via `OpenFromDevices`

## Parallel reads {#parallel-reads}

Since v0.8.0 the records of a read are read, checksum-verified (a damaged copy
falls back to the block's other DVAs) and decompressed on up to
`min(GOMAXPROCS, 8)` goroutines (`inflight.Ordered`: bounded, file order kept;
`SetReadParallelism(n)`, 1 = serial), and `ReadAt` keeps decoded records in a
bounded adaptive cache (`inflight.Cache`, 8 MiB growing to 256 MiB only when it
evicts records it is asked for again; `SetMaxDataCache`). One 256 MiB text-like
file against the OpenZFS 2.4.1 kernel (4 vCPU VM, regimes calibrated and uniform),
sequential read at 4 CPUs, MiB/s:

| data | kernel | v0.7.3 | v0.8.0 |
|---|---:|---:|---:|
| off, fletcher4 | 3865 | 4297 | 5199 |
| lz4 | 2072 | 1895 | 3361 |
| zstd | 1000 | 743 | 1876 |

At one CPU the sequential figures are unchanged. Random 4 KiB reads of compressed
data, warm: lz4 88 k reads/s against the kernel's 27 k (v0.7.3: 15 k), zstd 86 k
against 9.9 k (5.9 k). Still behind the kernel: sequential zstd at one CPU (795
against 924 MiB/s) and random reads of uncompressed data (0.43 to 0.57 x).

## Not implemented {#not-implemented}

- The writer only targets a single dataset in a single-vdev pool (the reader
  navigates nested datasets and multi-vdev/RAID-Z pools)
- The writer does not compress or encrypt data blocks it produces
- Snapshots/clones are eager block copies, not O(1) copy-on-write; no ACLs
- A missing RAID-Z data leg is not yet reconstructed from parity

## Untrusted input {#untrusted-input}

Every allocation and walk sized from a value on the image is checked against
what the image can hold before it happens, with bounds taken from the OpenZFS
sources: block pointer sizes are at most `SPA_MAXBLOCKSIZE` (16 MiB), and
`ReadFile` / `ReadLink` are capped at `DefaultMaxReadSize` (1 GiB, changed with
`SetMaxReadSize`) and refused with `ErrTooLarge` before any buffer is sized.

## Install {#install}

```bash
go get github.com/go-filesystems/zfs
```

- Source: <https://github.com/go-filesystems/zfs>
- API reference: <https://pkg.go.dev/github.com/go-filesystems/zfs>

{{< callout type="info" >}}
See the module's README for full, up-to-date details.
{{< /callout >}}
