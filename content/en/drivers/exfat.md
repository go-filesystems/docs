---
title: "exfat"
weight: 60
---

# exfat {#exfat}

<img src="../../assets/fs/go-filesystems-exfat.png" width="110" align="right">

Extended FAT for large media.

Implements [`filesystem.Filesystem`]({{< relref "../interface.md" >}}). No cgo, no root.

## Status {#status}

| Read | Write | Format | Label | Symlinks | On-disk format |
|:--:|:--:|:--:|:--:|:--:|---|
| ✅ | ✅ | ✅ | ✅ | ✕ | exFAT |

## Supported {#supported}

- Read/write files
- Directories, recursive delete, rename
- Bare images + MBR/GPT

## Not implemented {#not-implemented}

- No POSIX symlinks, permissions or ACLs (not in exFAT)

## Untrusted input {#untrusted-input}

Every allocation a value in the image can size is bounded before it happens: a
cluster buffer is allocated only after the cluster's last byte has been read, and
a directory is at most 256 MiB. A hostile image gets an error, not an
out-of-memory kill.

## Install {#install}

```bash
go get github.com/go-filesystems/exfat
```

- Source: <https://github.com/go-filesystems/exfat>
- API reference: <https://pkg.go.dev/github.com/go-filesystems/exfat>

{{< callout type="info" >}}
See the module's README for full, up-to-date details.
{{< /callout >}}
