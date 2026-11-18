Title: Configuration - The Parameters
Subtitle: Every parameter SD accepts, by subject, and the parameters that were removed.

This page continues [Changing a Parameter from
SD](16b-changing-a-parameter-from-sd.html).

## Sessions and limits

| Parameter | Default | Effect |
|---|---|---|
| `NUMUSERS` | 20 | Maximum concurrent sessions. Sizes the user table in shared memory |
| `NUMFILES` | 80 | Maximum open files across all sessions |
| `NUMLOCKS` | 100 | Maximum record locks across all sessions. Range 1 to 32767 (the lock table's cells are 16 bits; a value outside it stops SD starting) |
| `MAXCALL` | 10000 | Maximum subroutine call depth. Range 10 to 1000000 |
| `CMDSTACK` | 99 | Depth of the command stack |
| `USERPOOL` | 1023 | The highest user number handed out; user numbers rotate through 1 to this, 0 always takes the lowest free one |
| `USPLIT` | 0 | `1`: interactive sessions take the lowest free user number and every other process (a phantom, an API session) the highest free one in 1 to `NUMUSERS`, so the two never meet. `PHANTOMS` and `USERPOOL` are ignored while it is on. `config('USPLIT')` reads it |
| `PHANTOMS` | unset | `base,range` — user numbers kept for phantoms started with `phantom user n`. Nobody else is given one: not a session, not an API session, not a phantom started without `user`. It must lie between 1 and the highest user number, or SD will not start. `config('PHANTOMS')` reads it as `base,range`, null when none are kept |
| `CLEANUP` | 300 | Seconds between the checks that look for sessions that ended without logging out (a killed terminal, a lost connection) and clear them from the user table. 30 to 3600. The daemon wakes every ten seconds, so an interval is kept to within ten seconds; a very short one costs a pass over the user table that often. `config('CLEANUP')` reads it |
| `PWDELAY` | 3 | Seconds SD waits after a failed network sign-on before it answers; 0 removes the pause |
| `NETWAIT` | 5 | Seconds to wait for a connection to a server defined for SDNet (`set.server`, `set.netlogin`, a file opened through an `N` pointer): the TCP connection, then the TLS handshake, each. 1 to 3600; zero is refused |
| `NETREPLY` | 30 | Seconds to wait for each reply of such a server. 1 to 3600; zero is refused. `config('NETWAIT')` and `config('NETREPLY')` read them. See the SDNet page of the User set |
| `FDS` | unset | Limit on file descriptors. No limit when unset |

## Files and locking

| Parameter | Default | Effect |
|---|---|---|
| `GRPSIZE` | 4 | Default group size for a new dynamic file, in 1 KB units. Read by `create.file` and `configure.file` when no group size is given. **4 as shipped**. A Solo installed with an older release keeps the `sd.conf` it has, and so may keep 2 until you change it; a file keeps the size it was made with |
| `MAXIDLEN` | 63 | Maximum record id length. 63 is also the lower bound |
| `MUSTLOCK` | 0 | When 1, a `write` or `delete` requires the record to be locked first |
| `DEADLOCK` | 0 | When 1, SD traps deadlocks |
| `SAFEDIR` | 0 | When 1, directory files are updated by write-and-rename rather than in place |
| `RECCACHE` | 0 | Records cached per file. Range 0 to 32 |
| `FSYNC` | 0 | Bit flags controlling when SD forces data to disk |

## Directories

| Parameter | Default on a new install | Effect |
|---|---|---|
| `SDSYS` | `<installation>\sdsys` | SD's own files. SD does not start if the global catalog is not found beneath it |
| `USRDIR` | `<installation>\user_accounts` | The parent of the account folders. Solo's one account is `user_accounts\sduser` |
| `GRPDIR` | `<installation>\group_accounts` | The parent of group account folders. Solo has none |
| `DUMPDIR` | empty | Where process dumps are written. Empty means the system directory, `sdsys` |
| `TEMPDIR` | empty | Temporary files. Empty means the `TMP` environment variable, or `/tmp` if that is not set |
| `SORTWORK` | empty | Work files for a sort that does not fit in memory. Empty means `TEMPDIR` |
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

## File transfer over ssh

`receive` and `send` (see *Moving Files In and Out* in the User documentation).
Each takes effect when SD is next started, as every
parameter not listed above. None of them is one of the 14 parameters `set.config` changes: `XFER`
is a door, and the limits are added by hand.

| Parameter | Default | Range | Effect |
|---|---|---|---|
| `XFER` | 0 | 0 to 1 | **The system's switch, off as delivered.** 1 lets the account move files with `receive` and `send` over Solo's ssh, once its own switch is on too (`set.xfer.access on`). It opens a door from the network into the account's transfer folders: see [Security](12-security.html) |
| `XFERFILE` | 10240 | 1 to 1048576 | The largest file one transfer takes, in kilobytes; a bigger file is refused and discarded |
| `XFERSLOT` | 102400 | 1 to 4194304 | The most one transfer takes in all its files, in kilobytes |
| `XFERWAIT` | 600 | 10 to 3600 | Seconds a transfer waits for the sftp program's first connection before it closes |
| `XFERIDLE` | 120 | 10 to 3600 | Seconds a transfer stays open after its last file has finished; a file still moving is never cut off |
| `XFERMAX` | 3600 | 0 to 86400 | Seconds a transfer may stay open at most; 0 removes the ceiling |
| `XFERKIND` | blank | | The endings a received file's name must have, separated by commas (`csv,txt`); blank takes any plain-text file |

## The API

| Parameter | Default | Effect |
|---|---|---|
| `APIPORT` | 4249 | Switches the API on. Any number above zero means on, and SD listens on port 4249 whatever the number is; the port cannot be changed. If the line is absent or commented out no socket is created at all, which is how the API is turned off. A file that says `APIPORT=4243` still means on |
| `BACKUPDIR` | unset | The folder `backup.account` and `restore.account` use. Set by `set.backup.directory`, not by hand - see [Backing up and restoring the account](06c-backup-and-restore.html) |
| `NETDIRS` | unset | Directories outside its own account that an API session, or a session held to its account by the `os` switch, may open, separated by semicolons because a Windows path contains a colon. See *`NETDIRS`: directories outside the account*, below |
| `APILOCAL` | 0 | 1 makes the API listener (`APIPORT`) listen on 127.0.0.1 only, so only this computer can reach it; 0 or no line, every address. `remote.api local` writes it and `remote.api open` takes it away. `config('APILOCAL')` reads it; the `config` verb does not print it |
| `SDCLIENT` | 0 | Restricts what an API session may do. Non-zero disables file access outright (no file can be opened, so no dictionary can be evaluated either) and refuses `SDIConv`, `SDOConv`, `SDIConvs` and `SDOConvs` a conversion code that reads a file (`T` with a file) or calls a program (`U`); any non-zero value also refuses `SDOpenSeq`, `SDExecute`, `SDTrans` and `SDRTrans`; `2` additionally refuses any subroutine not compiled as callable from a client (a subroutine whose name begins with `$` must be so compiled at every setting, `0` included) |

Unset is the strict value for `NETDIRS`. With nothing there, an API session can
open files in the account it is standing in and nothing else. It never grants
the credential store, the global catalog or the account register, and naming
those has no effect. A directory listed here is reachable by every API session,
so it is a decision about the computer.

### `NETDIRS`: directories outside the account

```
NETDIRS=D:\Data\Sales;D:\Data\Shared
```

The entries are absolute paths separated by semicolons (a Windows path holds a colon).
**Nothing is listed as delivered.** With the `os` switch off, a session opens no file by
pathname outside the account's own directory; `NETDIRS` names directories it may reach all
the same, to read, change, create and delete, without turning the switch on, which would open
the whole computer. It widens the API session's reach as well as the `os` switch's. `sh` and
`os.execute`, the rule that a VOC changes only after `admin`, and what the `ai` line may send
are separate and unchanged. A session that has typed `admin`, or has the `os` switch on,
reaches every path already. `NETDIRS` is asked last, after the account's own directory.

**Entries SD ignores.** An entry is left out, and named in the error log when SD starts, when
it is not an absolute path, is not an existing directory, is the root of a drive or a share,
or holds or lies inside the SD Core Solo folder, the SD system directory, the directory of
`sd.conf`, the user or group accounts directory (`USRDIR`, `GRPDIR`), or the directory of any
registered account. An account created later inside a listed directory closes that entry at
once. At most 16 entries are taken, and an element longer than a pathname is dropped whole.

**How a path is judged.** The name a program gives is resolved first, and a name that still
holds a `.` or `..` component is refused. It is then compared with each entry ignoring upper
and lower case, and the whole folder name must match: `D:\Data\xkey` is not inside `D:\Data\x`.

SD reads the line when it starts, so a change takes effect the next time SD is started.
**A value longer than 191 characters is refused, by name**, rather than cut, because a cut
path is a shorter directory and would admit more than you wrote. It is not shown by `config()`:
read the file.

`SDCLIENT` has no entry in the shipped `sd.conf`. It defaults to 0, which
permits everything.

## Printing

| Parameter | Default | Effect |
|---|---|---|
| `LPTRWIDE` | 80 | Default page width for the printer. Range 10 to 1000 |
| `LPTRHIGH` | 66 | Default page depth for the printer. Range 10 to 32767 |
| `SPOOLER` | empty | The Windows printer a print job goes to when `setptr` names none (`at` wins); empty is the Windows default printer. On SD Core Solo for Linux it is the spooler command instead |

## Sorting

| Parameter | Default | Effect |
|---|---|---|
| `SORTMEM` | 4096 kb | Above this much data a sort works on disk instead of in memory |
| `SORTMRG` | 4 | Files merged at once in a disk sort. Range 2 to 10 |

## Numbers and dates

| Parameter | Default | Effect |
|---|---|---|
| `INTPREC` | 11 | Digits of precision in integer arithmetic. Range 0 to 14 |
| `FLTDIFF` | 0.00000000002910 | Two floating point numbers closer together than this compare equal |
| `YEARBASE` | 1930 | The century a two-digit year is read into |

## Diagnostics

| Parameter | Default | Effect |
|---|---|---|
| `ERRLOG` | 50 kb | Size of the error log. A non-zero value below 10 KB is raised to 10 KB, and the oldest entries are discarded when it fills |
| `PDUMP` | 0 | Bit flags controlling when a process dump is written |
| `DEBUG` | unset | Bit flags enabling debugging features |
| `OBJECTS` | 0 (no limit) | Compiled programs held in memory at once |
| `OBJMEM` | 0 (no limit) | Memory those programs may occupy, in KB |

`STARTUP` (a command run when SD starts) no longer exists. A
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
[Operating system access](06b-operating-system-access.html). **Both run with
your own Windows permissions, once the `sh` switch is on; as delivered it is off
and `sh` is refused** (`set.os.access sh on`).

## The screen

| Parameter | Default | Effect |
|---|---|---|
| `SCREENTHEME` | unset (`mono`); `paper` in the shipped `sd.conf` | The color theme that a program using the `SCREEN` statement gets when it does not choose one: `paper`, `night`, `mono`, `classic` or `blue`. `paper` is black and dark blue text on light gray, with the menu bar and menus the other way round (light gray on black); `night` is white and yellow text on black, its menus black on light gray; `mono` uses only reverse video, underline and bold and reads on any terminal. The shipped `sd.conf` says `SCREENTHEME=paper`, so the toolkit pages, the IDE, SED and the debugger show paper on a color terminal on a fresh install. An upgrade adds `SCREENTHEME=paper` to a kept `sd.conf` that has no `SCREENTHEME` line, and leaves a line that is there alone. A terminal with no color (vt100, a phantom, a pipe) is `mono` whatever the line says, and so is a name that is not a theme, without an error. A program that asks for a theme gets that one, and reads the value with `config('SCREENTHEME')`. **Your own choice wins**: Ctrl-T or View > Theme on any full-screen program keeps the theme chosen, and every full-screen program starts in it from then on; `SCREENTHEME` is what you see until you have chosen |

## Parameters that were removed

Thirteen parameters of older releases do not exist. Seven did nothing (`JNLMODE`, `JNLDIR`,
`CREATUSR`, `CODEPAGE`, `EXCLREM`, `RINGWAIT`, `TXCHAR`), and six more imitated
another product or chose between ways of working that SD does not have
(`APILOGIN`, `FILERULE`, `FIXUSERS`, `GDI`, `INVCASE`, `PORTMAP`). See [A name SD does not recognize stops it starting](16-configuration.html#a-name-sd-does-not-recognize-stops-it-starting)
for what happens to a kept `sd.conf` that still names one. The `PATH:` form
of a file name went with `FILERULE`: write an F-pointer instead.

`NETFILES` is a live parameter. A `server;file` VOC reference is refused
whatever it holds, but **bit 2 (`NETFILES=2`) opens API request 29**, which opens
an `account;file` file for a client that has signed in. It is 0 as delivered. With
it set, the request is still refused unless the account's NET gate is on
(`set.net.access on`, an administrator command, off as delivered), and under
`SDCLIENT`; see [API access](09-api-access.html). A pointer in the account's VOC
that leads out of the account is refused too, unless the account has the `os`
switch on. Setting `NETFILES=2` is therefore not enough to open such a file: turn
the NET gate on only knowing what it allows. When it opens the file, the
connection's containment applies: only the account the session stands in, the
shipped system files its VOC names and the directories named in `NETDIRS`.
The SDNet **client** (`set.server`, `set.netlogin`, a VOC pointer of
type `N`) does not read `NETFILES`; see the SDNet page of the User set.

There is no license parameter and no license verb. SD Core Solo for Windows is
GPL v3 and is not licensed per site or per user; `config gpl` displays the
license text.
