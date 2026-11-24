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
  the next mount. Since v0.12.3 the first commit that changes anything, on
  every volume, journaled or not, stamps `lastMountedVersion` =
  `'GoHF'` into both volume headers (TN1150: an implementation that modifies
  the volume sets this field). It is not `'HFSJ'`, because these direct writes
  do not maintain the journal. Writing through the journal (below) stamps
  `'HFSJ'`.
- **Since v0.18.0, journal writes are on by default.** A writable open of a
  journaled volume writes through its journal at `DurabilityStrict` and
  `DurabilityMounted`, as macOS's own driver does, and stamps `'HFSJ'`. A
  journal that fails the checks below falls back silently to direct writes,
  exactly as in v0.17.0, and `(*Volume).JournalWritesReason()` says why. It
  returns `nil` when journal writes are on, `ErrReadOnly` or `ErrClosed`, and
  otherwise an error wrapping `ErrJournalWritesUnavailable`. In v0.18.0
  `DurabilityLazy` and `Batch` wrote in place, after the journal was emptied;
  since v0.19.0 they are a group commit (below). From v0.15.0
  to v0.17.0 this was an opt-in, **`(*Volume).EnableJournalWrites()`**, which
  remains and does nothing when journal writes are already on.
  A commit costs 2 barriers for a session's first commit and 1 after that,
  against 3 on the direct path. A commit falls back to the direct path, after the journal
  is emptied (barrier, empty header, barrier), when it needs more than 30 runs,
  does not fit even an empty journal, or has a run that is not 512-byte aligned.
  Since v0.17.0 a commit that fits an empty journal but not the live one
  empties the journal and stays in journal mode (4 barriers; see below). It is refused with `ErrJournalWritesUnavailable` on a volume that is
  not journaled, a journal that holds transactions or was never initialised,
  a `jhdr_size` other than 512, or a writer without `Sync`. Checked:
  - our replayer finds no wrong state on any crash image of commits, `Close`
    and `SetDurability`;
  - 820 distinct crash images all mount on macOS, and `fsck_hfs -fn` exits 0
    after the mount.

  A raw `fsck_hfs -fn` run before any mount, on an image cut mid-commit, cannot
  verify it: 407 of the 820 exit non-zero with "could not be verified
  completely". The kernel mounts them all. The reason is not established. The
  encoder writes only inside the journal.

  Since v0.16.0 commits that write **file data** stay in journal mode.
  - **Small data**, up to min(1 MiB, (journal size − 512) / 8), is journaled
    with its metadata, then written in place: 1 barrier, 2 on a session's
    first commit.
  - **Large data** is written in place, then a barrier, then the metadata
    commit: 2 barriers, 3 on the first.
  - **Revoke rule:** a block that one of the last three transactions journaled
    is never overwritten in place by unjournaled data, because replay would
    put its old bytes back. Such a write is journaled too, or the allocator
    skips the block.
  - **Forged sectors:** file data passes the same forged-sector check as
    metadata. A data sector that would pass as a header takes the large-data
    path and never enters the ring.

  A crash oracle checks every file's bytes after replay, over about 137,000
  images, with barrier groups landing in order and in reverse. After any crash
  a file reads its old or its new contents, never another file's stale bytes.
  On macOS, 24,144 distinct crash images pass. Not covered: the emulated CI
  lanes skip the data oracles, and no oracle builds a big-endian journal
  ([go-filesystems/hfsplus#61](https://github.com/go-filesystems/hfsplus/issues/61)).
  On a quiet machine, create + 1 byte + fsync takes 4.37 ms, against 9.41 ms
  for macOS on Journaled HFS+ (see [Durability levels](#durability)).

  Since v0.17.0, a full journal, `Close`, `Sync` and level switches follow
  macOS:
  - **Journal full:** the journal is emptied and the commit is written as a
    session's first commit, 4 barriers in all. Sequence numbers continue from
    the previous transaction. Ordinary commits on a journal of 256 KiB or more
    (the smallest the driver opens, as Apple's `journal_open`) never reach
    this. The tests force it, and no test yet reaches it through the API
    ([go-filesystems/hfsplus#64](https://github.com/go-filesystems/hfsplus/issues/64)).
  - **`Close`** commits a last transaction that sets the clean-unmount bit,
    writes it in place, then empties the journal: 3 barriers. The volume is
    left as macOS leaves it after unmount. When two steps fail, `Close`
    returns both errors (`errors.Join`).
  - **`Sync`** issues 1 barrier when in-place writes are pending, and none
    otherwise.
  - **Leaving `DurabilityStrict`** empties the journal first (2 barriers), so
    journal and direct writes never interleave.

  The crash oracle replays about 28,000 crash images of these paths (wrap,
  journal full, full across the wrap, bypass right after emptying, `Close`).
  After an injected write or barrier failure, nothing more reaches the device.

  **Since v0.19.0, `DurabilityLazy` and `Batch` are an in-memory group commit**
  on a journaled volume written through its journal. A Lazy commit writes
  nothing and pays 0 barriers; see [Durability levels](#durability).
  **Fixed in v0.19.0:** a span sort that wrapped on 32-bit for volumes over
  2 GiB (present in earlier releases). A linux-386 CI lane now covers it.

  **Security, v0.15.1:** upgrade from v0.15.0 if you use `EnableJournalWrites`
  ([go-filesystems/hfsplus#56](https://github.com/go-filesystems/hfsplus/issues/56)).
  After a crash, replay reads past the journal's `end` and accepts a sector
  there as a block list header when its 32-byte checksum verifies and its
  sequence number is guessable (Apple `hfs_journal.c:1225-1298`). Journaled
  catalog nodes hold file names a user chooses, so a crafted sector left in the
  ring could make replay write blocks anywhere on the volume. Since v0.15.1:
  - every 512-byte sector a commit would journal is checked against that
    checksum in both byte orders, and a commit that holds one takes the direct
    path, so the sector never reaches the ring. Every commit touching the same
    catalog node falls back, whoever makes it;
  - the writer never emits sequence number 0, and `EnableJournalWrites` refuses
    a journal whose `start` or `end` is not sector-aligned.

  Shown: our own replayer follows such a forged list without the guard.
  Unproven: whether a file name can be placed on a sector boundary through the
  API, and whether the kernel or `fsck_hfs` follows the forged list.

  Since v0.15.2
  ([go-filesystems/hfsplus#58](https://github.com/go-filesystems/hfsplus/issues/58)),
  forged sectors already in the ring are removed too: one written by v0.15.0, or
  by another implementation. A clean unmount does not remove them. The first
  journaled commit of a session scans the ring once and zeroes every sector
  that verifies as a header, in either byte order. This happens only after
  re-reading the journal header and finding the journal empty, and the scan
  never touches the journal header or info block. If anything was zeroed, one
  barrier follows before the first transaction: that commit costs 3 barriers
  instead of 2, and later commits still cost 1. The scan reads the whole ring
  once per session, up to 1 GiB, in 64 KiB chunks.
  Journal writes are PR 3 of 7 of
  [go-filesystems/hfsplus#45](https://github.com/go-filesystems/hfsplus/issues/45).
- Since v0.14.0, a **journal with transactions in it** on a volume last
  mounted by a journaling implementation (`lastMountedVersion == 'HFSJ'`,
  TN1150) is **replayed onto the volume** by `OpenWritable`, `OpenWritableAt`
  and `OpenFileWritable`, then opened as above. The order is:
  1. the replayed blocks;
  2. a barrier;
  3. the journal header with `start = end`;
  4. a barrier.

  A crash at any point leaves either a journal that still replays to the same
  result, or the fully replayed volume. This is stricter than macOS, which
  flushes only after the header (`hfs_journal.c:515-539`). It costs 2 barriers
  at open, at every durability level; a barrier is `F_FULLFSYNC` on darwin. On
  a journal macOS wrote, macOS then lists our replay plus a new file, and
  `fsck_hfs -fn` exits 0. Replayed blocks are bounded by the image size and
  never overlap the journal.
- These are still refused by the writable opens with the typed error
  `ErrJournaled`; test it with `errors.Is(err, hfsplus.ErrJournaled)`:
  - a journal with transactions on a volume **not** marked `'HFSJ'`. macOS
    discards such a journal (`hfs_vfsutils.c:723-727`, `:2886-2895`); we keep
    it, because it may hold the only copy of metadata. Mount and unmount the
    volume on macOS first;
  - a journal with transactions when the writer has no `Sync`. Without a
    barrier, an emptied header could land over blocks that never did;
  - a replay that rewrites the journal info block so that it makes the journal
    unreadable;
  - a journal the replayer itself refuses (`ErrJournalReplay`,
    `ErrReplayBudget`, or a replayed block inside the journal). Both errors
    wrap `ErrJournaled`;
  - a journal that cannot be verified: a bad checksum, or a journal outside
    the volume or on another device.

  `OpenFileWritable` closes the file it opened. Nothing is written in these
  cases, except the journal-info-block case: there the replay has already been
  written when the check refuses. An error part-way through a replay is crash-safe in
  the same way as a crash. Since v0.12.3 the header checks follow
  Apple's `journal_open` (`core/hfs_journal.c`, hfs-715.100.10): a journal is
  refused when `jhdr_size` is 0, the journal size is outside [256 KiB, 1 GiB],
  or start or end is 0, beyond the journal size, beyond 1 GiB, or not a
  multiple of 512. Two bounds guards protect our reader: `jhdr_size` at least
  the 44-byte header and at most the journal size. Three older checks stay
  stricter than Apple's: the magic (we accept only `JNLx`, Apple also accepts
  `JHDR`, which we refuse), the endian field, and the journal size recorded in
  the journal header against the Journal Info Block's. They can turn a journal macOS opens into
  `ErrJournaled`
  ([go-filesystems/hfsplus#48](https://github.com/go-filesystems/hfsplus/issues/48)).
- Read-only `Open` reads a journaled volume whatever its journal holds. By
  default it reads the blocks in place, which can be stale for metadata still in
  the journal. A caller that gets `ErrJournaled` can fall back to it.
- Since v0.13.0, `Open(r, size, hfsplus.WithJournalReplay())` (and the same
  option on `OpenFile`) replays the journal into memory first, read-only, and
  serves the volume as macOS would mount it. The image is never written, and
  the size must be known (not -1). The
  replayer ports Apple's `replay_journal` (`core/hfs_journal.c`,
  hfs-715.100.10):
  - a transaction starts at a block list marked `BLHDR_FIRST_HEADER`;
  - replay reads past `end` while the sequence number stays equal or goes up
    by one;
  - a torn block drops its transaction;
  - a bad block list header inside the journal's `start`..`end` range drops
    that transaction AND the one before it; past `end`, a bad list just ends
    the read.

  On a journal macOS wrote, the replayed volume lists the same 200 files with
  the same bytes as macOS's own mount. The journal is untrusted input: every
  size and offset is bounded. The two fuzz targets found no failure: at v0.16.0,
  5 minutes each gave 83.0 million executions of `FuzzJournalReplay` (machine
  load 7.3) and 14.1 million of `FuzzOpenJournalReplay` (load 54). Earlier runs
  showed 0 execs/sec ticks. Those were Go minimizing each new input, which the
  rate does not count: with `-fuzzminimizetime 0` no tick drops to zero
  ([go-filesystems/hfsplus#51](https://github.com/go-filesystems/hfsplus/issues/51)).
  More than 256 MiB of pending data is refused with `ErrReplayBudget`; that
  limit is ours, Apple's code has none. Since v0.14.0 the writable opens replay
  onto the volume with the same replayer (above).
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
not a transaction). A barrier is `File.Sync` (`F_FULLFSYNC` on macOS).

Since v0.18.0, on a journaled volume whose journal passes the checks,
`DurabilityStrict` (the default) and `DurabilityMounted` are both journal
commits. Each costs 2 barriers for a session's first commit and 1 after
that, with or without data. `Sync` costs 0 or 1, and the volume is never
marked dirty. A commit that cannot be journaled takes the direct
`DurabilityStrict` path at both levels (3 barriers).

Since v0.19.0, on the same volumes `DurabilityLazy` and `Batch` are an
in-memory group commit: a Lazy commit writes nothing and pays 0 barriers. These
write the group as one journal transaction:

- `Sync` and `Close`;
- a switch away from `DurabilityLazy`;
- the group's size bound, min(1 MiB, journal/8), 30 runs and 4096 freed spans;
- `Bytes()` on an in-memory volume.

A `Batch` whose mutations fit one group is atomic after a crash, file data
included. A larger `Batch` is a sequence of groups made of whole mutations, so
a crash leaves a prefix of them. A single mutation too large for an empty group
is not atomic, as on `DurabilityStrict`. Volumes without journal writes behave
as in v0.18.0.

On a volume that is not journaled, or whose journal fails the checks, the
counts per commit are:

| Level | First commit | Next, no data | With data | `Sync` | Durable when the mutator returns | Volume between commits |
|---|:--:|:--:|:--:|:--:|:--:|---|
| `DurabilityStrict` (default) | 3 | 3 | 3 | 1 | yes | marked clean |
| `DurabilityMounted` | 2 | 1 | 2 | 1 | yes | marked dirty until `Sync` or `Close` |
| `DurabilityLazy` | 1 | 0 | 0 | 2 | no, only after `Sync` or `Close` | marked dirty until `Sync` or `Close` |

Measured against macOS's own HFS+ driver (v0.12.2, 2026-11-16, 1-minute load
4.19 to 4.75, 30 interleaved rounds, opt-in test `measure_darwin_test.go` with
`HFSPLUS_MEASURE=1`), create + 1 byte + fsync:

| Driver | Time | Against macOS |
|---|--:|--:|
| macOS | 5.13 ms | 1.00x |
| `DurabilityStrict` | 12.66 ms | 2.47x |
| `DurabilityMounted` | 8.20 ms | 1.60x |
| `DurabilityLazy` | 0.095 ms | not durable until `Sync` |

The same 6 barriers over plain fsync(2) cost 0.29 ms, so the gap is the
`F_FULLFSYNC` count.

Re-measured at v0.16.0 (2026-11-21, 1-minute load 4.85 to 5.29, same test, 30
interleaved rounds, medians of rounds 2-30), create + 1 byte + fsync:

| Driver | Time | Barriers |
|---|--:|--:|
| macOS, HFS+ | 5.24 ms | (kernel) |
| macOS, Journaled HFS+ | 9.41 ms | (kernel) |
| `DurabilityStrict`, Journaled HFS+ image | 13.41 ms | 3 |
| `DurabilityMounted` | 8.84 ms | 2 |
| `EnableJournalWrites`, Journaled HFS+ image | **4.37 ms** | 1 |

On a journaled image, journal writes are 3.1x faster than `DurabilityStrict`
and 2.2x faster than macOS's own driver on the same kind of volume.
Metadata-only operations (mkdir, Chtimes) cost about 4.25 ms, the same as
`DurabilityMounted`. This is one run on one machine. The design's 2.5-3 ms
estimate is still not met, because one `F_FULLFSYNC` costs about 4 ms here. The likely reason, not measured: the kernel's journal barrier is a
`DKIOCSYNCHRONIZE` barrier request, cheaper than `F_FULLFSYNC`. Closing it is
[go-filesystems/hfsplus#45](https://github.com/go-filesystems/hfsplus/issues/45);
the design is `docs/design/journal-write.md` in the hfsplus repository.
The v0.18.0 default has not been timed yet: the machine was too loaded
(1-minute load 56 to 81). Its barrier counts are the same as the
`EnableJournalWrites` row above.

- **`DurabilityMounted`** behaves like a mounted HFS+ volume (Apple TN1150) and
  is data-ordered, like ext4 `data=ordered`: a commit that wrote file data puts a
  barrier between the data and the metadata that points at it. A crash between
  syncs leaves the volume marked dirty, so the next mount runs `fsck_hfs`, and a
  journaled volume stays unjournaled until `diskutil enableJournal`. Since
  v0.18.0 this applies only where journal writes are off. On a journaled
  volume, `DurabilityMounted` is a journal commit and the volume is never
  marked dirty.
- **`DurabilityLazy`** is data-writeback: nothing orders file data before its
  metadata, so after a crash a file may hold stale bytes of a deleted one, and
  the mutations since the last `Sync` are lost. (On a journaled volume written
  through its journal, v0.19.0 makes a flushed group one transaction, but the
  mutations since the last flush are still lost.) **Do not use it where different
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
