# Drivers

The 12 filesystem drivers available today.

| | Module | Platform | Read | Write | Format | Label | Symlinks | Quota | Snapshots | Encryption | On-disk format |
|---|---|---|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|---|
| <img src="../assets/fs/go-filesystems-apfs.png" width="28"> | [`apfs`](apfs.md) | macOS / iOS | ✅ | ✅ | ✅ | — | ✅ | — | ✅ | ✕ | Real APFS on-disk (kext-mountable), GPT-aware |
| <img src="../assets/fs/go-filesystems-btrfs.png" width="28"> | [`btrfs`](btrfs.md) | Linux | ✅ | ✅ | ✅ | ✅ | ✅ | ✕ | ✕ | — | Single-device, CRC32c (btrfs-progs ≥ 5.x) |
| <img src="../assets/fs/go-filesystems-exfat.png" width="28"> | [`exfat`](exfat.md) | Windows / removable | ✅ | ✅ | ✅ | ✅ | ✕ | — | — | — | exFAT |
| <img src="../assets/fs/go-filesystems-ext4.png" width="28"> | [`ext4`](ext4.md) | Linux | ✅ | ✅ | ✅ | ✅ | ✅ | ✕ | — | ✕ | ext4 — extents, 64-bit, flex_bg, dir htree, metadata_csum (CRC32c) |
| <img src="../assets/fs/go-filesystems-fat32.png" width="28"> | [`fat32`](fat32.md) | cross-platform | ✅ | ✅ | ✅ | ✅ | ✕ | — | — | — | FAT32 |
| <img src="../assets/fs/go-filesystems-iso9660.png" width="28"> | [`iso9660`](iso9660.md) | cross-platform (optical) | ✅ | ✕ | ✕ | — | ✅ | — | — | — | ISO 9660 / ECMA-119 + Rock Ridge (names/perms/symlinks) + Joliet (UCS-2 names) |
| <img src="../assets/fs/go-filesystems-ntfs.png" width="28"> | [`ntfs`](ntfs.md) | Windows | ✅ | ✅ | ✅ | ✅ | ✕ | — | — | — | Minimal in-image blob model — NOT the real NTFS on-disk format |
| <img src="../assets/fs/go-filesystems-squashfs.png" width="28"> | [`squashfs`](squashfs.md) | Linux | ✅ | ✕ | ✅ | — | ✅ | — | — | — | SquashFS 4.0 read-only archive; gzip/xz/zstd/lzo/lz4 blocks + fragments |
| <img src="../assets/fs/go-filesystems-uefi.png" width="28"> | [`uefi`](uefi.md) | firmware (UEFI) | ✅ | ✅ | ✅ | — | — | — | — | — | OVMF/EDK2 NvVar variable store; time-based authenticated writes |
| <img src="../assets/fs/go-filesystems-ufs.png" width="28"> | [`ufs`](ufs.md) | BSD | ✅ | ✅ | ✅ | — | ✅ | ✕ | ✕ | — | UFS2 (FreeBSD 14.x) read+write; UFS1 read; NetBSD/OpenBSD FFSv1/FFSv2 (via `ffs` alias module) |
| <img src="../assets/fs/go-filesystems-xfs.png" width="28"> | [`xfs`](xfs.md) | Linux | ✅ | ✅ | ✅ | ✅ | ✅ | ✕ | — | ✕ | XFS v5 (CRC32c, ftype) |
| <img src="../assets/fs/go-filesystems-zfs.png" width="28"> | [`zfs`](zfs.md) | illumos / BSD / Linux | ✅ | ✅ | ✅ | — | — | ✕ | ✕ | ✕ | Single pool / single vdev (test-oriented subset) |

**Legend:** ✅ supported by the driver · ✕ not yet implemented (the format has the feature) · — not applicable. **Read** = open & inspect · **Write** = mutate in place · **Format** = create a fresh image. Quota / Snapshots / Encryption reflect **driver** support.
