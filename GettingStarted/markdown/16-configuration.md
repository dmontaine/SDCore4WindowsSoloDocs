Title: Configuration
Subtitle: The sd.conf file, the two ways to read a setting, and every parameter SD accepts.

SD reads one configuration file at start-up. It sizes the shared memory
segment, sets the limits every session inherits, and names the directories SD
writes to.

SD folds case, so a command may be typed in either case. Commands are shown
here in lower case.

> **How this page was checked:** the parameter names, counts, defaults and
> ranges below were read from SD's source (`config.c`, `op_config.c`), which is
> the same code as the multiuser SD Core for Windows apart from where the
> folders default to. The `config` listing shows the form the verb prints; it
> is abbreviated.

## The file

```
%USERPROFILE%\SDCoreSolo\sd.conf
```

**SD finds it beside its own installation**, so nothing needs setting. The
server and the client both read the `SD_CONFIG` environment variable first and,
if it is not set, use `sd.conf` in the `SDCoreSolo` folder. The file is
installed only if it does not already exist and is marked never to uninstall,
so edits to it survive an upgrade and survive removal of the product.

It is plain text in one section. **There is no path in it:** `SDSYS`, `USRDIR`
and `GRPDIR` default to folders in the installation's own folder, which is why
a whole `SDCoreSolo` folder can be moved.

```
[sd]
GRPSIZE=2
NUMUSERS=20
SORTMEM=4096
ERRLOG=50
APILOGIN=1
APIPORT=4249
SH=C:/Windows/System32/WindowsPowerShell/v1.0/powershell.exe -NoProfile -NoLogo
SH1=C:/Windows/System32/WindowsPowerShell/v1.0/powershell.exe -NoProfile -NonInteractive -ExecutionPolicy Bypass -Command
```

**`APIPORT` is active only if the API was chosen at installation.** A computer
installed without it has the same file with the line commented out
(`# APIPORT=4249`), and so no API listener. A managed computer needs it, and
chooses it like any other. `solo-api-listener.ps1` switches it; see
[API access](09-api-access.html).

Lines beginning `#` are comments. The shipped file is heavily commented and
those comments record why each value was chosen. Read them before changing
anything.

### A name SD does not recognise stops it starting

An unrecognised parameter is not ignored. The parser abandons the file with
`Unrecognised configuration parameter`, and SD does not start.

That is why obsolete names are still parsed rather than deleted. `CREATUSR` has
done nothing since 14 August 2026, but a configuration file copied from a Linux
installation still carries it, and removing the branch would turn a tidy-up
into a failure to start.

Range failures behave the same way. `GRPSIZE`, `INTPREC`, `LPTRHIGH`,
`LPTRWIDE`, `MAXCALL`, `RECCACHE`, `SORTMRG` and `MAXIDLEN` are bounds-checked
once the file has been read, and a value outside its range stops start-up with
a message naming the parameter.

## Reading the settings

**The `config` verb reports what is in force, and needs `ADMIN` first** —
`config gpl` and `config contrib` are the exceptions. See
[Administrator commands](06-administrator-commands.html).

```
:config
Virtual Machine Version Number WS1.1-3
APILOGIN  1
APIPORT   4249
CMDSTACK  99
DEADLOCK  0
...
YEARBASE  1930
```

SD accepts **52** parameters and the verb prints **43** of them. Nine are
accepted in the file and never displayed: `CODEPAGE`, `CREATUSR`, `DEBUG`,
`FDS`, `FIXUSERS`, `NETDIRS`, `PORTMAP`, `SDSYS` and `TXCHAR`.

`NETDIRS` is the one to know about. It decides what an API session may reach
outside its own account, and the verb will not tell you what it is set to. Read
it from the file.

The `config()` function reads one parameter from a program, and needs no
`ADMIN`:

```
group.size = config('GRPSIZE')
```

**49 parameters are readable this way.** The three that are not are `CREATUSR`,
which stores nothing, `SDSYS`, and `TXCHAR`. The name is truncated to eight
characters before it is matched; no parameter name is longer than eight, so
that only shows up if you pass something which is not a parameter.

Two further forms exist. `config lptr` reports the settings of the default
printer, and `config gpl` and `config contrib` display the licence and the list
of contributors.

## Changing a parameter

Editing `sd.conf` and restarting SD is the durable route, and for most
parameters it is the only one.

**28 parameters can also be changed for the current session:**

```
config sortmem 8192
```

That writes to the session's own copy of the settings, taken from shared memory
when the session started. It does not reach `sd.conf`, it does not affect any
other session, and it is gone when the session ends.

