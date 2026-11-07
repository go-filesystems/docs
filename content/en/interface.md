---
title: "Interface"
weight: 20
---

# Interface {#interface}

Every driver implements the contract defined in
[`go-filesystems/interface`](https://github.com/go-filesystems/interface) (package `filesystem`):

```go
type Filesystem interface {
	Close() error
	ReadFile(path string) ([]byte, error)
	ListDir(path string) ([]DirEntry, error)
	Stat(path string) (Stat, error)
	WriteFile(path string, data []byte, perm os.FileMode) error
	ReadLink(path string) (string, error)
	MkDir(path string, perm os.FileMode) error
	DeleteFile(path string) error
	DeleteDir(path string) error
	Rename(oldPath, newPath string) error
}
```

Optional capabilities are probed via type assertion:

- `Labeller` — volume label get/set.
- `Resizer` — grow/shrink (returns `ErrShrinkUnsupported` where shrink is impossible).
- `TimesReader` (v0.6.0) — the read side of the times `MetadataSetter.Chtimes` writes.

## Timestamps {#timestamps}

`Stat` has mode, size and inode, and no times. `TimesReader` returns all four
POSIX times in one call:

```go
type TimesReader interface {
	Times(path string) (FileTimes, error) // Atime, Mtime, Ctime, Btime
}

if tr, ok := fsys.(filesystem.TimesReader); ok {
	ft, _ := tr.Times("/report.txt")
	mtime := ft.Mtime.Or(time.Now()) // an explicit fallback, never a zero
}
```

Each time is a `Timestamp`, not a `time.Time`:

- **A missing time is not a zero one.** FAT has no change time and several
  formats have no birth time. The zero `Timestamp` means "not recorded", and
  there is no accessor that returns a bare `time.Time` without saying whether
  one exists (`Get() (time.Time, bool)`), so it cannot be written out as 1601
  or 1970 by accident.
- **Resolution is reported** (`Resolution()`): FAT keeps the write time to 2 s,
  the access time to one day and the creation time to 10 ms; HFS+ keeps 1 s;
  NTFS 100 ns; ext4 and the host 1 ns. `Same` compares two times at the coarser
  of the two.
- **One call, not four**: the driver resolves the path once.
- It is on the `Filesystem`, by path, not on `File`: the callers (NFS GETATTR,
  SMB QUERY_DIRECTORY, SFTP stat, WebDAV PROPFIND) have a path, and a `File` is
  a snapshot of the size.

Implemented by [`osfs`](https://github.com/go-filesystems/osfs) v0.4.0 and
[`fat32`]({{< relref "drivers/fat32.md" >}}) v0.12.0; the SMB, NFS, SFTP and
WebDAV servers report them to clients when the driver has them.

See the [package reference](https://pkg.go.dev/github.com/go-filesystems/interface) for the
full API, including `DirEntry` and `Stat`.
