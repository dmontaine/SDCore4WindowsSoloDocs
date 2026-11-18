Title: Configuration
Subtitle: The sd.conf file, the two ways to read a setting, and every parameter SD accepts.

SD reads one configuration file at start-up. It sizes the shared memory
segment, sets the limits every session inherits, and names the directories SD
writes to.

SD folds case, so a command may be typed in either case. Commands are shown
here in lower case.

> The sample output below is illustrative; the values on your system will
> differ where your configuration does.

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
SCREENTHEME=paper
```

**`APIPORT` is active only if the API was chosen at installation.** A computer
installed without it has the same file with the line commented out
(`# APIPORT=4249`), and so no API listener. A managed computer needs it, and
chooses it like any other. `solo-api-listener.ps1` switches it; see
[API access](09-api-access.html).

Lines beginning `#` are comments. The shipped file is heavily commented and
those comments record why each value was chosen. Read them before changing
anything. **An upgrade keeps your `sd.conf`** — see
[Upgrading and uninstalling](01a-upgrading-and-uninstalling.html).

### A name SD does not recognize stops it starting

An unrecognized parameter is not ignored. The parser abandons the file with
`Unrecognized configuration parameter`, and SD does not start.

**The parameters SD no longer has are removed from a kept `sd.conf` for you.**
In 2.0-0 thirteen names went, because they did nothing or imitated another
product: `APILOGIN`, `CODEPAGE`, `CREATUSR`, `EXCLREM`, `FILERULE`, `FIXUSERS`,
`GDI`, `INVCASE`, `JNLDIR`, `JNLMODE`, `PORTMAP`, `RINGWAIT` and `TXCHAR`. A line
naming one would now stop SD, so an upgrade, and a new install that reloads a kept
`sd.conf`, take each such line out first and log which. A configuration file you
copy in by hand from an older installation needs the same done to it.

Range failures behave the same way. `GRPSIZE`, `INTPREC`, `LPTRHIGH`,
`LPTRWIDE`, `MAXCALL`, `RECCACHE`, `SORTMRG` and `MAXIDLEN` are bounds-checked
once the file has been read, and a value outside its range stops start-up with
a message naming the parameter.

## Reading the settings

The `config` verb reports what is in force — after `admin`, apart from `config
gpl` and `config contrib`:

```
:config
Virtual Machine Version Number WS2.0-0
APIPORT   4249
CLEANUP   300 seconds
CMDSTACK  99
DEADLOCK  0
DHCACHE   10
...
YEARBASE  1930
```

**`config` pages its output** — press Return for more, or `Q`.

SD accepts **59** parameters. The verb prints all of them except `APILOCAL`,
`BACKUPDIR`, `NETDIRS` and `SDSYS`, and `DEBUG`, `FDS` and `PHANTOMS` only when
they are set.

`NETDIRS` is the one to know about. It decides what an API session, and a
session held to its account by the `os` switch, may reach outside its own
account (see [The parameters](16c-the-parameters.html#netdirs-directories-outside-the-account)).
The verb will not tell you what it is set to, and `config('NETDIRS')` does not
read it. Read it from the file.

The `config()` function reads one parameter from a program, and needs no
`admin`:

```
group.size = config('GRPSIZE')
```

**56 parameters are readable this way.** The three that are not are `BACKUPDIR`,
`NETDIRS` and `SDSYS`. The name is truncated to sixteen characters before it is matched,
which is longer than any parameter name, so that only shows up if you pass
something which is not a parameter.

Two further forms exist. `config lptr` reports the settings of the default
printer, and `config gpl` and `config contrib` display the license and the list
of contributors — those two need no `admin`.

## Changing a parameter

Editing `sd.conf` and restarting SD is the durable route, and for most
parameters it is the only one. Fourteen tuning parameters can also be changed
from SD itself, with `set.config` or the `E` key of `tool.config` (see
[Changing a Parameter from SD](16b-changing-a-parameter-from-sd.html)).

**Some parameters can also be changed for the current session** (after `admin`):

```
config sortmem 8192
```

That writes to the session's own copy of the settings, taken from shared memory
when the session started. It does not reach `sd.conf`, it does not affect any
other session, and it is gone when the session ends.

The parameters that can be changed this way are `DHCACHE`, `DUMPDIR`, `FLTDIFF`,
`FSYNC`, `GRPSIZE`, `INTPREC`, `LPTRHIGH`, `LPTRWIDE`, `MAXCALL`, `MAXRLOCK`,
`MUSTLOCK`, `NETREPLY`, `NETWAIT`, `OBJECTS`, `OBJMEM`, `PWDELAY`, `RECCACHE`,
`SAFEDIR`, `SDCLIENT`, `SH`, `SH1`, `SORTMEM`, `SORTMRG`, `SORTWORK`, `SPOOLER`,
`TEMPDIR`, `TERMINFO` and `YEARBASE`.

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
made by the verb [`set.config`](16b-changing-a-parameter-from-sd.html), and only after you have typed a value, the page has checked
it against the range SD enforces and a box has shown the exact command and been answered Run. The
other parameters are not changed from SD; editing the file and restarting SD stays the way to change
them.

The page has three views. **P, the parameters**: one row for every parameter SD knows, from the table
`config.table` in the `sdsys` folder, which names every parameter SD reads. The columns are the value
this SD is running with (what `config()` reports), the value `sd.conf` says now (the file is read
again, a parameter written twice counted the way SD counts it, the flag parameters added up), the
default (what SD uses when the file does not say), when a change takes effect and whether anything
acts on it:

| Takes effect | Meaning |
|---|---|
| `restart` | it takes effect when SD is next started (`sd-solo -stop`, then `sd-solo -start`); every parameter the page can change is this, because a Solo session never reads `sd.conf` itself |
| `now` | a program reads `sd.conf` each time it uses the parameter (`BACKUPDIR`), so a change needs no restart and no new session |

| Status | Meaning |
|---|---|
| `live` | something acts on it |
| `file` | read from `sd.conf` by a program each time it is used, not by the kernel (`BACKUPDIR`) |
| `inert` | accepted, and nothing on Windows acts on it (none at present) |
| `unknown` | a name in the file that SD does not know |

A row marked `*` is one where the file and the running SD disagree. **A name in the file
that SD does not know is listed at the end as `unknown`, and the top of the page names it**: SD does
not start with it (the section *A name SD does not recognize stops it starting* above). `D` shows only
the rows where the file and SD disagree, and `S` only the parameters the file sets. The last column,
`Change`, says `e` for a parameter the page can change (press `E`) and `by hand` for every other.

`SDSYS`, `BACKUPDIR` and `NETDIRS` cannot be read with the `config()` function; the page says `(not readable)`
for what SD is running with and shows the file's value. `DEBUG` is not compared: the function reports
it in another form than the file writes it.

**B, the backup directory**: the directory saved with `set.backup.directory` and whether it is there;
**R, the report**: the operating system part of `settings.report`, made when the view is opened and
on F5. It holds no key and no password hash. `F5` reads the file, the backup directory and the report
again; `E` changes the parameter under the cursor, or says why it cannot be changed here; `F1`
explains the page; `Ctrl-Q` leaves. Opening the page writes the audit line
`tool start app=tool.config`; a change writes the line of `set.config` (below), and nothing else the
page does is written.

## Continued in

[Changing a Parameter from SD](16b-changing-a-parameter-from-sd.html) —
changing a parameter from SD with set.config and the E key, then every
parameter
