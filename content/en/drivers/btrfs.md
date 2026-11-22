---
title: "btrfs"
weight: 50
---

# btrfs {#btrfs}

<img src="../../assets/fs/go-filesystems-btrfs.png" width="110" align="right">

Copy-on-write Linux fs with snapshots and subvolumes.

Implements [`filesystem.Filesystem`]({{< relref "../interface.md" >}}). No cgo, no root.

## Status {#status}

| Read | Write | Format | Label | Symlinks | On-disk format |
|:--:|:--:|:--:|:--:|:--:|---|
| ✅ | ✅ | ✅ | ✅ | ✅ | Single-device, CRC32c (btrfs-progs ≥ 5.x) |

## Supported {#supported}

- Read/write files
- Directories, rename, symlinks
- MBR/GPT auto-detect
- Compressed extents: kernel-format zstd extents (v0.5.4/v0.5.5) and LZO through the shared `go-compressions/lzo` decoder, zlib through `go-compressions/deflate` (v0.6.4)

## Not implemented {#not-implemented}

- Snapshots, send/receive, multi-device/RAID, quotas and reflink are not implemented

## Parallel reads {#parallel-reads}

Since v0.6.0 the compressed extents (zlib, LZO, zstd) of a file are read and decoded in
parallel by `ReadFile` and `OpenFile().ReadAt` (`inflight.Ordered`: bounded,
file order kept, the first error in file order; `SetReadParallelism(n)`, default
`min(GOMAXPROCS, 8)`, 1 = serial). Sequential read of a 256 MiB text-like file
the Linux kernel (6.17, 4 vCPU VM) wrote with `compress-force`, MiB/s:

| codec | kernel 1 CPU | ours 1 CPU | kernel 4 CPUs | ours 4 CPUs |
|---|---:|---:|---:|---:|
| lzo | 1169 | 1376 | 3317 | 2536 |
| zstd | 809 | 1175 | 2471 | 2290 |
| zlib | 360 | 303 | 1233 | 924 |

Before (v0.5.6, serial): 4 CPUs 1326 (lzo), 667 (zstd), 236 (zlib). Still behind
the kernel: zlib at 1 and 4 CPUs, lzo at 4 CPUs.

Since v0.6.1 the Adler-32 trailer of a zlib extent is checked with
`go-simd/adler32` (NEON, AVX2, AVX-512) over raw DEFLATE, instead of the scalar
sum, which was about 14 % of a 128 KiB extent's decode time. Against the kernel
in the same run, zlib reads go from 0.88 to 0.91 of its speed at 1 CPU (285 / 324
to 325 / 357 MiB/s) and from 0.70 to 0.78 at 4 CPUs (824 / 1179 to 939 / 1212);
inflate itself is still the cost. A later run of v0.6.1 (kernel / ours, 1 and 4
CPUs) gave zlib 345 / 304 and 1204 / 839, lzo 1058 / 1334 and 3093 / 2322, so the
VM's run-to-run spread is up to about 10 % and the 4-CPU lzo ratio sits around
0.75 to 0.79.

Since v0.6.3 a `ReadAt` that continues the previous one, on a file with
compressed extents and more than one worker, decodes 16 MiB at once (about 16 MiB
of memory per open file) and serves the next calls from that window: a 1 MiB
`ReadAt` holds only 8 extents, and decoding each call alone filled the workers and
drained them again. Against the kernel in the same run (v0.6.2 to v0.6.3), lzo at
4 CPUs goes from 0.78 (2377 / 3042) to 1.05 (3509 / 3327), zlib at 4 CPUs from
0.72 to 0.87, zstd at 4 CPUs from 0.70 to 0.78; lzo at 1 CPU stays at 1.16 to 1.19.
Still behind the kernel after v0.6.3: zlib at 1 and 4 CPUs, zstd at 4 CPUs.

Since v0.6.4 zlib extents are inflated by the whole-buffer raw DEFLATE decoder of
`go-compressions/deflate` v0.2.0 (`InflatePrefix`, which also reports where the
stream ends so the Adler-32 trailer can be read), instead of klauspost/compress.
On the host, one core, 128 KiB extents, the extent decode alone went from 396 MiB/s
to 1.03 GiB/s (geometric mean of six inputs, 6 interleaved rounds against v0.6.3).
Ratio ours / kernel in the same VM run, medians of 5, sequential read of the
256 MiB file:

| codec | CPUs | ours / kernel | MiB/s (ours / kernel) |
|---|---:|---:|---|
| zlib | 1 | 2.15 (v0.6.3: 0.91) | 763 / 354 |
| zlib | 4 | 1.89 (v0.6.3: 0.87) | 2304 / 1222 |
| lzo | 4 | 1.05 | |
| zstd | 4 | 0.78 | |

zlib and lzo at 4 CPUs are now ahead of the kernel; zstd at 4 CPUs is still behind
([btrfs#56](https://github.com/go-filesystems/btrfs/issues/56) stays open for it).

## Untrusted input {#untrusted-input}

Every allocation, walk and recursion sized from a value on the image is checked
against what the format can produce before it happens: a compressed extent is at
most 128 KiB (the kernel's `BTRFS_MAX_UNCOMPRESSED`), a zstd window 128 KiB,
overlapping file extent items are refused, and a chunk `stripe_len` must be
exactly 64 KiB.

## Install {#install}

```bash
go get github.com/go-filesystems/btrfs
```

- Source: <https://github.com/go-filesystems/btrfs>
- API reference: <https://pkg.go.dev/github.com/go-filesystems/btrfs>

{{< callout type="info" >}}
See the module's README for full, up-to-date details.
{{< /callout >}}