The 28 are `CODEPAGE`, `DUMPDIR`, `EXCLREM`, `FILERULE`, `FLTDIFF`, `FSYNC`,
`GDI`, `GRPSIZE`, `INTPREC`, `LPTRHIGH`, `LPTRWIDE`, `MAXCALL`, `MUSTLOCK`,
`OBJECTS`, `OBJMEM`, `RECCACHE`, `RINGWAIT`, `SAFEDIR`, `SDCLIENT`, `SH`,
`SH1`, `SORTMEM`, `SORTMRG`, `SORTWORK`, `SPOOLER`, `TEMPDIR`, `TERMINFO` and
`YEARBASE`.

Everything else takes effect only when SD is next started. That includes every
limit which sizes the shared memory segment and every setting the API listener
reads.

## Sessions and limits

| Parameter | Default | Effect |
|---|---|---|
| `NUMUSERS` | 20 | Maximum concurrent sessions. Sizes the user table in shared memory |
| `NUMFILES` | 80 | Maximum open files across all sessions |
| `NUMLOCKS` | 100 | Maximum record locks across all sessions |
| `MAXCALL` | 10000 | Maximum subroutine call depth. Range 10 to 1000000 |
| `CMDSTACK` | 99 | Depth of the command stack |
| `FDS` | unset | Limit on file descriptors. No limit when unset |
| `FIXUSERS` | unset | `base,range` — user numbers reserved for sessions that ask for a specific number |
| `PORTMAP` | unset | `base_port,base_user,range` — gives a session a fixed user number derived from the port its connection arrived on. Refused if the range overlaps `FIXUSERS` |

## Files and locking

| Parameter | Default | Effect |
|---|---|---|
| `GRPSIZE` | 2 | Default group size for a new dynamic file, in 1 KB units. Read by `create.file` and `configure.file` when no group size is given |
| `MAXIDLEN` | 63 | Maximum record id length. 63 is also the lower bound |
| `MUSTLOCK` | 0 | When 1, a `write` or `delete` requires the record to be locked first |
| `DEADLOCK` | 0 | When 1, SD traps deadlocks |
| `SAFEDIR` | 0 | When 1, directory files are updated by write-and-rename rather than in place |
| `RECCACHE` | 0 | Records cached per file. Range 0 to 32 |
| `FSYNC` | 0 | Bit flags controlling when SD forces data to disk |
| `FILERULE` | 0 | Bit flags deciding which special VOC file references are honoured. Bit 4 allows a `PATH:` reference to name a pathname directly |

## Directories

| Parameter | Default | Effect |
|---|---|---|
| `SDSYS` | `<installation>\sdsys` | SD's own files. SD does not start if the global catalogue is not found beneath it |
| `USRDIR` | `<installation>\user_accounts` | The parent of the account folders. Solo's one account is `user_accounts\sduser` |
| `GRPDIR` | `<installation>\group_accounts` | The parent of group account folders. Solo has none |
| `DUMPDIR` | empty | Where process dumps are written. Empty means the system directory, `sdsys` |
| `TEMPDIR` | empty | Temporary files. Empty means the `TMP` environment variable, or `/tmp` if that is not set |
| `SORTWORK` | empty | Work files for a sort that does not fit in memory. Empty means `TEMPDIR` |
| `JNLDIR` | empty | Journal directory |
| `TERMINFO` | empty | An additional terminfo directory. The shipped definitions are found without it |

`<installation>` is the `SDCoreSolo` folder, found at run time from where SD's
programs are.

**A process dump carries the whole variable state of the session that wrote
it**, passwords a program was holding included. With `DUMPDIR` empty it goes
into `sdsys`, in your own profile; Windows' protection of that profile is what
keeps it from other Windows users. A dump is a file you own — treat it as
sensitive, and point `DUMPDIR` somewhere else if you want it kept apart.

`TEMPDIR` and `SORTWORK` are reported in POSIX form because that is how the
server's runtime addresses them. A Windows path such as
`/cygdrive/c/Users/you/AppData/Local/Temp` is the same directory as its
`C:\Users\you\AppData\Local\Temp` spelling. **A directory named here that does
not exist is ignored**, and the default applies.

## The API

| Parameter | Default | Effect |
|---|---|---|
| `APIPORT` | 4249 | Switches the API on. Any number above zero means on, and SD listens on port 4249 whatever the number is; the port cannot be changed. If the line is absent or commented out no socket is created at all, which is how the API is turned off. A file that says `APIPORT=4243` still means on |
| `BACKUPDIR` | unset | The folder `BACKUP.ACCOUNT` and `RESTORE.ACCOUNT` use. Set by `SET.BACKUP.DIRECTORY`, not by hand - see [Backing up and restoring the account](06c-backup-and-restore.html) |
| `APILOGIN` | 1 | Accepted and does nothing. The API always requires the account's password, whatever this says |
| `NETDIRS` | unset | Directories outside its own account an API session may open, separated by semicolons because a Windows path contains a colon |
| `SDCLIENT` | 0 | Restricts what an API session may do. Non-zero disables file access outright; `2` additionally refuses any subroutine not compiled as callable from a client |

