---
title: "Drivers"
weight: 30
---

# Drivers {#drivers}

The 15 filesystem drivers available today (plus `oci`, a read-only
image-as-filesystem overlay, and `detect`, a type-probing dispatch registry —
both listed separately below since neither targets an on-disk *format* in the
same sense as the rows in this table).

| | Module | Read | Write | Format | Label | Symlinks | Quota | Snapshots | Encryption | On-disk format | Platform |
|---|---|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|---|---|
| <img src="../assets/fs/go-filesystems-apfs.png" width="28"> | [`apfs`]({{< relref "apfs.md" >}}) | ✅ | ✅ | ✅ | — | ✅ | — | ✅ | ✅ | Real APFS on-disk (kext-mountable), GPT-aware | macOS / iOS |
| <img src="../assets/fs/go-filesystems-btrfs.png" width="28"> | [`btrfs`]({{< relref "btrfs.md" >}}) | ✅ | ✅ | ✅ | ✅ | ✅ | ✕ | ✕ | — | Single-device, CRC32c (btrfs-progs ≥ 5.x) | Linux |
| <img src="../assets/fs/go-filesystems-exfat.png" width="28"> | [`exfat`]({{< relref "exfat.md" >}}) | ✅ | ✅ | ✅ | ✅ | ✕ | — | — | — | exFAT | Windows / removable |
| <img src="../assets/fs/go-filesystems-ext4.png" width="28"> | [`ext4`]({{< relref "ext4.md" >}}) | ✅ | ✅ | ✅ | ✅ | ✅ | ✕ | — | ✕ | ext4 — extents, 64-bit, flex_bg, dir htree, metadata_csum (CRC32c) | Linux |
| <img src="../assets/fs/go-filesystems-fat32.png" width="28"> | [`fat32`]({{< relref "fat32.md" >}}) | ✅ | ✅ | ✅ | ✅ | ✕ | — | — | — | FAT32; FAT12 / FAT16 (resize keeps the FAT type) | cross-platform |
| <img src="../assets/fs/go-filesystems-ffs.png" width="28"> | [`ffs`]({{< relref "ffs.md" >}}) | ✅ | ✅ | ✅ | — | ✅ | ✕ | ✕ | — | NetBSD/OpenBSD FFSv1/FFSv2 — thin re-export of `ufs` | BSD |
| <img src="../assets/fs/go-filesystems-hfsplus.png" width="28"> | [`hfsplus`]({{< relref "hfsplus.md" >}}) | ✅ | ✅ | ✅ | ✅ | ✅ | — | — | — | HFS+ / HFSX (Mac OS Extended), big-endian catalog + extents-overflow B-trees | macOS (legacy) |
| <img src="../assets/fs/go-filesystems-iso9660.png" width="28"> | [`iso9660`]({{< relref "iso9660.md" >}}) | ✅ | ✕ | ✕ | — | ✅ | — | — | — | ISO 9660 / ECMA-119 + Rock Ridge (names/perms/symlinks) + Joliet (UCS-2 names) | cross-platform (optical) |
| <img src="../assets/fs/go-filesystems-ntfs.png" width="28"> | [`ntfs`]({{< relref "ntfs.md" >}}) | ✅ | ✅ | ✅ | ✅ | ✕ | — | — | — | Minimal in-image blob model — NOT the real NTFS on-disk format; a separate read-only reader parses genuine NTFS | Windows |
| <img src="../assets/fs/go-filesystems-squashfs.png" width="28"> | [`squashfs`]({{< relref "squashfs.md" >}}) | ✅ | ✕ | ✅ | — | ✅ | — | — | — | SquashFS 4.0 read-only archive; gzip/xz/zstd/lzo/lz4 blocks + fragments | Linux |
| <img src="../assets/fs/go-filesystems-uefi.png" width="28"> | [`uefi`]({{< relref "uefi.md" >}}) | ✅ | ✅ | ✅ | — | — | — | — | — | OVMF/EDK2 NvVar variable store; time-based authenticated writes | firmware (UEFI) |
| <img src="../assets/fs/go-filesystems-ufs.png" width="28"> | [`ufs`]({{< relref "ufs.md" >}}) | ✅ | ✅ | ✅ | — | ✅ | ✕ | ✕ | — | UFS2 (FreeBSD 14.x) read+write; UFS1 read; NetBSD/OpenBSD FFSv1/FFSv2 (via `ffs` alias module) | BSD |
| <img src="../assets/fs/go-filesystems-xfs.png" width="28"> | [`xfs`]({{< relref "xfs.md" >}}) | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | — | ✕ | XFS v5 (CRC32c, ftype) | Linux |
| <img src="../assets/fs/go-filesystems-zfs.png" width="28"> | [`zfs`]({{< relref "zfs.md" >}}) | ✅ | ✅ | ✅ | — | ✅ | ✕ | ✅ | ✅ read-only | Single pool, single-vdev writer (multi-vdev/RAID-Z read-only) | illumos / BSD / Linux |
| | [`oci`](https://github.com/go-filesystems/oci) | ✅ | ✕ | — | — | ✅ | — | — | — | OCI/Docker image layers merged read-only (whiteouts, opaque dirs, hardlinks) | any (image, not disk) |

**Legend:** ✅ supported by the driver · ✕ not yet implemented (the format has
the feature) · — not applicable. **Read** = open & inspect · **Write** =
mutate in place · **Format** = create a fresh image. Quota / Snapshots /
Encryption reflect **driver** support, not just the on-disk format's
capability.

**Reading a range of a large file.** A server that answers a read of a few bytes
(NFS `READ`, SFTP) should not load the whole file. Since the audit of
2026-11-05, `apfs` v0.6.0, `btrfs` v0.5.0, `hfsplus` v0.6.0, `ntfs` v0.4.0,
`ufs` v0.5.0, `xfs` v0.5.0 and `zfs` v0.5.0 implement `filesystem.Opener`
(`OpenFile(path)` returns a file whose `ReadAt` reads only the extents or blocks of
its window; holes read as zeros). A 1 MiB read at 66 MiB of a 72 MiB file allocates 0 bytes
through it on `hfsplus`, `apfs`, `btrfs`, `xfs` and `ntfs`, 5.5 MB on `ufs` and
1.1 MB on `zfs`, against 75 to 538 MB through `ReadFile`. A compressed (LZNT1)
stream of a real NTFS volume is still decoded whole, once, on the first read.
go-filesystems/nfs bounds the whole-file fallback at 64 MiB (`NFS3ERR_IO` above).

[`oci`](https://github.com/go-filesystems/oci) bounds every input-driven
allocation before it happens (`DefaultLimits()`: manifests 16 MiB, 16 GiB for a
whole image); an excess returns an error wrapping `safeio.ErrTooLarge`.

Also in the org: [`detect`](https://github.com/go-filesystems/detect), a
`blkid`-style type prober and driver-opener registry that dispatches to the
drivers above — it isn't an on-disk format itself, so it has no row.

And [`cfb`]({{< relref "../cfb.md" >}}), a read-only reader for Microsoft Compound
File Binary containers (`.doc`, `.xls`, `.msi`, Prim'X `.zed`): a file system inside
a file, but not a `Filesystem` driver, so it has no row either.
