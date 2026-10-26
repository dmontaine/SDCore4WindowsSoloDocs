Title: Operating System Access
Subtitle: The two switches, sh and os, that decide whether SD can reach the operating system: off as delivered, turned on by the administrator.

`sh` runs a Windows command from the SD prompt. `!` is the same verb under a
shorter name. `OS.EXECUTE` is the way from a program, and a program can also
reach the operating system by naming a file or directory outside the account.
**All of it is off as delivered**, and the administrator turns it on, in two
independent switches, with `set.os.access`.

## The two switches

| Switch | What it lets the account do |
|---|---|
| **`sh`** | the `sh` verb and `!` |
| **`os`** | `OS.EXECUTE`; `OPENSEQ`, `OPENPATH`, `DIR()`, `DELETESEQ`, `DIGEST` of a file and the file-system keys of `OSPATH()` on a path **outside the account's own directory**; a print unit's pathname (`SETPTR ... AS PATHNAME`, `SETPU`, a CSV report's `TO` pathname) outside it; the Python helper; the screen editors `edit` and `micro`; and a VOC pointer to a file outside the account |

**Both are off as delivered, on a new install and on an upgrade unless you said
yes when the installer asked.** The account's own directory is never asked
about: inside it everything works whatever the switches say. **The
administrator needs no switch**: after `admin` the session passes both for as
long as it lasts. One difference remains for the shell: an administrator who has
not been given the `sh` switch may use `sh` and `!`, but only for a plain command
(see *A filter on the command*, below).

**What they run as, when they are on:** your Windows user, on an ordinary
unelevated token — even when your Windows account is an administrator, because
SD drops those rights (see [Running SD](03-running-sd.html)). A command that
needs elevation fails as it would in any unelevated window. Turning a switch on
is therefore not a small thing: whoever signs in to SD, at the keyboard, over
ssh or through the API, can do what your Windows user can do.

SD folds case, so a command may be typed in either case. Commands are shown here
in lower case.

## Looking and changing: `set.os.access`

An administrator command (type `admin` first):

```
:set.os.access
Account sduser has SH no, OS.EXECUTE no
:set.os.access sh on
Account sduser has SH yes, OS.EXECUTE no
:set.os.access os on
Account sduser has SH yes, OS.EXECUTE yes
:set.os.access sh off
Account sduser has SH no, OS.EXECUTE yes
```

With no argument it shows the state and changes nothing. **One switch, one
action, per command**: a command that names both, one word, or anything but `sh`
or `os` followed by `on` or `off` prints `Syntax: set.os.access ...` and changes
nothing. Saying it a second time says `sduser already had that access; nothing
changed`. The answer always gives **both** switches, because `sh off` says
nothing about `os`. The change is written to the audit trail
(`set.os.access sh on account=sduser`), and it is kept in fields 7 and 8 of the
account's record in the account register (`yes` is on; anything else is off, and
a record with fewer fields is off).

**A change is seen at the next sign-on, never by a session that is already
running**: sign out and in again, or open a new ssh or API session. The API
server reads the switches when it enters the account, as the sign-on does.

## All the gates on one page: `tool.gates`

```
tool.gates
```

**A full-screen page, in any session** (140 columns by 43 lines at least). It
lists the three gates this account has: **Shell** (`sh`), **OS.EXECUTE** (`os`)
and **Remote files** (`net`, [API access](09-api-access.html)). For each it shows
whether the gate is ON or off **for this session**, what the register holds
(shown to the administrator only), how it is delivered (off, every one), whether
it reaches beyond SD (every one does) and when a change reaches you. A box under
the list explains the row under the cursor in plain sentences: what ON lets the
account do, what it can still do with it off, and the command that changes it.

| Key | What it does |
|---|---|
| Up, Down, Page Up, Page Down, Home, End, a click | move to a gate |
| Enter, the space bar (**after `admin`**) | turn the gate on the cursor's row ON, or off, through its own verb |
| F5 | ask the system again |
| F1, ?, a click on the [?] | explain the page |
| Ctrl-Q | leave |

**What the columns say.** *This session* is what the system holds for the
session you are typing in: `sh`, `os` and `net` were loaded when you signed on,
so a change made later shows there **after the next sign-on**, which is why the
*Register* column is there (what the next sign-on will load). The page **asks and
does not try**: it never tests an operation to find out whether it works.

**What Enter does (administrator only).** It builds one command (`set.os.access
sh on`, `set.os.access os off`, `set.net.access on` and so on), shows it in a box
with what it will do, and runs it only when you answer Run; Cancel is what Enter
chooses. **Turning a gate ON asks you to type its name** (`sh`, `os` or `net`),
because every one of them reaches beyond SD, and the box says that a gate off as
delivered is being turned on and that leaving it off is the tighter choice. The verb
asks for the administrator itself and writes its own audit line
(`set.os.access sh on account=sduser`); the page writes only the line that says it
was started (`tool start app=tool.gates`), and only for the administrator. After the
command it reads again and shows the verb's own last line in the status bar.
**There is no key that turns everything on, and nothing is on as delivered.** A
session that has not typed `admin` sees the page and cannot change anything: Enter
says so and opens no box.

*Carried from the Linux product's page and not yet run on Windows.*

## What a refusal looks like

With both switches off, `sh` and `!` answer:

```
sduser is not permitted to use the operating system shell
```

