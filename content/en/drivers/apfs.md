---
title: "apfs"
weight: 40
---

# apfs {#apfs}

<img src="../../assets/fs/go-filesystems-apfs.png" width="110" align="right">

Apple File System.

Implements [`filesystem.Filesystem`]({{< relref "../interface.md" >}}). No cgo, no root.

## Status {#status}

| Read | Write | Format | Label | Symlinks | On-disk format |
|:--:|:--:|:--:|:--:|:--:|---|
| ✅ | ✅ | ✅ | — | ✅ | Real APFS on-disk (kext-mountable), GPT-aware |

## Supported {#supported}

- Real APFS read: FS-tree traversal, multi-extent files, xattrs, sealed volumes
- Snapshot read (`OpenSnapshot`)
- Write & format kext-mountable containers
- FileVault software encryption: write (`FormatContainerEncrypted`) and,
  since v0.4.0, read-only open (`OpenFDE`, `OpenFDEFromBlockDevice`, and the
  `…Limited` variants that bound key derivation for untrusted images).
  Only the FS-tree nodes flagged `OMAP_VAL_ENCRYPTED` and the file extents
  with a `crypto_id` are decrypted; the container metadata is read as the
  plaintext it is. A wrong passphrase (`ErrWrongPassphrase`) and a refused
  keybag (`ErrLimit`) are told apart with `errors.Is`.
- GPT + Apple DMG output
- POSIX special files
- Container resize (`Resize`)

## Not implemented {#not-implemented}

- Snapshot creation gated behind an Apple-private entitlement
- Some encrypted-container fsck stages are Apple-private
- Writing to a FileVault volume (it opens read-only); per-file-key
  (hardware) encryption

## Install {#install}

```bash
go get github.com/go-filesystems/apfs
```

- Source: <https://github.com/go-filesystems/apfs>
- API reference: <https://pkg.go.dev/github.com/go-filesystems/apfs>

{{< callout type="info" >}}
See the module's README for full, up-to-date details.
{{< /callout >}}
