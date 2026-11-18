Title: Administrator commands
Subtitle: admin, the commands that need it, and the maintenance verbs.

**The administrator commands are in your own account, and they need `admin`
first.** There is no separate administrator account to sign in to.

## `admin`

```
:admin
Administrator password:
Administrator commands unlocked for this session
```

**Type the administrator password** — or, on a managed computer, the global
password. **It lasts until you leave SD or type `admin off`**:

```
:admin off
Administrator commands locked
```

| | |
|---|---|
| *Wrong password - administrator commands stay locked* | one try; type `admin` again |
| *Administrator commands are already unlocked* | nothing to do |
| *No administrator password is set on this system* | the installation did not set one; install again |

**A session signed in with the global password is already unlocked.**

**Every attempt is recorded** in the audit trail, unlocked or refused. The
password is never displayed, stored or logged.

## What needs it

**Refused without `admin`, with *Command requires administrator privileges*:**

| | |
|---|---|
| `set.password admin` | change the administrator password — [The account and its passwords](05-account-types.html) |
| `config` | report or set configuration — except `config gpl` and `config contrib`, which need nothing |
| `set.date` | set the session's date |
| `clean.account` | empty the account's scratch files |
| `update.accounts` | refresh the account's VOC |
| `list.voc.removed`, `restore.voc` | the VOC records SD no longer ships, and putting back one the update removed — [`update.accounts`](#updateaccounts) below |
| `set.os.access` | show, or turn on and off, the shell and OS access switches — [Operating system access](06b-operating-system-access.html) |
| `set.net.access` | show, or turn on and off, the NET gate (API request 29, with `NETFILES=2`) — [API access](09-api-access.html) |
| `set.xfer.access` | show, or turn on and off, file transfer over Solo's ssh (`receive` and `send`, with `XFER=1` in `sd.conf`); off as delivered — [Security](12-security.html), and for the person *Moving Files In and Out* in the User documentation |
| `remote.ssh`, `remote.api`, `tool.remote` | show, or change, whether Solo's ssh server and the API are off, local or open — [ssh access](08-ssh-access.html), [API access](09-api-access.html) |
| `set.start.default`, `set.tile`, `set.pins`, `tool.menus` | where a session starts, which tiles the home screen has, whether the account may add its own — [The home screen's settings](#the-home-screens-settings) below |
| `set.viewer`, `tool.viewers`, `docs.install`, `load.help` | the readers `pdf` may start, the documents it shows, the context help — [Showing a document with pdf](06b1-showing-a-document-with-pdf.html) |
| `set.ai`, `tool.ai` | whether the AI may be used (`set.ai gate on`; off as delivered), its provider, model and key, and the page that does it — [Setting up the AI helper](24a-setting-up-the-ai-helper.html) |
| `set.jobs.access` | show, or turn on and off, SD's scheduled jobs (`set.job`, `run.job`, `tool.jobs`); off as delivered, and off refuses every job at its next run — [Scheduled jobs](04-scheduled-jobs.html) |
| `set.server`, `delete.server`, `list.servers` | the servers SDNet may reach — the SDNet page of the User set |
| `tool.gates` | all six gates on one page (sh, os, net, ai, file transfer and scheduled jobs), and the page that changes them (any session sees it; Enter changes a gate after `admin`) — [Operating system access](06b-operating-system-access.html) |
| `tool.sessions` | who is signed on, what is locked, and `logout`, `unlock` and `inhibit.login` on one screen — [Task locks and the sessions screen](06a1-task-locks-and-forcing-a-lock-open.html) |
| `backup.account`, `restore.account`, `set.backup.directory`, `tool.backups`, `settings.report` | [Backing up and restoring the account](06c-backup-and-restore.html) |
| `set.recovery` | show, or turn on and off, the IDE's recovery copy of text you have not saved |
| `append.sd.path` | put SD on your PATH, or take it off |
| `listu`, `logout all` | [Sessions and locks](06a-sessions-and-locks.html) |
| `list.readu` | [Sessions and locks](06a-sessions-and-locks.html) |
| `list.locks`, `lock`, `clear.locks`, `unlock` | [Task locks and the sessions screen](06a1-task-locks-and-forcing-a-lock-open.html) |
| anything on the deny list | a deny list is set by the control file or by the server — [Managed mode](15-managed-mode.html) |

**Refused without `admin`, with *The VOC can only be changed after admin*:**
editing the VOC directly — `ed voc`, a program's `write` or `delete` to the
VOC, `copy` into it — and saving or deleting a sentence with `.s` and `.d`.
What SD writes to the VOC as a side effect of an ordinary command —
`create.file`'s entry, the command stack — is not gated.

**Refused even with `admin`**: changing the global catalog (`catalog ...
global`, `delete.catalog` of a global entry, a write to `global.bp.out`), and
the commands that are the SD Core for Linux server's — see
[Managed mode](15-managed-mode.html).

**`logout` on its own needs nothing** — it ends your own session, like
`quit` — **and nor does `logout` *n***, because every session on a Solo
computer runs as `sduser`. **`sh` needs no `admin`, but it needs its
switch**: it is refused (*is not permitted to use the operating system shell*)
until the administrator has turned `sh` on with `set.os.access`; see [Operating
system access](06b-operating-system-access.html). **Nor does
`set.password`** for your own account password — it asks for the current one
instead; see [The account and its passwords](05-account-types.html).

## The maintenance verbs

### `config`

```
config                     report every setting
config lptr                the same, to the default printer
config param value         set one, for this session only
config gpl                 display the license
config contrib             display the contributors
```

**`config param value` sets a private, session-local value, not the
installation's.** The installation's settings live in `sd.conf` and are read
when SD starts — see [Configuration](16-configuration.html). This form
overrides one for the session you are in: the right tool for trying a value,
the wrong one for changing the installation.

| | |
|---|---|
| *New parameter value required* | `config numlocks` with nothing after it. **The report form is `config` alone** |
| *Not a recognized private configuration parameter name* | the name cannot be set per session |
| *Invalid value for this parameter* | it can, and the value is wrong |

### `set.date`

```
set.date date
```

**Sets the date SD reports in this session, not the computer's clock.** SD
keeps an offset from the real date for the session; `date()` and
`timedate()` return the new date until the session ends.
Other sessions and Windows are not affected. The argument goes through SD's
`D` conversion, so anything `iconv(…, 'D')` accepts will do:

| | |
|---|---|
| *Date required* | nothing after `set.date` |
| *Invalid date format* | the argument is not a date SD can read |

It is for testing date-dependent code without touching the clock.

### `clean.account`

```
:clean.account
Cleaned $COMO
Cleaned $hold
Cleaned $savedlists
```

Empties the account's captured transcripts (`$COMO`), its hold file of reports
(`$hold`) and its saved select lists (`$savedlists`). **Nothing else is
touched** — no data file, no program, no dictionary. A como capture that is
running is left alone and says so.

### `update.accounts`

```
update.accounts {all}
```

**Copies SD's shipped command definitions into your VOC**, adding what is
missing and leaving your own VOC records alone. An upgrade runs it for you,
so you will not normally type it. With one account, `all` and no keyword do
the same thing.

**The copy only adds and changes.** A record you removed stays removed. To keep
your own version of one of SD's records, put `[locked]` in field 1 **after the
type code** — `V[locked]`, `PA[locked]` — and it is left alone. **A verb is
updated anyway**, because a locked verb would go on naming a program this
release replaced; you are told which ones.

**What it takes away: the deletion list.** A record SD has **stopped shipping**
would stay in every account that already had it, so SD ships a second thing
next to the VOC records: a list of the records a release has withdrawn. After
the copy, `update.accounts` removes a listed record from the account's VOC,
**but only when it is the record SD shipped.** Each removal is checked, saved,
reported and audited:

- **It must be the shipped record** (its type and, for a keyword, its number). A
  record of your own with the same name is **kept**, and the update says so
  (12906).
- **A record marked `[locked]` is kept** (12907), except a verb, which is
  removed all the same (12908).
- **A copy is saved first**, in `sdsys\voc_removed.saved` under the account's
  name, the record's id, the date and the time. If it cannot be saved the
  record is not removed (12909).
- **It is reported** (`3 VOC record(s) removed, because SD no longer ships them:
  margin stats supp`, 12905) and one line goes to the audit trail.

The list can be read, and every removal undone (`admin` first):

```
list.voc.removed {sduser}
restore.voc sduser {id}
```

`list.voc.removed` prints the records SD no longer ships, with the release each
went in and what to write instead; given `sduser` it also says what the next
update would do there (removed, kept because it is not the shipped record,
kept because it is locked, or absent) and **changes nothing**. `restore.voc`
puts back the newest saved copy of a record, as it was, and never overwrites a
record of that name that is there now (12923); `all` puts back every record of
the account that has a saved copy. The next `update.accounts` removes a
restored record again, because the list has the last word, so give a record you
want to keep a name SD does not ship. The saved copies stay when you keep your
data at an uninstall.

If the VOC cannot be written, the update says so for the account and leaves it
as it was, instead of stopping with an error.

**Your own changes to the VOC: `newvoc.mods`.** An optional directory file called
`newvoc.mods` in the SD system folder (`sdsys` in the SD Core Solo folder) holds VOC
records of your own — a verb withdrawn, a Q-pointer to a shared file, a paragraph
the account should have. Each record in it is written to the account's VOC
**after** the standard records, so it has the last word, except over a record
marked `[locked]`. The file is a directory file: one record per file, the record
id as the file name, the VOC record as its contents. There is no file until you
make one, and nothing is created for you. It is applied when the VOC is built at
install and at every `update.accounts`, which an upgrade runs, and **an upgrade
leaves the file where it is.** An uninstall that keeps your data keeps it too (see
[Upgrading and uninstalling](01a-upgrading-and-uninstalling.html)). To see it work,
put a record in the file and run `update.accounts`; then `ct voc` the id.

### `append.sd.path`

```
append.sd.path             report whether SD is on your PATH
append.sd.path on          put %USERPROFILE%\SDCoreSolo\usr\bin on it
append.sd.path off         take it off
```

**It changes your own PATH**, not the computer's, and needs no elevation. The
installer puts SD on your PATH every time it runs, so `off` lasts until the
next install or upgrade. A window already open keeps the PATH it started with.

### `set.recovery`

```
set.recovery {on | off}
```

**`set.recovery on`** has the IDE keep a recovery copy of text you have not
saved, readable only by you, in `AppData\Local\.local\state\sd\recovery\sduser`
of your Windows profile (the profile Windows records for your user, never the
`USERPROFILE` or `LOCALAPPDATA` variable). When the record is opened again after
a lost session, the IDE offers the copy back: use it, delete it, or do not open
the record (which keeps it). The record itself is not changed until you save.
The copies are outside the account, so `backup.account` never contains them. It
is **off as delivered**; `set.recovery off` stops new copies (the ones that exist
are kept until answered), and without a word the verb says which it is. It takes
effect at once and writes the audit trail, in lower case.

## The home screen's settings

```
set.start.default {menu | terminal}
set.tile sduser tile {on | off}
set.tile
set.pins {on | off}
tool.menus
```

**The delivered start is the home screen.** A session on a terminal opens it
(tiles for the editor, the IDE, a terminal, the keyboard test, and Sign out, and
the development tiles), and the command prompt is what is left when it returns.
`set.start.default terminal` makes the prompt the start, `set.start.default menu`
the home screen again, and without a word the verb says what it is now. It is
read when a session starts. A piped session, a window under 140 by 43, a phantom
and an API server are never given the screen. **After an upgrade, if you want the
old start, type `admin` and then `set.start.default terminal`.** Starting in the
menu confines nobody: the Terminal tile is there unless you turn it off.

**`set.tile sduser tile off`** takes a tile off the page, and `on` puts it back
(every tile is on as delivered; the ids are `editor`, `ide`, `terminal`, `keys`
and `signout`, and the development tiles `files`, `voc`, `dict`, `query`, `bulk`,
`catalog`, `test` and `recent`; `set.tile` alone says what is off). **Sign out
cannot be turned off.** With **Terminal** off and a start in the home screen, the
account is never given the prompt: when the screen ends it signs the person out.
That is a wall only for the prompt the screen gives; the editor and IDE tiles run
the account's own programs, so turn those off as well, and a shell still needs
`set.os.access sh on`. **`set.pins on`** lets the account add programs of its own
to its home screen (**off as delivered**; `set.pins off` takes it back). A program
of its own runs the command the person could type at the prompt, so it adds no
access.

**`tool.menus`** is the same settings as a page. **It writes nothing itself**:
Enter on a row shows the command it would run with its effect in a box answered
Run or Cancel (Cancel is what Enter chooses). The three verbs write the audit
trail, in lower case. The tile ids and the person's side are in *SD TCL - The Home
Screen* in the User documentation.
