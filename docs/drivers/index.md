# Drivers

The 12 filesystem drivers available today. **Read** = open & inspect · **Write** = mutate in place · **Format** = create a fresh image.

| | Module | Read | Write | Format | Label | Symlinks | On-disk format |
|---|---|:--:|:--:|:--:|:--:|:--:|---|
| <img src="../assets/fs/go-filesystems-apfs.png" width="28"> | [`apfs`](apfs.md) | ✅ | ✅ | ✅ | — | ✅ | Real APFS on-disk (kext-mountable), GPT-aware |
| <img src="../assets/fs/go-filesystems-btrfs.png" width="28"> | [`btrfs`](btrfs.md) | ✅ | ✅ | ✅ | ✅ | ✅ | Single-device, CRC32c (btrfs-progs ≥ 5.x) |
| <img src="../assets/fs/go-filesystems-exfat.png" width="28"> | [`exfat`](exfat.md) | ✅ | ✅ | ✅ | ✅ | ✕ | exFAT |
| <img src="../assets/fs/go-filesystems-ext4.png" width="28"> | [`ext4`](ext4.md) | ✅ | ✅ | ✅ | ✅ | ✅ | ext4 — extents, 64-bit, flex_bg, dir htree, metadata_csum (CRC32c) |
| <img src="../assets/fs/go-filesystems-fat32.png" width="28"> | [`fat32`](fat32.md) | ✅ | ✅ | ✅ | ✅ | ✕ | FAT32 |
| <img src="../assets/fs/go-filesystems-iso9660.png" width="28"> | [`iso9660`](iso9660.md) | ✅ | ✕ | ✕ | — | ✅ | ISO 9660 / ECMA-119 + Rock Ridge (names/perms/symlinks) + Joliet (UCS-2 names) |
| <img src="../assets/fs/go-filesystems-ntfs.png" width="28"> | [`ntfs`](ntfs.md) | ✅ | ✅ | ✅ | ✅ | ✕ | Minimal in-image blob model — NOT the real NTFS on-disk format |
| <img src="../assets/fs/go-filesystems-squashfs.png" width="28"> | [`squashfs`](squashfs.md) | ✅ | ✕ | ✕ | — | ✅ | SquashFS 4.0 read-only archive; gzip/xz/zstd/lzo blocks + fragments |
| <img src="../assets/fs/go-filesystems-uefi.png" width="28"> | [`uefi`](uefi.md) | ✅ | ✅ | ✅ | — | — | OVMF/EDK2 NvVar variable store; time-based authenticated writes |
| <img src="../assets/fs/go-filesystems-ufs.png" width="28"> | [`ufs`](ufs.md) | ✅ | ✅ | ✅ | — | ✅ | UFS2 (FreeBSD 14.x) read+write; UFS1 read |
| <img src="../assets/fs/go-filesystems-xfs.png" width="28"> | [`xfs`](xfs.md) | ✅ | ✅ | ✅ | ✅ | ✅ | XFS v5 (CRC32c, ftype) |
| <img src="../assets/fs/go-filesystems-zfs.png" width="28"> | [`zfs`](zfs.md) | ✅ | ✅ | ✅ | — | — | Single pool / single vdev (test-oriented subset) |
