---
title: "SMB server"
weight: 25
---

# The SMB server {#smb}

[`go-filesystems/smb`](https://github.com/go-filesystems/smb) is a pure-Go SMB2
server that exports any `Filesystem`, so a disk image can be mounted by the file
manager of Windows, macOS or Linux. Release: **v0.17.2** (the README below is
at v0.15.0). This page is written
from the [README](https://github.com/go-filesystems/smb/blob/v0.15.0/README.md)
at that tag and the release notes of v0.14.0 and v0.15.0; where it differs from
the repository, the repository wins.

## What v0.13.0 to v0.15.0 changed {#v015}

- **SMB 3.1.1**: negotiate contexts, the pre-authentication integrity hash over
  NEGOTIATE and SESSION_SETUP, and signing with AES-GMAC when the client offers
  it. Dialects are 2.1, 3.0, 3.0.2 and 3.1.1.
- **Encryption**: AES-128-CCM on 3.0 and 3.0.2; AES-128-GCM, AES-128-CCM,
  AES-256-GCM and AES-256-CCM on 3.1.1. It is offered by default and **required**
  per share with `smb.RequireEncryption()` or for every session with
  `Server.EncryptData`.
- **Signing required**: `Server.RequireSigning` makes the server insist on
  signing for every session, whether or not the client asked.
- **Replay window** (v0.14.0): a command sequence window, so a captured signed or
  encrypted frame sent again is refused.
- **Signing enforced** (v0.14.0): a session that requires signing refuses an
  unsigned request, and on 3.1.1 a TREE_CONNECT that is neither signed nor
  encrypted ends the connection.
- **Session-hijack fix** (v0.15.0): the logins in progress are keyed by the
  session id the server allocated, not one per connection, so an injected
  SESSION_SETUP cannot overwrite another's exchange.
- **Connection limits** (v0.15.0): `Server.MaxConnections`,
  `Server.MaxPreAuthConnections`, `Server.PreAuthTimeout` and
  `Server.IdleTimeout` bound the connections and what an unauthenticated one may
  hold.

## v0.16.0 and v0.16.1: per-source caps {#v016}

- **v0.16.0**: the connections one source may hold are capped.
- **v0.16.1**: a zoned link-local source is keyed by its /64.

Release notes: [v0.16.0](https://github.com/go-filesystems/smb/releases/tag/v0.16.0),
[v0.16.1](https://github.com/go-filesystems/smb/releases/tag/v0.16.1). The values
used by theca v0.6.1 (32 per source, 16 pre-auth; loopback is not exempt) are
from its CHANGELOG.

## v0.17.2: a handle belongs to its session and tree {#v0172}

A file handle belongs to the session and tree that opened it. Before v0.17.2 a
second session on the same connection could READ, WRITE and CLOSE another
session's open file without access to its share. Now READ, WRITE, CLOSE, FLUSH,
LOCK, IOCTL, QUERY_DIRECTORY, CHANGE_NOTIFY, QUERY_INFO and SET_INFO answer
`STATUS_FILE_CLOSED` to any other session or tree (MS-SMB2 3.3.5.10 to 3.3.5.13);
the all-ones FileId no longer resolves to a connection-wide last file, and the
related-operation FileId (3.3.5.2.7.2) lives in the compound chain only. FLUSH of
an unknown handle is `STATUS_FILE_CLOSED` instead of success. Release notes:
[v0.17.2](https://github.com/go-filesystems/smb/releases/tag/v0.17.2).

## Measured, and where {#measured}

The README records the macOS kernel client (`mount_smbfs`, which settles on
3.1.1), the Linux kernel client in CI, and go-smb2. The Windows run it reports
predates 3.1.1 and encryption (it negotiated 3.0.2, not encrypted) and has not
been repeated.

Used by [go-fileshare/fileshare](https://github.com/go-fileshare/fileshare) and,
since [theca](https://go-theca.github.io/docs/) v0.5.0, by `theca serve`.
