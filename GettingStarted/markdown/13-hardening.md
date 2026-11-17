Title: Other hardening
Subtitle: The global catalog, the pcode library, the logs, line endings, the terminal, and the rest of the smaller changes.

Everything on this page is a change you may notice while testing, grouped by
what it touches. The passwords and what each one guards are on
[Security](12-security.html); this is the remainder.

## The global catalog

**Nobody changes the global catalog in a session** — not with `admin`, not with
the global password. `catalog ... global`, and `delete.catalog` of a name that is
global (a name starting `!`, `_` or `$`), are refused, and so is a program's
own `write`, `delete` or `clear` of `global.bp.out` and `gcat`:

```
:catalog bp myprog global
The global catalog holds the SD Core server's programs from global.bp.out and is changed only by sync.global.catalog
```

This matters because the global catalog holds the programs SD runs for
everybody, `$login` among them. **Replacing one runs your code in every session,
and deleting one stops everybody signing in.** In the multiuser product `admin`
was enough to overwrite SD's own `$` and `!` programs; here it is not.

**On a managed computer the catalog is the server's**, made from the compiled
programs the server puts in `global.bp.out` and kept in step by
`sync.global.catalog`, which only a session signed in with the global password
can run. See [Managed mode](15-managed-mode.html). **On a computer with no
global password the global catalog is only SD's own** and `sync.global.catalog`
says there is nothing to manage.

**Nothing changes for local and private cataloging**, which is what programmers
use day to day:

```
catalog bp myprog          private catalog, this account
catalog bp myprog local    this account's VOC
```

Both still work, and neither offers to remove a same-named global entry any
more. The only thing you cannot do is catalog a program whose name starts with
`!`, `_` or `$` — those characters mean *global*. Name it without one. You may
**run** a globally cataloged program.

## The pcode library

`sdsys\bin` holds the pcode library — the interpreter itself, which SD loads
into shared memory at start-up and every session then runs.

**SD does not protect it from you.** The folder belongs to your Windows user,
like the rest of `SDCoreSolo`, so anything that runs as you can replace it and
the next start of SD will load what it finds. Windows' own protection of your
profile is what keeps other Windows users away from it — see
[Security](12-security.html).

## Scheduled jobs

A scheduled task can run an SD command as you, with no `admin`: it is
`sd-solo <command>`, which signs in with the kept copy of the account password. It
has its own page: **[Scheduled jobs](04-scheduled-jobs.html)**.

## The logs

There are two SD keeps itself, and they are not interchangeable.

| File | Where | For |
|---|---|---|
| `audit` | `%USERPROFILE%\SDCoreSolo\sdsys` | **who did what** — sign-ins, refusals, `admin`, password changes. See [Security and the operating system](12a-security-and-the-operating-system.html#the-audit-trail) |
| `errlog` | `%USERPROFILE%\SDCoreSolo\sdsys` | diagnostics, and API connection records |

### The error log records who connects to the API port

Every accepted API connection adds a line naming the Windows process and
account at the other end:

```
API connection from 127.0.0.1:59314 - pid 11448, DESKTOP-1\sam
```

**Nothing is refused on the strength of it.** This records who connected; it
does not decide who may. The API's own checks — the account password over
SCRAM, inside TLS 1.3 — are unchanged.

**A connection forwarded over ssh shows `sshd`, not the person at the far
end.** The tunnel ends on this computer, so the process that connects
genuinely is `sshd`. What the line distinguishes is a client running *on* this
computer from one arriving through a tunnel; **it cannot name a remote
person.**

*"peer process not identified"* means the client had already gone by the time
the connection was looked up. It is not an error and the connection proceeds
normally.

### Two things about error-log trimming

**`ERRLOG` now applies to these lines too.** The background daemon used to
append without ever trimming — it only wrote at start-up and on failure, so it
never grew. Now that it writes per connection, it discards the oldest part of
the log on reaching the `ERRLOG` size in `sd.conf`. **If you have set `ERRLOG`
unusually large, consider what an entry per connection adds to it.**

**After the log is trimmed, its first line may have no timestamp.** An entry
is two lines — a timestamped header and the message indented below it — and
trimming restarts the file at a **line**, not at an entry, so the first message
can be left without its header. **This is not damage** and no entry after it is
affected. SD has always trimmed this way; it is only visible now because the log
turns over more often.

## Line endings

Both halves are fixed, and they were fixed separately.

### Reading — files edited in Notepad or saved from Excel

Directory files exist so you can edit their records with an ordinary Windows
editor, and Windows editors end each line with CR+LF. **SD only ever looked for
the LF, so it kept the CR — and put it on the end of the data.**

You would have seen it as an invisible extra character at the end of every
line: comparisons failing for no visible reason, a name that would not match, a
trailing space that was not a space.

**Reading a CSV saved by Excel is the clearest case.** The last column of
every row picked up the stray character, because a comma ended the other
columns and the line ending only ever touched the last one. `READCSV`,
`READSEQ` and reading a directory file record are all corrected.

**A CR on its own is still data** and is left exactly as it is. Only the CR+LF
pair that ends a line is treated as a line ending, so this cannot alter data
that happens to contain a CR.

### Writing

Anything SD writes that an ordinary Windows program can open now ends its lines
with CR+LF: records in a directory file, `WRITESEQ` and `WRITECSV` output,
command output captured with `COMO`, printer output sent to a file, and the
error log.

SD's CSV statements are documented as following RFC 4180, and **that standard
asks for CR+LF.**

**Dynamic files are unaffected** — they are stored in SD's own format and are
not readable by other programs.

**Existing files are left alone**, so a file can contain both endings. SD
reads either, so this is untidy rather than a problem.

## The terminal

**The default terminal type is now `WINDOWS`.** `TERM` on its own should say
`Device : windows`.

**THE ARROW KEYS DID NOTHING in cmd, PowerShell or Windows Terminal** on
earlier builds. A terminal has two spellings for an arrow key: in its ordinary
state it sends `ESC [ D` for Left, and the other spelling `ESC O D` only after
the application asks it to switch — **which SD never does.** The `vt100`
definition SD was defaulting to lists only the second spelling, so SD was
listening for a key no Windows console ever sends.

The shipped `WINDOWS` definition is an exact copy of `LINUX`, which had this
right all along — its name describes an operating system, but what matters is
the byte protocol.

**An account keeps the terminal setting in its VOC** until that VOC is updated;
an upgrade runs `update.accounts` for you, and you can run it yourself. Until
then, `term windows` sets it for the session. **26 definitions ship, compiling
to 37 terminal names** — the extra names are aliases such as `vt200` and
`wyse60` — so `term wyse60` still works.

**A name that is not installed is refused and your current type is kept** —
*"Unrecognized terminal name"* — so a typo costs you nothing. **`term` with no
argument reports the type actually in force**, which is how to check.

Watch for near-misses all the same. There is no `vt320` definition; `vt220`
or `xterm` does the same job. `terminfo.src` ships with SD, so `sdtic` can add
a definition that is not there.

**Backspace works**, at the prompt and when you are asked for a password.

### The page is 140 × 43, not 80 × 24

**SD's default terminal size is 140 columns by 43 lines** (it was 120 by 36
in earlier builds). **A console window narrower than that
makes wide reports look wrapped or truncated**, which reads as a formatting bug
and is not one. SD's own full-screen programs (`sed`, `ced`, `ide`, the home
screen) want at least 140 columns by 43 lines and refuse a smaller window.

