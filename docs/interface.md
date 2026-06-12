# Interface

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

See the [package reference](https://pkg.go.dev/github.com/go-filesystems/interface) for the
full API, including `DirEntry` and `Stat`.
