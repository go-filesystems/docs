---
title: "CFB"
weight: 26
---

# CFB {#cfb}

[`go-filesystems/cfb`](https://github.com/go-filesystems/cfb) (v0.1.0) is a
pure-Go reader for Microsoft Compound File Binary files (MS-CFB, "OLE2"): the
container of `.doc`, `.xls`, `.msg` and `.msi` files and of Prim'X `.zed`
archives. A compound file is a small file system inside a file: storages are
its directories, streams its files. It is read-only, and it is deliberately
**not** a [`Filesystem`]({{< relref "interface.md" >}}): it has no write side
and no mount, so it has no row in the drivers table. It uses only the standard
library.

```go
f, err := cfb.Open(r, size) // r is an io.ReaderAt
if err != nil {             // errors.Is(err, cfb.ErrFormat)
	log.Fatal(err)
}
for _, e := range f.Root().Children { // storages and streams, in tree order
	fmt.Println(e.Name, e.IsDir(), e.Size)
}
e := f.Lookup("Storage/Stream")  // case-insensitive, as MS-CFB orders names
data, err := f.ReadAll(e, 1<<20) // cfb.ErrLimit above 1 MiB
s, err := f.Open(e)              // or an io.ReaderAt: s.ReadAt, s.Size
```

## What it reads {#reads}

- Version 3 (512-byte sectors) and version 4 (4096-byte sectors) files.
- The FAT, the DIFAT (the 109 header entries and the DIFAT sector chain), the
  mini FAT and the mini stream (streams under 4096 bytes).
- The directory: red-black sibling trees walked iteratively, names compared
  as MS-CFB orders them, times, CLSID and state bits.
- Streams as an `io.ReaderAt`.

Writing is out of scope.

## What it refuses {#refuses}

The input is untrusted. Every chain is bounded by the number of sectors the
file can hold, and a sector met twice is a cycle:

- a FAT, DIFAT or mini FAT count above the sector count, a DIFAT chain that
  revisits a sector, a FAT sector number at or beyond the end;
- a directory chain that loops or has no root, a sibling or child pointer out
  of range or reached twice, a bad name length, an unknown object type;
- a stream that needs more sectors than its chain has or than the file holds,
  and a mini stream shorter than its chains say (`ErrFormat`, never a
  zero-filled tail);
- a stream larger than the limit passed to `ReadAll` (`ErrLimit`).

A version 3 stream size keeps its low 32 bits only, as MS-CFB recommends.

## Checks {#checks}

100 % statement coverage, `-race`, s390x under QEMU, `govulncheck` and a
fuzzer with a committed corpus. The tests build files with `cfbtest`, a public
writer of test images (not hardened for untrusted input, not a general MS-CFB
writer); no file written by Word or any Microsoft tool is committed.

The reader was written for [go-fde/zed](https://github.com/go-fde/zed), the
Prim'X ZED! archive reader, and moved here unchanged.