`term` reports the size in force, above the `Device` line:

```
:term
Page width: 140
Page depth: 43
Device    : windows
```

**The size is worked out at login**, in this order: the `LINES` and `COLUMNS`
environment variables if they are numeric, otherwise the terminfo entry's
`lines` and `cols`, **otherwise 43 and 140** — then raised to a minimum of
10 × 20 if smaller. So a console or ssh session normally gets its real window
size and 140 × 43 is the fallback when nothing answers, which is the case for a
phantom or a piped script.

> **`term default` restores it, and it prints nothing when it does.** It sets
> the same 140 × 43 the login path falls back to and returns silently, so run a
> bare `term` after it to see the result. `term 120,36` does the same by hand
> for the old size.

## Paths

**A Windows path typed at the command prompt is no longer cut off at the first
backslash.** Typing `C:\Data\Sales` was read as just `C:`, with the rest
treated as a second, separate thing. A command that is given a folder was the
most likely to show it — the work went to the wrong place, or the command failed
for a reason that made no sense from what you had typed.

Forward slashes always worked and still do. **Both are now read the same way.**

## PowerShell execution policy — leave it alone

This is a hardening page, so it is worth saying plainly: **tightening your
PowerShell execution policy does not break SD, and loosening it does not help
SD.** Set it to whatever your own security policy wants.

SD does some of its Windows-side work — `append.sd.path` and the editor verbs
among it — by running a small PowerShell script that the installer put in
`%USERPROFILE%\SDCoreSolo`. Each one is launched with `-ExecutionPolicy Bypass`
**on that single command line** (the `SH1` line of `sd.conf`), which applies to
that one process and that one script. It does not change the computer, and it
does not affect any other script you or anyone else runs.

**The `sh` verb gives you a PowerShell prompt under your computer's own
policy** — SD lifts the restriction only for the scripts it installed itself,
never for a shell you type into.

**The one case that does stop SD is Group Policy.** A policy that sets the
execution policy outranks anything a program can pass on a command line. Run
`Get-ExecutionPolicy -List`: if `MachinePolicy` or `UserPolicy` reads
`Restricted` or `AllSigned`, the commands that run those scripts will fail and
only your Windows administrator can change it. `Undefined`, `RemoteSigned`,
`Unrestricted` or `Bypass` on those two rows are all fine. **[The installed
scripts](17-the-installed-scripts.html) has the detail.**

## Running SD

| | |
|---|---|
| How it starts | a scheduled task, **SD Core Solo** — see [Running SD](03-running-sd.html). There is no Windows service |
| After an unclean shutdown | SD starts anyway, rather than refusing because the last stop was abrupt |
| Nested sessions | SD will not start a second time inside itself |
| `sd-solo <command>` | signs in with the kept copy of the account password, so a script or a scheduled job can use it — see [Running SD](03-running-sd.html) |
