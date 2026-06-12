# go-filesystems

Pure-Go filesystem drivers — read & write disk images, no cgo, no root.

`go-filesystems` is a set of dependency-free Go modules that read and write on-disk
filesystem images — one module per filesystem, all implementing the shared
[`filesystem.Filesystem`](interface.md) contract. No cgo, no root.

## Drivers

<div class="fs-grid" markdown>
<a class="fs-card" href="drivers/apfs.md"><img src="assets/fs/go-filesystems-apfs.png"><span><code>apfs</code><br><small>Apple File System.</small></span></a>
<a class="fs-card" href="drivers/btrfs.md"><img src="assets/fs/go-filesystems-btrfs.png"><span><code>btrfs</code><br><small>Copy-on-write Linux fs with snapshots and subvolumes.</small></span></a>
<a class="fs-card" href="drivers/exfat.md"><img src="assets/fs/go-filesystems-exfat.png"><span><code>exfat</code><br><small>Extended FAT for large media.</small></span></a>
<a class="fs-card" href="drivers/ext4.md"><img src="assets/fs/go-filesystems-ext4.png"><span><code>ext4</code><br><small>Linux ext4 — extents, 64-bit, journaling, metadata_csum.</small></span></a>
<a class="fs-card" href="drivers/fat32.md"><img src="assets/fs/go-filesystems-fat32.png"><span><code>fat32</code><br><small>FAT with 32-bit allocation.</small></span></a>
<a class="fs-card" href="drivers/ntfs.md"><img src="assets/fs/go-filesystems-ntfs.png"><span><code>ntfs</code><br><small>Windows NT filesystem.</small></span></a>
<a class="fs-card" href="drivers/uefi.md"><img src="assets/fs/go-filesystems-uefi.png"><span><code>uefi</code><br><small>EFI System Partition (FAT-based).</small></span></a>
<a class="fs-card" href="drivers/ufs.md"><img src="assets/fs/go-filesystems-ufs.png"><span><code>ufs</code><br><small>Unix File System (BSD).</small></span></a>
<a class="fs-card" href="drivers/xfs.md"><img src="assets/fs/go-filesystems-xfs.png"><span><code>xfs</code><br><small>High-performance journaling filesystem.</small></span></a>
<a class="fs-card" href="drivers/zfs.md"><img src="assets/fs/go-filesystems-zfs.png"><span><code>zfs</code><br><small>Copy-on-write pooled storage filesystem.</small></span></a>
</div>
