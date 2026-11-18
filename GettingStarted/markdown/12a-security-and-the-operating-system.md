Title: Security and the operating system
Subtitle: Reaching the machine from inside SD, how privileged work is done, and the audit trail.

This page continues [Security](12-security.html).

## Reaching the operating system from inside SD

**There are three ways out of SD onto the computer, and each is behind a switch
that is off as delivered.** The administrator turns a switch on with
`set.os.access` (after `admin`); no other command does.

| | What it is | Switch |
|---|---|---|
| **`sh`** | a shell at the `:` prompt | `sh` |
| `OS.EXECUTE` | the operating system from inside a BASIC program | `os` |
| **`edit`** and **`micro`** | a text editor, running outside SD | `os` |

**`os` also opens** the files and directories outside the account's own
directory (by `OPENSEQ`, `OPENPATH`, `DIR()` and the other routes), the Python
helper and a VOC pointer to a file outside the account. **The administrator
needs no switch**: after `admin` a session passes both. The installer asks about
both on a new installation, and the default is no; an upgrade keeps the switches
the installation has.

**A session with no terminal is refused for the editors regardless** — an API
session or a piped script has nowhere to draw a full screen for an editor.

**When they are on, they run as your Windows user, on an ordinary token.**
Nothing in SD then stands between a session and the computer beyond what Windows
itself allows that user — see [Operating system
access](06b-operating-system-access.html).

**That applies to every kind of session**, ssh and the API included: the
switches are read at sign-on, and the API server reads them when it enters the
account. With a switch on, anyone who has the account password — and, over ssh,
your Windows sign-in — can run Windows commands as you. That is why the account
password matters, and why it should be one only you know.

**On a managed computer**, the SD Core for Linux server can put `sh` on the
list of denied commands. **A program's `OS.EXECUTE` is
not a command the list can hold**, so the list alone does not close the
computer off from a program. See [Managed mode](15-managed-mode.html).

## Privileged work is done through a script, not a command line

When SD has to ask Windows to do something — editing your PATH, for one — it
writes a short script to a file and runs it, rather than putting values on a
command line where any local program could read them through Task Manager or
WMI.

Those scripts go in **`sdsys\pstmp`**, a folder of its own that the installer
creates. **If it is missing, SD refuses that work** — the command fails with an
error — rather than falling back to somewhere less private.

## The audit trail

`%USERPROFILE%\SDCoreSolo\sdsys\audit` records what happened in SD, one line
each, with the date, time and user:

```
2026-09-30 02:56:31 user=sduser uid=1 pid=109059 login password account=sduser via=stored
2026-09-30 02:56:31 user=sduser uid=1 pid=109059 login account=sduser
```

Every word before the first `=` is lower case, the event names included. What
follows an `=` is data and keeps its case: the account name, a `reason=` text,
and what a caller typed in `command=`. A file written by an older version may hold
upper case lines, which are not rewritten, so match without regard to case when you
search a long-lived file.

| Recorded | |
|---|---|
| **Sign-ins** | every one, with how the password was proved — `via=account`, `via=global`, `via=first` (the first password, chosen at the console), `via=stored` for a command-line `sd-solo <command>`, or `via=windows-user` for a scheduled job's `run.job` — and every refusal |
| **`admin`** | every unlock and every refusal |
| **Passwords** | a change of the account, administrator or global password, and a refused change |
| **The API** | every login, every refused request, and every failed login with its reason — `api refused user=sduser reason=wrong password`. **The address is not recorded** |
| **Managed mode** | `deny.verbs` changes and `sync.global.catalog` runs, and the installer's own internal sessions |

**The refusals are the interesting half.** An `admin refused`, a
`login refused` or an `api refused` is somebody trying something that did not
work.

