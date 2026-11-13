Title: Operating System Access
Subtitle: The two switches, sh and os, that decide whether SD can reach the operating system: off as delivered, turned on by the administrator.

`sh` runs a Windows command from the SD prompt. `OS.EXECUTE` is the way from a program, and a program can also
reach the operating system by naming a file or directory outside the account.
**All of it is off as delivered**, and the administrator turns it on, in two
independent switches, with `set.os.access`.

## The two switches

| Switch | What it lets the account do |
|---|---|
| **`sh`** | the `sh` verb |
| **`os`** | `OS.EXECUTE`; `OPENSEQ`, `OPENPATH`, `DIR()`, `DELETESEQ`, `DIGEST` of a file and the file-system keys of `OSPATH()` on a path **outside the account's own directory**; a print unit's pathname (`SETPTR ... AS PATHNAME`, `SETPU`, a CSV report's `TO` pathname) outside it; the Python helper; the screen editors `edit` and `micro`; and a VOC pointer to a file outside the account |

**Both are off as delivered, on a new install and on an upgrade unless you said
yes when the installer asked.** The account's own directory is never asked
about: inside it everything works whatever the switches say. **The
administrator needs no switch**: after `admin` the session passes both for as
long as it lasts.

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

With both switches off, `sh` answers:

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

## The verb

```
sh command
```

With `sh` on, everything after the verb is handed to the shell as typed:

```
:sh echo hello-from-the-shell
hello-from-the-shell
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

**No session gets a pipe, a redirect or a chain at the prompt.** `sh` refuses a
command that holds `;`, `|`, `&`, `$`, a backquote, `<`, `>` or a line break, for
everyone, the administrator included and switch or no switch, with `Error 2
executing operating system command`. The filter stops a one-line command being
turned into two. For more than one step, write a script and run the script:
`sh powershell -File .\nightly.ps1`. **`sh` with nothing after it** starts the
interactive shell, where you may type what you like.

## The shell you get is Windows PowerShell 5.1

Not `pwsh`. Inside a script, **`&&` and `||` do not exist**, nor do the ternary,
null-coalescing or null-conditional operators. Chain with `;`, and test with
`if ($?) { … }`.

Each `sh` is a new shell, but **the working directory persists between them**:

```
:sh Set-Content zzsh.txt gamma
:sh Get-Content zzsh.txt
gamma
```

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
and OS access**, with two boxes, both unticked: *Shell (SH)* and *OS access
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
(see [Managed computers](15-managed-mode.html)); it then needs `admin` first. That list is separate from the switches: a
denied `sh` needs `admin` even when the switch is on, and `set.os.access` is
itself an administrator command.

## The one bypass: showing a document with `pdf`

```
pdf
pdf name
set.viewer
set.viewer add pdf program
set.viewer remove pdf program
docs.install directory
load.help
```

**`pdf name` shows a documentation PDF in a PDF reader, whatever the `sh` and `os`
switches say.** It is the one place where SD starts a program outside itself for an
account that may not run programs, so it is held narrow and it is **off as delivered**:
until you name a reader, `pdf` says that no viewer has been allowed and starts nothing.
The installer does not name one for you.

**What `pdf` starts, and with what.**

* **The program** is the first one on *your* list for `pdf` that passes the test below
  **at the moment it is started**, not only when it was added. The list is the file
  `viewers` in the SD system directory, one line to a program:
  `pdf C:/Program Files/Some Reader/reader.exe`.
* **The document** is a regular file of the `pdf` directory of the SD system directory,
  found by its name in lower case letters, digits, dot, dash and underscore. The caller
  never supplies a path; a link or a directory in that directory is not a document.
* **The command** is built from those two parts alone, started with PowerShell's
  `Start-Process`, so a reader that stays open does not hold the session. The reader is
  given the document as its one argument, in quotes.
* **A session that is not at the computer** (an ssh session, or one that runs in the
  background) is told so, and nothing is started.
* **One audit line**: `pdf open file=name viewer=C:/Program Files/Some Reader/reader.exe`.

**The test a program must pass** (made by `set.viewer add` and again by `pdf`): the full
path of an `.exe` of letters, digits, space and `: / . _ + - ( )` with no `..`; under
`Program Files`, `Program Files (x86)` or the Windows folder; an existing regular file (not a
folder, link or reparse point); owned by SYSTEM, the Administrators or TrustedInstaller;
and no other user or group has an *Allow* entry that writes, appends, deletes, changes
permissions or takes ownership of it. **It does not say a program is safe.** A graphical
reader can open other files its user may read and follow links, so allow only readers you
trust, and never a shell or an editor.

### Naming a reader: `set.viewer`

```
set.viewer                                                    says which programs are allowed
set.viewer add pdf "C:/Program Files/Some Reader/reader.exe"   allows it
set.viewer remove pdf "C:/Program Files/Some Reader/reader.exe" stops allowing it
```

An administrator command (`admin` first; without it: *Command requires administrator
privileges*). The program is always a full path, with `/` or `\`; the double quotes are
optional and take the spaces. Takes effect at once.

### Where the documents come from: `docs.install`

The PDFs are generated from the documentation and are not part of the source tree.
**A release package carries them**, in a folder `pdf` beside the installer, and **the
installer puts them in the `pdf` directory with `docs.install`**. It also copies the
release's changes document from beside it (`sd-core-solo-for-windows-<version>-changes.pdf`,
`.html` and `.md`). An installer run without them says so in its report, and the install
goes on. At any other time, or for documents of your own, the administrator runs:

```
docs.install C:/path/to/the/pdf/files
```

copies every `*.pdf` of that directory into the `pdf` directory of the SD system
directory (which the installer makes). A file is copied only if its name is lower case
letters, digits and `. _ -`, ends in `.pdf` and does not begin with a dot, and the file
begins with `%PDF-`; anything else is skipped and named. A document already there is
replaced. `pdf` alone lists what is installed. `docs.install` is an administrator command
(`admin` first).

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
