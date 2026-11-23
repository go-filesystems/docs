---
title: "ext4"
weight: 70
---

# ext4 {#ext4}

<img src="../../assets/fs/go-filesystems-ext4.png" width="110" align="right">

Linux ext4 — extents, 64-bit, journaling, metadata_csum.

Implements [`filesystem.Filesystem`]({{< relref "../interface.md" >}}). No cgo, no root.

## Status {#status}

| Read | Write | Format | Label | Symlinks | On-disk format |
|:--:|:--:|:--:|:--:|:--:|---|
| ✅ | ✅ | ✅ | ✅ | ✅ | ext4 — extents, 64-bit, flex_bg, dir htree, metadata_csum (CRC32c) |

## Supported {#supported}

- Read/write files (extents, sparse writes)
- Directories, rename, symlinks
- metadata_csum (CRC32c) writes
- Online grow / shrink / resize (`Grow`, `Shrink`, `Resize`)
- `CheckImage` / `RepairImage` hooks
- `GetFlags`/`SetFlags` ioctls
- MBR/GPT auto-detect
- Times (`filesystem.TimesReader`, v0.5.0): access, modification and change to 1 ns
  when the inode's `_extra` words fit (their two epoch bits extend the seconds), 1 s
  otherwise; creation from `i_crtime` only when the inode is large enough, so 128-byte
  ext2/ext3 inodes have none. All-zero is "not recorded". Checked against images made
  by `mke2fs` and `debugfs set_inode_field`

## Not implemented {#not-implemented}

- Some cross-check tests require `mke2fs`
- Not every kernel ioctl / obscure feature is implemented

## Checksums {#checksums}

Since v0.6.0, with `metadata_csum` set, the superblock, group descriptors, inodes,
inode and block bitmaps and directory block tails are verified on read and on every
read-modify-write, as the
[kernel ext4 documentation](https://docs.kernel.org/filesystems/ext4/checksums.html)
specifies (crc32c seeded with the filesystem UUID, the inode number and the inode
generation). A mismatch returns `*ChecksumError` (`errors.As`), so a write can no
longer re-seal tampered metadata. `Options.IgnoreChecksums` is the forensic opt-out,
off by default. Extent-tree block checksums are not enforced yet
([ext4#46](https://github.com/go-filesystems/ext4/issues/46)).

## Untrusted input {#untrusted-input}

Every allocation and walk sized from a value on the image is checked against
what the image can hold before it happens. `ReadFile` is capped at the smaller of
the declared filesystem size and `DefaultMaxReadSize` (1 GiB), changed with
`SetMaxReadSize` and refused with `ErrTooLarge`; larger files are read with
`OpenFile`. Overlapping extent leaves are refused and the extent tree depth is
at most 5. Since v0.5.1 `DeleteDir` refuses a directory cycle (the
`go-volumes/safeio` walk: `safeio.ErrCycle`, and `safeio.ErrTooDeep` past 256
levels).

## Install {#install}

```bash
go get github.com/go-filesystems/ext4
```

- Source: <https://github.com/go-filesystems/ext4>
- API reference: <https://pkg.go.dev/github.com/go-filesystems/ext4>

{{< callout type="info" >}}
See the module's README for full, up-to-date details.
{{< /callout >}}
