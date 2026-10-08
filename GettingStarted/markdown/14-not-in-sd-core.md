Title: Not in SD Core
Subtitle: What has been removed, why, and what to use instead.

This page exists so you do not spend time hunting for something that is not
there. **It names what is gone and what to use in its place; it does not
document the removed features themselves.**

Everything here was in OpenQM, in ScarletDME, or in SD on Linux, and is not in
SD Core Solo for Windows. **What the multiuser SD Core for Windows has and Solo
does not — accounts, groups, `SDSYS`, the Windows service — is on
[Differences from multiuser SD Core for Windows W1.1-1](01b-differences-from-multiuser-w1-1-1.html)**,
not here.

> **If you had a use for any of these, say so.** Several were removed on the
> reasoning that nothing needed them. That reasoning is worth testing against
> real use.

## Editors

| Gone | Use instead |
|---|---|
| `sed` — the full-screen editor | **`edit`**, or **`ed`** for the line editor |
| `update.record` — the full-screen record editor | **`edit`** or **`ed`** |
| `modify` — the full-screen record editor from OpenQM | **`edit`** or **`ed`** |

**SD Core's own full-screen editing is `edit` and `micro`**, which open the
record in Microsoft Edit and in micro — see
[Programmer commands](07-programmer-commands.html#editors), which also says
what they are good for and what they cannot do. The three above are
gone as *programs*; the capability is not.

`modify` is not in SD Core at all.

**`modify.password` is gone too**: `set.password` changes your account password
— see [The account and its passwords](05-account-types.html).

> **`ed`** was never affected by the keyboard faults that hit the full-screen
> editors — it reads whole lines and goes through the command-line editor. **If
> backspace is ever reported broken in `ed`, that is a new fault, not an old
> one returning.**

## The PROC language

`PROC` is gone. **A VOC item of type `PQ` now reports that PROC is not
supported instead of running.** Your `PQ` records are left alone — it is the
interpreter that has gone, not the records.

**Do not confuse PROC with the query processor.** `LIST`, `COUNT`, `SELECT`
and `SORT` are unaffected. They are a different thing despite the similar name.

## SDNet — remote file access

SD could open a file held on another SD server by putting `server;file` in a
VOC entry. **That is gone. A VOC entry containing a semicolon is now simply a
file name that does not resolve.**

`SET.SERVER`, `DELETE.SERVER` and `LIST.SERVERS` have gone with it. **Two of
those three had never worked in any case** — their VOC entries were malformed.

**Why it went:** each server's user name and password were kept in `sd.conf`,
obscured with a simple letter substitution that is not encryption, and the
session ran over port 4245. There was also no way to switch the feature off —
the `NETFILES` setting was read at start-up and then never consulted.

**The API is not affected.** `SDClient` and the remote API are a separate
mechanism and are unchanged. See [API access](09-api-access.html).

**`NETFILES` is still accepted in `sd.conf`** and does nothing, so an existing
configuration file will not stop SD starting.

## Virtual file systems

**SD has never been able to open a virtual file system.** Nothing in the
file-opening code ever recognised a `VFS:` pathname. What the language carried
was the *outline* of one, and none of it could be reached: a VOC F-pointer
written as `VFS:something` was reported as a virtual file system, passed the
name resolver, and then failed to open with an unrelated error.

All of it has been removed, so the language no longer offers a feature it
cannot perform.

**These names are no longer defined, and a program mentioning one will no
longer compile:**

```
FL$TYPE.VFS      SYSCOM KEYS.H
ER$VFS.NAME      SYSCOM ERR.H
ER$VFS.CLASS     SYSCOM ERR.H
ER$VFS.NGLBL     SYSCOM ERR.H
```

`FTYPE` no longer returns `VFS` for a `VFS:` pathname. **Error numbers 3038,
3039 and 3040 are retired and will not be given a new meaning.**

**IF ONE OF YOUR PROGRAMS REFERS TO ANY OF THESE, it was testing for a state
SD could not reach, and the test can be deleted.**

Two unreachable pieces went with it: `_EXTENDLIST`, which was loaded at every
start-up although nothing ever called it; and the debugger's `(Networked)` file
type, which no file could report once SDNet was gone.

## Language and locale

**SD Core is English only.** `SET.LANGUAGE` and `LOAD.LANGUAGE` do not exist.
`NLS` does: it shows and sets the currency symbol and the thousands and decimal
separators.

## Embedded Python — reversed again, and reversed differently

**Python inside `sd-solo.exe` is dropped, permanently — but calling Python from
SD BASIC is back**, as a separate, native helper process SD talks to over a
pipe rather than a library loaded into the server. The distinction is not
cosmetic: the earlier removal was because Python and the MSYS2 runtime
`sd-solo.exe` is built on cannot share one process safely (§5.3 — `long` is a
different width on each side). The helper avoids that by never being the
same process at all.

**21 `gpl.bp/PY_*` programs** are BASIC-callable (`CALL !PY_CREATEDICT`,
and so on) — there is no TCL verb, so this is a programming capability, not
a command you type at the prompt. A session may start the helper on the same
terms as `OS.EXECUTE`, checked once, at the moment Python starts; on Solo that
is always allowed, because the operating system is yours already — see
[Operating system access](06b-operating-system-access.html). All
twenty-one functions, their arguments and their error codes are in the User
set's *SD BASIC - Python Integration* chapter.

## Field-level encryption

**`encrypt.field` is gone, and with it field-level encryption from TCL.** The
verb is in **no VOC**. While it was still there it could not
have worked: the `$CRYPTO` program behind it is not in the distribution, and
every form of the verb failed at load, before it looked at what you typed.

**Encryption in SD BASIC is unaffected and is the supported route.**
`sdencrypt()` and `sddecrypt()` ship — see *SD Basic - System and Environment*
— and replaced the older `encrypt()` and `decrypt()` functions. What has gone
is the TCL verb that encrypted a field in place, and **nothing replaces that**.

## Configuration items

| Gone | Notes |
|---|---|
| `CREATUSR` | **`config`** no longer lists it; `config('CREATUSR')` returns nothing. A `CREATUSR` line in `sd.conf` is still accepted and ignored |
| `umask` | removed entirely. It controls POSIX file-mode bits, which Windows does not use for security — see [Security](12-security.html#what-ships-secured) for what does the equivalent job here |

## The five programs SD used to ship into the SDSYS BP file

`PCL` and `PCL.GRID` (printer control), `U0032` and `U50BB` (user exits) and
`VFS.CLS` (a template class module) are no longer installed there.

**PCL is unaffected as a printer feature** — the `PCL` keyword and the
catalogued `PCL` routine are both still there. What has gone is a second, older
copy of the source sitting in `BP`.

## Things that were never features, and are not coming

These are not removals. They are stated here because a reader coming from
another MultiValue system will otherwise assume they exist.

**scp AND sftp DO NOT WORK INBOUND ONCE SD HAS CONFIGURED THE ssh SERVER.**
This is the accepted cost of putting every ssh session straight into SD.
**Pull files rather than pushing them** — see
[ssh access](08-ssh-access.html#the-cost-no-scp-or-sftp-to-your-user-and-no-windows-prompt).
A computer with no ssh server is unaffected, because SD has configured nothing.

**The cleartext API login is gone**, and a client that still sends a password
in clear is refused outright. See [API access](09-api-access.html).

## Linux-only mechanisms

The Linux privilege model does not survive the move and has been replaced
rather than emulated. Most of this is invisible unless you are reading source,
but two consequences show:

- **`chmod` does nothing.** The MSYS2 mount is `noacl`, so file-mode bits are
  not a security control here. Windows' own protection of your profile is, and
  SD relies on it.
- **There is no `sdsys` uid to drop to.** SD runs as your Windows user, on an
  ordinary token, and the passwords are the whole of its own authorisation
  model. See [Security](12-security.html).
