Title: Showing a Document with pdf
Subtitle: The one bypass of the os switch: pdf, naming a reader, where the documents come from, and the context help.

This page continues [Operating System
Access](06b-operating-system-access.html).

## The one bypass: showing a document with `pdf`

```
pdf
pdf name
set.viewer
set.viewer add pdf program
set.viewer remove pdf program
tool.viewers
docs.install directory
load.help
```

**`pdf name` shows a documentation PDF in a PDF reader, whatever the `sh` and `os`
switches say.** It is the one place where SD starts a program outside itself for an
account that may not run programs, so it is held narrow and it is **off as delivered**:
until you name a reader, `pdf` says that no viewer has been allowed and starts nothing.
The installer asks once, unticked, when it finds a known reader and none is allowed yet
(*Allow msedge to show the documentation*); the control file's `viewer=` answers it.

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

### Naming a reader: `set.viewer` and the Bypass programs page

```
set.viewer                                                    says which programs are allowed
set.viewer add pdf msedge                                     allows it (a bare name is looked up in App Paths)
set.viewer add pdf "C:/Program Files/Some Reader/reader.exe"   a full path works too
set.viewer remove pdf msedge                                  stops allowing it
```

An administrator command (`admin` first; without it: *Command requires administrator
privileges*). **A bare name** (letters, digits and `. _ + -`) is the program this
computer's **App Paths** key names for *name*`.exe` — the place Windows records which
program a name starts; only the machine's key is read, which only an administrator can
change. **A full path** is written with `/` or `\`; the double quotes are optional and
take the spaces. Either way the program must pass the test above. Takes effect at once.

**`tool.viewers`**, also an administrator command and a tile (*Bypass programs*) on the
home screen, is the same list as a page: *Allowed for pdf* (each program and whether it
still passes the test) and *Found on this machine* (the known readers App Paths names —
`msedge`, `acrobat`, `acrord32`, `foxitpdfreader`, `sumatrapdf`, `pdfxedit`, `firefox` and
`chrome` — whether each is allowed and whether it could be). Like the menu management
page it **writes nothing itself**: Enter on a reader shows the command (`set.viewer add
pdf msedge`) with its effect in a box answered Run or Cancel (Cancel is what Enter
chooses), allowing one **asks for its name to be typed**, and the line under the page is
the verb's own answer. A program on the list under a path App Paths does not give for its
name is shown and removed at the prompt, because the commands the page may run carry no
slash. F1, `?` or a click on a `[?]` explains each box.

### Where the documents come from: `docs.install`

The PDFs are generated from the documentation and are not part of the source tree.
**A release package carries them**, in a folder `pdf` beside the installer (one PDF
for each chapter of the User and Getting Started documents, the documents the help pages
name), and **the installer puts them in the `pdf` directory by running `docs.install` on
that folder**. It checks each document afterwards and says in its report how many
arrived. An installer run without a package (the installer alone, with no `pdf` folder
beside it) finds no folder and says so, and the install goes on. At any other time, or
for documents of your own, the administrator runs:

```
docs.install C:/path/to/the/pdf/files
```

copies every `*.pdf` of that directory into the `pdf` directory of the SD system
directory (which the installer makes). A file is copied only if its name is lower case
letters, digits and `. _ -`, ends in `.pdf` and does not begin with a dot, and the file
begins with `%PDF-`; anything else is skipped and named. A document already there is
replaced. `pdf` alone lists what is installed. `docs.install` is an administrator command
(`admin` first).

**The changes document comes with the installer.** A release package holds, beside the
installer, the document that lists what changed in the release as
`sd-core-solo-for-windows-<version>-changes.pdf`, `.html` and `.md`. The installer copies
each of those three into the same `pdf` directory, so the PDF opens with `pdf` and the
HTML and Markdown copies are there beside it. A file is copied only if it is a regular
file (not a link) with a plain lower case name of the same shape; a file of that kind
whose name has a capital or another character is left, with a note that says so. An
installer that was not run from a package finds nothing and says so; the install goes on.

### The context help: `load.help`

```
load.help
```

The `[?]` button on the administrator's pages shows help kept in the file `context.help`
of the SD system directory, one record to a setting. The installer fills it from the text
under `helptext` there; `load.help` does the same by hand, after that text changes, and
is an administrator command (`admin` first). **It replaces the shipped records and keeps
any you added** under another id (the count says how many). Reading the help is not
gated: the text is the product's own documentation of its screens.

A record that has more in the documentation names the document on its second line, as the
`pdf` directory holds it: the book first, then the chapter (`user-29a-sd-tcl-the-home-screen`;
the books are `user` and `gettingstarted`). The Documentation button runs `pdf` with that name.

## See also

[Administrator commands](06-administrator-commands.html) ·
[Sessions and locks](06a-sessions-and-locks.html) ·
[Security and the operating system](12a-security-and-the-operating-system.html).