**The audit trail has a second copy.** Every record SD writes to the audit file is
also sent to the Windows Application event log: the source is `sd-audit`, the
level Information and the event id 1, and the event is stamped with the Windows
user of the session that made the record. The audit file is in your Solo folder and
belongs to you, so a program you run could change it; the event log copy gives you
a second place to look, read with the tools you already use for Windows (Event
Viewer, under *Windows Logs*, *Application*, or `Get-WinEvent` in PowerShell). A
line in the audit file with no matching event is one to look at. **The copy is a
convenience, not a protected record**: any program on the computer can write events
under the source name `sd-audit`, and an administrator can clear the log, so a line
there proves nothing by itself. If the event log cannot be reached, SD carries on
and the file is still written. The installer registers the source on a first
install and on an upgrade.

**Nothing is ever discarded.** When SD starts and the file is 1 MB or more, it is
**renamed with the date and time and a new one started**. Removing the old ones
is your decision; SD will not, and they accumulate.

**This is not the error log and does not behave like it.** `sdsys\errlog`
throws away its oldest half when it fills, and holds diagnostics, not a record
of who did what.

**The audit trail does not protect itself from you.** The file belongs to your
Windows user, who can edit or delete it, and so can any administrator of the
computer. It is a record for reading, not evidence that survives somebody with
your Windows sign-in. Windows keeps its own separate record in the Security
event log; **read the two together.**

### Reading it: `tool.audit`

`tool.audit` is a full-screen page for the administrator (`admin` first). It needs a window of at
least 140 columns by 43 lines, like the other toolkit pages, and it asks for the administrator
before it reads anything.

The top box names the file shown, its size, the lines loaded, whether older lines remain, the filters in
force and whether the page is following the file. The list under it is the trail, the oldest loaded line
at the top and the newest at the bottom, one row to a line: time, user, event words, then the pairs. The
box at the foot shows the whole line under the cursor, wrapped, and the same line cut into its parts.

**The page reads from the end of the file, in blocks, and never reads a large trail whole.** It opens
on the newest 200 lines that pass the filter; when the cursor goes above the first line it loads 200
more. It keeps at most 3000 lines; beyond that it drops the newest while you go back and the oldest
while you follow. A last line that has no line end yet is left until it is whole.

| Key | What it does |
|---|---|
| Up, Down, PgUp, PgDn | move; above the first loaded line, older lines are loaded when the file has them |
| Home, End | the first loaded line; the newest line (End reads the file again) |
| F | asks, one after another: from, to, event words, account, user, text. `from` and `to` are the start of a time stamp (`2026-10-25`, or `2026-10-25 14:30`; `to` with only a date includes the whole day). Event, account, user and text are matched without regard to case; `account` is the line's `account=` value, `user` its `user=`. Esc leaves every filter as it was |
| / | the same as F: on every page `/` is the key that finds by typing |
| C | clears the filters |
| D | goes to the first line at or after a time; no filter changes |
| O | shows the next audit file: the current one, then the renamed ones, newest first |
| L | follows the file: new lines are read every three seconds, and the cursor stays on the newest line if it was there |
| X | saves the lines loaded to a record of the account's `$HOLD` file, named `audit.yyyymmdd.hhmmss`, and writes the audit line `tool export app=tool.audit lines=n record=id`. `ED $HOLD id` reads it |
| F5 | reads again |
| F1, ? | explain the page |
| Ctrl-Q | leave |

**The page changes nothing.** It runs no command and writes no record but the one X saves. Opening it
writes `tool start app=tool.audit`, and X writes the one export line; both are on the trail.

## What is still not true

**SD's own access control to operating-system files is the account's `os`
switch, and it is the only one.** While the switch is off, as delivered, SD holds
every session — at the keyboard, over ssh and over the API alike — to the
account's own folder when it opens an operating-system file by pathname (plus
any folder the administrator listed in `NETDIRS`); with the switch on, a
session is held only by Windows' permissions on your user. What a session may do
inside the files SD itself opens is whatever Windows lets your user do — see
[API access](09-api-access.html#a-session-is-held-to-its-own-account).