Unset is the strict value for `NETDIRS`. With nothing there, an API session can
open files in the account it is standing in and nothing else. It never grants
the credential store, the global catalogue or the account register, and naming
those has no effect. A directory listed here is reachable by every API session,
so it is a decision about the computer.

`SDCLIENT` is not reported by the `config` verb and has no entry in the shipped
`sd.conf`. It defaults to 0, which permits everything.

## Printing

| Parameter | Default | Effect |
|---|---|---|
| `LPTRWIDE` | 80 | Default page width for the printer. Range 10 to 1000 |
| `LPTRHIGH` | 66 | Default page depth for the printer. Range 10 to 32767 |
| `SPOOLER` | empty | The spooler to hand print jobs to |
| `GDI` | 0 | Selects the Windows printing interface used by default |

## Sorting

| Parameter | Default | Effect |
|---|---|---|
| `SORTMEM` | 4096 kb | Above this much data a sort works on disk instead of in memory. The shipped `sd.conf` sets 4096; with no line the built-in value is 1024 |
| `SORTMRG` | 4 | Files merged at once in a disk sort. Range 2 to 10 |

## Numbers and dates

| Parameter | Default | Effect |
|---|---|---|
| `INTPREC` | 13 | Digits of precision in integer arithmetic. Range 0 to 14 |
| `FLTDIFF` | 0.00000000002910 | Two floating point numbers closer together than this compare equal |
| `YEARBASE` | 1930 | The century a two-digit year is read into |

## Diagnostics

| Parameter | Default | Effect |
|---|---|---|
| `ERRLOG` | 50 kb | Size of the error log. A non-zero value below 10 KB is raised to 10 KB, and the oldest entries are discarded when it fills |
| `PDUMP` | 0 | Bit flags controlling when a process dump is written |
| `DEBUG` | unset | Bit flags enabling debugging features |
| `JNLMODE` | 0 | Journalling mode |
| `OBJECTS` | 0 (no limit) | Compiled programs held in memory at once |
| `OBJMEM` | 0 (no limit) | Memory those programs may occupy, in KB |

`STARTUP` (a command run when SD starts) was removed in WS1.1-3. It never ran. A
line that still has it stops SD from starting, with a message that says to remove
the line.

## The shell

| Parameter | Default | Effect |
|---|---|---|
| `SH` | PowerShell, interactive | The shell a bare `sh` starts |
| `SH1` | PowerShell, non-interactive | The shell `sh command` uses, and the one SD uses for its own PowerShell scripts |

Both are full paths on a real install:

```
SH        C:/Windows/System32/WindowsPowerShell/v1.0/powershell.exe -NoProfile -NoLogo
SH1       C:/Windows/System32/WindowsPowerShell/v1.0/powershell.exe -NoProfile -NonInteractive -ExecutionPolicy Bypass -Command
```

The difference between them matters. `SH1` carries `-NonInteractive`, and
`-ExecutionPolicy Bypass` so the scripts SD installed run on a stock Windows
whose default policy is Restricted; `SH` carries neither, so a bare `sh` in a
phantom or a scheduled job hands control to a shell with nobody at the
keyboard, under your computer's own execution policy. Operating system access
has its own page in this set — see
[Operating system access](06b-operating-system-access.html).

## Parameters that are accepted and do nothing

These are parsed so that an existing `sd.conf` still loads. None of them
changes SD's behaviour, and none should be offered as a control.

| Parameter | Why it is inert |
|---|---|
| `NETFILES` | SDNet was removed from this port. `netfiles.c` is deleted, a `server;file` VOC reference is refused, and the request that reports open SDNet connections returns an empty list. The value is still stored and still reported, and setting it opens nothing |
| `CREATUSR` | Nothing creates operating system accounts from `sd.conf`. The value is discarded as it is read |
| `CODEPAGE` | Stored, readable and settable. Nothing acts on it |
| `EXCLREM` | Stored, readable and settable. It described exclusive access to a remote file, and there are no remote files |
| `RINGWAIT` | Stored, readable and settable. Nothing acts on it |
| `TXCHAR` | Stored, and not readable — there is no `config('TXCHAR')` |

`FILERULE` is **not** in this list, and is easy to mistake for it. Its remote
bits died with SDNet, but bit 4 is live and decides whether a `PATH:` VOC
reference may name a pathname directly.

There is no licence parameter and no licence verb. SD Core Solo for Windows is
GPL v3 and is not licensed per site or per user; `config gpl` displays the
licence text.