A program's `OS.EXECUTE` is a run-time error that says `sduser is not permitted
to use OS.EXECUTE`. A path outside the account given to `OPENSEQ`, `OPENPATH`,
`DELETESEQ`, `DIR()` or the other routes in the table is the `ELSE` branch with
`STATUS()` 3035, the number of "permission"; `DIR()` and `DIGEST` give a null
string, and the `OSPATH()` keys answer false (or a null string, for the keys that
return one). A print file or a `SETPTR ... AS PATHNAME`
outside the account says:

```
sduser has no OS access, so a file or directory outside its own account cannot be reached by pathname (the administrator can give it: set.os.access os on)
```

`edit` and `micro` say that they need OS.EXECUTE permission and name
`set.os.access os on`. `ed`, SD's own line editor, needs none of it.

## The two verbs

```
sh command
! command
```

With `sh` on, everything after the verb is handed to the shell as typed:

```
:sh echo hello-from-the-shell
hello-from-the-shell
:! echo via-the-bang-form
via-the-bang-form
```

**Never type `sh` with nothing after it in a script.** The configured shell
for the bare form is interactive — `config` reports it as

```
SH        C:/Windows/System32/WindowsPowerShell/v1.0/powershell.exe -NoProfile -NoLogo
SH1       C:/Windows/System32/WindowsPowerShell/v1.0/powershell.exe -NoProfile -NonInteractive -Command
```

so a bare `sh` in a piped session, a phantom or a scheduled job hands control to
a shell with nobody at the keyboard and **waits for ever**. The `SH1` form,
which is what `sh command` uses, is `-NonInteractive`.

## A filter on the command

**An account that has the `sh` switch gets a real shell**: pipes, redirection and
chaining work (below). **An administrator who has not been given the switch does
not**: `sh` and `!` refuse a command that contains a shell metacharacter, with
`Error 2 executing operating system command`. The filter stops a one-line command
being turned into two by accident, and it is why giving the `sh` switch is the
step that matters even for the administrator.

## The shell you get is Windows PowerShell 5.1

Not `pwsh`, and the difference bites:

```
:sh echo delta && echo epsilon
At line:1 char:12
+ echo delta && echo epsilon
+            ~~
The token '&&' is not a valid statement separator in this version.
```

**`&&` and `||` do not exist there.** Nor do the ternary, null-coalescing or
null-conditional operators. Chain with `;`, and test with `if ($?) { … }`.

**Pipes and redirection work:**

```
:sh echo alpha-beta | findstr alpha
alpha-beta
:sh echo gamma > zzsh.txt
:sh Get-Content zzsh.txt
gamma
```

The second and third lines are separate `sh` invocations, so **the working
directory persists between them** — the file written by one was read by the
next.

## SD refuses to nest

```
:sh sd-solo
SD is already running in this session - type EXIT to return to it.
```

The child shell is marked, and SD checks the mark:

```
:sh Get-ChildItem Env:SD_SESSION
Name                           Value
----                           -----
SD_SESSION                     1
```

**That is worth knowing when writing a script for `sh` to run** — anything it
invokes inherits `SD_SESSION`, so a script that starts SD as part of its work
will be refused, wherever it is called from.

## `OS.EXECUTE` and the editors

**A program's `OS.EXECUTE` needs the `os` switch**, and so do the screen editors
`edit` and `micro`, which run as Windows programs: they are a way out of SD.
`ed`, the line editor, is not. The form `execute 'sh …'` from a program goes
through the `sh` verb itself, and is asked about the `sh` switch.

## Over ssh and the API

**The switches are loaded the same way for a remote session.** An ssh or API
session signs in to the account, and the sign-on reads the register: with a
switch off, it is off there too. With one on, anyone who has the account
password — and, over ssh, your Windows sign-in — can run Windows commands as
your Windows user. That is one reason every session asks for the account
password; see [Security](12-security.html).

## At install and at upgrade

**The installer asks**, on a new install and on an upgrade, on one page, **Shell
and OS access**, with two boxes, both unticked: *Shell (SH and !)* and *OS access
(OS.EXECUTE, files outside the account, Python, editors)*. **Before this release
the shell and OS access were always on**, so an upgrade asks the same, and an
existing Solo would otherwise lose them without a word: tick each you want to
keep. The answer can also be given in the control file, `os-access=none`, `sh`,
`os` or `both`, on a new installation (an upgrade always asks); with a silent
install, or a page left unticked, nothing is turned on here, which on a new
account is off and on an upgrade leaves the switches as they were. The installer
writes the answer with `set.os.access`, and says in `install-summary.log` what
it was.

## On a managed computer

**The SD Core for Linux server can put `sh` on the list of denied commands**
(see [Managed computers](15-managed-mode.html)); it and `!`, which runs the same
command, then need `admin` first. That list is separate from the switches: a
denied `sh` needs `admin` even when the switch is on, and `set.os.access` is
itself an administrator command.

## The context help: `load.help`

```
load.help
```

The `[?]` button on the administrator's pages shows help kept in the file `context.help`
of the SD system directory, one record to a setting. The installer fills it from the text
under `helptext` there; `load.help` does the same by hand, after that text changes, and
is an administrator command (`admin` first). **It replaces the shipped records and keeps
any you added** under another id (the count says how many). Reading the help is not
gated: the text is the product's own documentation of its screens.

## See also

[Administrator commands](06-administrator-commands.html) ·
[Sessions and locks](06a-sessions-and-locks.html) ·
[Security and the operating system](12a-security-and-the-operating-system.html).
