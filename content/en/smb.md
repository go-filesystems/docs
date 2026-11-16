---
title: "SMB server"
weight: 25
---

# The SMB server {#smb}

[`go-filesystems/smb`](https://github.com/go-filesystems/smb) is a pure-Go SMB2
server that exports any `Filesystem`, so a disk image can be mounted by the file
manager of Windows, macOS or Linux. Release: **v0.15.0**. This page is written
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

## Measured, and where {#measured}

The README records the macOS kernel client (`mount_smbfs`, which settles on
3.1.1), the Linux kernel client in CI, and go-smb2. The Windows run it reports
predates 3.1.1 and encryption (it negotiated 3.0.2, not encrypted) and has not
been repeated.

Used by [go-fileshare/fileshare](https://github.com/go-fileshare/fileshare) and,
since [theca](https://go-theca.github.io/docs/) v0.5.0, by `theca serve`.
