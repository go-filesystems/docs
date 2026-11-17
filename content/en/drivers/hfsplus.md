---
title: "hfsplus"
weight: 100
---

# hfsplus {#hfsplus}

<img src="../../assets/fs/go-filesystems-hfsplus.png" width="110" align="right">

Apple HFS+ (Mac OS Extended) and its HFSX (case-sensitive) variant.

Implements [`filesystem.Filesystem`]({{< relref "../interface.md" >}}). No cgo, no root.

## Status {#status}

| Read | Write | Format | Label | Symlinks | On-disk format |
|:--:|:--:|:--:|:--:|:--:|---|
| ✅ | ✅ | ✅ | ✅ | ✅ | HFS+ / HFSX, big-endian catalog + extents-overflow B-trees |

## Supported {#supported}

- Read/write files
- Directories, rename
- `SetLabel` / `Symlink` / `Truncate` (optional `Labeller` / `Symlinker` / `Truncater` capabilities)
- MBR/GPT auto-detect
- Times (`filesystem.TimesReader`, v0.7.0): creation, content modification, attribute
  modification (Mac OS X's change time) and access from the catalog record, to 1 s,
  GMT seconds since 1904-01-01 (Apple TN1150). A date of 0 is "not recorded", and so is
  the root folder's. Checked against an image written by macOS `newfs_hfs`

## Journaled volumes {#journal}

Since v0.9.0 the write path knows about the HFS+ journal (Apple TN1150). It
does not write transactions itself, and macOS replays a journal that holds some
over whatever was written around it, so:

- A **journaled volume with an empty journal** (start == end, what a clean
  unmount leaves) is written directly. The journal info block, the journal and
  the journaled attribute are left as they were, so macOS replays nothing at
  the next mount. Since v0.12.3 the first write stamps `lastMountedVersion` =
  `'GoHF'` into both volume headers (TN1150: an implementation that modifies
  the volume sets this field). It is not `'HFSJ'`, because this driver does not
  maintain the journal; writing through the journal
  ([go-filesystems/hfsplus#45](https://github.com/go-filesystems/hfsplus/issues/45))
  will switch it to `'HFSJ'`.
- A **journal with transactions in it**, or one that cannot be verified (a bad
  checksum, a journal outside the volume or on another device), is refused by
  `OpenWritable` and `OpenFileWritable` with the typed error `ErrJournaled`.
  Nothing is written, and `OpenFileWritable` closes the file. Test it with
  `errors.Is(err, hfsplus.ErrJournaled)`. Since v0.12.3 the header checks follow
  Apple's `journal_open` (`core/hfs_journal.c`, hfs-715.100.10): a journal is
  refused when `jhdr_size` is 0, the journal size is outside [256 KiB, 1 GiB],
  or start or end is 0, beyond the journal size, beyond 1 GiB, or not a
  multiple of 512. Two bounds guards protect our reader: `jhdr_size` at least
  the 44-byte header and at most the journal size. Three older checks stay
  stricter than Apple's (the `JHDR` magic, the endian field, the header size
  against the Journal Info Block) and can turn a journal macOS opens into
  `ErrJournaled`
  ([go-filesystems/hfsplus#48](https://github.com/go-filesystems/hfsplus/issues/48)).
- Read-only `Open` is unchanged: it reads a journaled volume whatever its
  journal holds. A caller that gets `ErrJournaled` can fall back to it.
- Volumes made by `Format` and `Mkfs` are not journaled.

{{< callout type="warning" >}}
**Do not write an image that is mounted.** Neither the driver nor the system
that mounted it knows about the other's changes. Write an image nothing else
has open.
{{< /callout >}}

## Durability levels {#durability}

Since v0.12.0 a writable volume chooses, per volume, when a mutation is on the
medium: `SetDurability(DurabilityStrict | DurabilityMounted | DurabilityLazy)`
and `Durability()`. `Batch(fn)` runs a group of mutations at `DurabilityLazy` and
makes them durable with one `Sync`, so the group pays one set of barriers (it is
not a transaction). The default, `DurabilityStrict`, is unchanged from v0.11.0.
A barrier is `File.Sync` (`F_FULLFSYNC` on macOS); the counts per commit are:

| Level | First commit | Next, no data | With data | `Sync` | Durable when the mutator returns | Volume between commits |
|---|:--:|:--:|:--:|:--:|:--:|---|
| `DurabilityStrict` (default) | 3 | 3 | 3 | 1 | yes | marked clean |
| `DurabilityMounted` | 2 | 1 | 2 | 1 | yes | marked dirty until `Sync` or `Close` |
| `DurabilityLazy` | 1 | 0 | 0 | 2 | no, only after `Sync` or `Close` | marked dirty until `Sync` or `Close` |

Measured against macOS's own HFS+ driver (v0.12.2, 2026-11-16, 1-minute load
4.2 to 4.75, 30 interleaved rounds, opt-in test `measure_darwin_test.go` with
`HFSPLUS_MEASURE=1`), create + 1 byte + fsync:

| Driver | Time | Against macOS |
|---|--:|--:|
| macOS | 5.13 ms | 1.00x |
| `DurabilityStrict` | 12.66 ms | 2.47x |
| `DurabilityMounted` | 8.20 ms | 1.60x |
| `DurabilityLazy` | 0.095 ms | not durable until `Sync` |

The same 6 barriers over plain fsync(2) cost 0.29 ms, so the gap is the
`F_FULLFSYNC` count. Closing it is
[go-filesystems/hfsplus#45](https://github.com/go-filesystems/hfsplus/issues/45);
the design is `docs/design/journal-write.md` in the hfsplus repository.

- **`DurabilityMounted`** behaves like a mounted HFS+ volume (Apple TN1150) and
  is data-ordered, like ext4 `data=ordered`: a commit that wrote file data puts a
  barrier between the data and the metadata that points at it. A crash between
  syncs leaves the volume marked dirty, so the next mount runs `fsck_hfs`, and a
  journaled volume stays unjournaled until `diskutil enableJournal`.
- **`DurabilityLazy`** is data-writeback: nothing orders file data before its
  metadata, so after a crash a file may hold stale bytes of a deleted one, and
  the mutations since the last `Sync` are lost. **Do not use it where different
  users share a volume**, or for a volume other clients mount at the same time.
- `SetDurability` and `Batch` must not run concurrently with a mutator, `Sync` or
  `Close`. Leaving a level that holds the volume dirty settles it first.

[fileshare](https://github.com/go-fileshare/fileshare) exposes the level as the
per-share option `hfsplus_durability`.

## Untrusted input {#untrusted-input}

Every size in an image is a claim. A fork's logical size is bounded by the image
size and every extent it uses must lie inside the image before the buffer is
made; with the size unknown the buffer grows with the bytes the reader delivers.
The B-tree leaf-chain walk is bounded by image size / node size.

## Install {#install}

```bash
go get github.com/go-filesystems/hfsplus
```

- Source: <https://github.com/go-filesystems/hfsplus>
- API reference: <https://pkg.go.dev/github.com/go-filesystems/hfsplus>

{{< callout type="info" >}}
See the module's README for full, up-to-date details.
{{< /callout >}}
