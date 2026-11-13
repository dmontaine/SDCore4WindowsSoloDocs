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
GRPSIZE=4
NUMUSERS=20
SORTMEM=4096
ERRLOG=50
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

That is why a name SD has dropped has to be taken out of the file. On 8 November
2026 twelve parameters were removed (`APILOGIN`, `CODEPAGE`, `CREATUSR`,
`EXCLREM`, `FIXUSERS`, `GDI`, `JNLDIR`, `JNLMODE`, `PORTMAP`, `RINGWAIT`,
`TXCHAR` and `FILERULE`), and `INVCASE` before them. The installer removes those
lines from an `sd.conf` it keeps across a re-install or an upgrade. A file copied
in by hand must have them taken out before SD will start.

Range failures behave the same way. `GRPSIZE`, `INTPREC`, `LPTRHIGH`,
`LPTRWIDE`, `MAXCALL`, `RECCACHE`, `SORTMRG` and `MAXIDLEN` are bounds-checked
once the file has been read, and a value outside its range stops start-up with
a message naming the parameter.

## Reading the settings

**The `config` verb reports what is in force, and needs `admin` first** —
`config gpl` and `config contrib` are the exceptions. See
[Administrator commands](06-administrator-commands.html).

```
:config
Virtual Machine Version Number WS1.1-3
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
`admin`:

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
parameters it is the only one. Fourteen tuning parameters can also be changed
from SD itself, with `set.config` or the `E` key of `tool.config` (see
*Changing a parameter from SD*, below).

**23 parameters can also be changed for the current session:**

```
config sortmem 8192
```

That writes to the session's own copy of the settings, taken from shared memory
when the session started. It does not reach `sd.conf`, it does not affect any
other session, and it is gone when the session ends.

The 23 are `DUMPDIR`, `FLTDIFF`, `FSYNC`, `GRPSIZE`, `INTPREC`,
`LPTRHIGH`, `LPTRWIDE`, `MAXCALL`, `MUSTLOCK`, `OBJECTS`, `OBJMEM`, `RECCACHE`,
`SAFEDIR`, `SDCLIENT`, `SH`, `SH1`, `SORTMEM`, `SORTMRG`, `SORTWORK`,
`SPOOLER`, `TEMPDIR`, `TERMINFO` and `YEARBASE`.

Everything else takes effect only when SD is next started (`sd-solo -stop`, then
`sd-solo -start`). That includes every limit which sizes the shared memory
segment and every setting the API listener reads.

**In Solo every change to `sd.conf` waits for the next start.** SD reads the file
once, when it starts, and keeps the settings in shared memory; a new session copies
them from there and does not read the file itself. (The multiuser SD Core has each
session read `sd.conf` when it starts, so there an edit reaches the sessions
started after it.) So `tool.config` says `restart` for every parameter it can
change, and `set.config` says so after each change.

## The parameters on one screen: `tool.config`

```
tool.config
```

A full-screen page for the administrator (`admin` first; a window of 140 columns by 43 lines or
more). **It looks, and it can change 14 of the parameters**; it writes no file itself. A change is
made by the verb `set.config` (below), and only after you have typed a value, the page has checked
it against the range SD enforces and a box has shown the exact command and been answered Run. The
other parameters are not changed from SD; editing the file and restarting SD stays the way to change
them.

The page has three views. **P, the parameters**: one row for every parameter SD knows, from the table
`config.table` in the `sdsys` folder, which names every parameter SD reads. The columns are the value
this SD is running with (what `config()` reports), the value `sd.conf` says now (the file is read
again, a parameter written twice counted the way SD counts it, the flag parameters added up), the
default (what SD uses when the file does not say), when a change takes effect and whether anything
acts on it. A row marked `*` is one where the file and the running SD disagree. **A name in the file
that SD does not know is listed at the end as `unknown`, and the top of the page names it**: SD does
not start with it (the section *A name SD does not recognise stops it starting* above). `D` shows only
the rows where the file and SD disagree, and `S` only the parameters the file sets. The last column,
`Change`, says `e` for a parameter the page can change (press `E`) and `by hand` for every other.

**B, the backup directory**: the directory saved with `set.backup.directory` and whether it is there;
**R, the report**: the operating system part of `settings.report`, made when the view is opened and
on F5. It holds no key and no password hash. `F5` reads the file, the backup directory and the report
again; `E` changes the parameter under the cursor, or says why it cannot be changed here; `F1`
explains the page; `Ctrl-Q` leaves. Opening the page writes the audit line
`tool start app=tool.config`; a change writes the line of `set.config` (below), and nothing else the
page does is written.

### Changing a parameter from SD: `set.config` and the `E` key

```
set.config cleanup 600
```

The administrator's verb (`admin` first). It changes **one** line of `sd.conf` in the Solo folder
(`%USERPROFILE%\SDCoreSolo\sd.conf`): the first word is a parameter name (letters and digits; SD makes
it capitals, as the file spells it) and the second a whole number of up to seven digits. It runs
`sd-confset.ps1`, in the Solo folder beside `sd.conf`, as you, since the file is yours; nothing runs
elevated. The script checks everything again and is the authority. `set.config` prints
`CLEANUP is now 600 in sd.conf` and that it takes effect when SD is next stopped and started. A
refusal prints the script's own sentence after `sd.conf was not changed:` and writes nothing.

**What the script does.** It accepts only the 14 parameters below, and only a value inside the range
SD itself enforces for that parameter when SD starts (so it cannot accept what SD would refuse). It
adds or replaces the line **inside the `[sd]` section**, the only place SD reads; a second line for
the same parameter in that section is dropped, because SD keeps the last one; every other line,
comments included, is kept exactly as it was, with the file's own line ends. The new file is written
beside the old one and put in its place by one Windows replace, which keeps the old file's
permissions. **The file as it was is kept as `sd.conf.bak`** (one generation; a second change
replaces it with the file as it was before that change). A change that says what the file already says
writes nothing. A file holding anything but plain ASCII is refused, not rewritten. When the session
reads another file (the `SD_CONFIG` variable), the page says so in its box: `set.config` changes the
Solo folder's `sd.conf` only. SD has no key that puts `sd.conf.bak` back; from PowerShell,
`Copy-Item $env:USERPROFILE\SDCoreSolo\sd.conf.bak $env:USERPROFILE\SDCoreSolo\sd.conf` does.

**One audit line** for each change asked for: `set.config key=cleanup value=600 status=ok`, or
`status=refused`.

**On the page.** `E` on a row whose `Change` column says `e` asks for the new value (the question
shows what SD runs with and what the file says), checks that it is a whole number inside the range
before anything else happens, and then shows the exact command in the toolkit's box with a plain
sentence about when it takes effect. Cancel, which is what Enter chooses, runs nothing. `E` on any
other row writes nothing: it shows why the parameter is not changed from SD, and the exact line to
add by hand in the `[sd]` section.

**The 14 parameters** are the tuning ones: each is range-checked by SD and each only sizes or times
something SD does; none is a door, a path, a command, a log or a security setting:

| Parameter | Range |
|---|---|
| `CLEANUP` | 30 to 3600 |
| `DHCACHE` | 0 to 1024 |
| `GRPSIZE` | 1 to 8 |
| `LGNWAIT` | 0 to 32767 |
| `LPTRHIGH` | 10 to 32767 |
| `LPTRWIDE` | 10 to 1000 |
| `MAXCALL` | 10 to 1000000 |
| `MAXRLOCK` | 0 to 100000 |
| `NETREPLY` | 1 to 3600 |
| `NETWAIT` | 1 to 3600 |
| `NUMLOCKS` | 1 to 32767 |
| `RECCACHE` | 0 to 32 |
| `SORTMRG` | 2 to 10 |
| `USPLIT` | 0 to 1 |

## Sessions and limits

| Parameter | Default | Effect |
|---|---|---|
| `NUMUSERS` | 20 | Maximum concurrent sessions. Sizes the user table in shared memory |
| `NUMFILES` | 80 | Maximum open files across all sessions |
| `NUMLOCKS` | 100 | Maximum record locks across all sessions |
| `MAXCALL` | 10000 | Maximum subroutine call depth. Range 10 to 1000000 |
| `CMDSTACK` | 99 | Depth of the command stack |
| `NETWAIT` | 5 | Seconds to wait for a connection to a server defined for SDNet (`set.server`, `set.netlogin`, a file opened through an `N` pointer): the TCP connection, then the TLS handshake, each. 1 to 3600; zero is refused |
| `NETREPLY` | 30 | Seconds to wait for each reply of such a server. 1 to 3600; zero is refused. `config('NETWAIT')` and `config('NETREPLY')` read them. See the SDNet page of the User set |
| `FDS` | unset | Limit on file descriptors. No limit when unset |
| `USERPOOL` | 1023 | The highest user number handed out; user numbers rotate through 1 to this, 0 always takes the lowest free one |
| `USPLIT` | 0 | `1`: interactive sessions take the lowest free user number and every other process (a phantom, an API session) the highest free one in 1 to `NUMUSERS`, so the two never meet. `PHANTOMS` and `USERPOOL` are ignored while it is on. `config('USPLIT')` reads it |
| `PHANTOMS` | unset | `base,range` — user numbers kept for phantoms started with `phantom user n`. Nobody else is given one: not a session, not an API session, not a phantom started without `user`. It must lie between 1 and the highest user number, or SD will not start. `config('PHANTOMS')` reads it as `base,range`, null when none are kept |
| `CLEANUP` | 300 | Seconds between the checks that look for sessions that ended without logging out (a killed window, a lost connection) and clear them from the user table. 30 to 3600. `config('CLEANUP')` reads it |
| `PWDELAY` | 3 | Seconds SD waits after a failed network sign-on before it answers; 0 removes the pause |

## Files and locking

| Parameter | Default | Effect |
|---|---|---|
| `GRPSIZE` | 4 | Default group size for a new dynamic file, in 1 KB units. Read by `create.file` and `configure.file` when no group size is given. **4 since 12 Nov 2026** (the shipped `sd.conf` said 2). A Solo that is already installed keeps the `sd.conf` it has, and so keeps 2 until you change it; a file keeps the size it was made with |
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
| `BACKUPDIR` | unset | The folder `backup.account` and `restore.account` use. Set by `set.backup.directory`, not by hand - see [Backing up and restoring the account](06c-backup-and-restore.html) |
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

## The screen

| Parameter | Default | Effect |
|---|---|---|
| `SCREENTHEME` | unset (`mono`); `paper` in the shipped `sd.conf` | The color theme that a program using the `SCREEN` statement gets when it does not choose one: `paper`, `night`, `mono`, `classic` or `blue`. The shipped `sd.conf` says `SCREENTHEME=paper`, so the toolkit pages, the IDE, SED and the debugger show paper on a fresh install. A name that is not a theme is `mono`, without an error. A program reads the value with `config('SCREENTHEME')`. **Your own choice wins**: Ctrl-T or View > Theme on any full-screen program keeps the theme chosen |

## `NETFILES`

A `server;file` VOC reference is
refused whatever it holds (the old SDNet is gone), and the value is parsed,
stored, shown by `config` and readable with `config('NETFILES')`. **One thing
still reads it:** the API server's request 29, "open an SDNet file", which
opens the file that a VOC entry of the named account names, for a client that
has signed in. It is refused unless bit 2 of `NETFILES` is set. The delivered
value is 0, so as delivered the request is refused; leave it at 0 unless an API
client needs it. With bit 2 set the request is still refused unless the
account's NET gate is on (`set.net.access on`, an administrator command, off as
delivered), and under `SDCLIENT`; a pointer in the account's VOC that leads out
of the account is refused too, unless the account has the `os` switch on. When
it opens the file, the connection's containment applies: only the account the
session stands in, the shared `SDSYS` entries (read only) and the directories
named in `NETDIRS`. This is read from the source and has not been run on an
install. The SDNet **client** (`set.server`, `set.netlogin`, a VOC pointer of
type `N`) does not read `NETFILES`; see the SDNet page of the User set.

There is no licence parameter and no licence verb. SD Core Solo for Windows is
GPL v3 and is not licensed per site or per user; `config gpl` displays the
licence text.
