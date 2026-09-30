Title: SD Core - Introduction and Getting Started
Subtitle: What a multivalue database is, what SD is, the four components, and your first session.

This page orients you to SD Core Solo for Windows: what a multivalue database
is, where SD came from, what the pieces are, and how to take your first steps.
It is the only page in this set that assumes nothing.

## What is a multivalue database?

A multivalue database stores data in records made of **fields**, where
each field can hold **more than one value** — and each value can hold
**more than one subvalue**. A single field in a customer record can
therefore carry every phone number the customer has, without a separate
table or a join.

The model was designed by Dick Pick in the 1970s as the Pick Operating
System. It has been through PI/open, UniVerse, Unidata, D3, jBASE,
QM, and ScarletDME — and SD is one of its direct descendants.

The three delimiters that make it work are **field marks**, **value
marks** and **subvalue marks** — control characters that separate the
levels inside a single string. A dynamic array in SDBasic is a string
that carries these marks, and `extract`, `insert`, `delete` and
`replace` work on them directly.

## What SD is

SD Core Solo for Windows is a version of SD, with elements found in the main
SD version and in ScarletDME. ScarletDME was a fork of the original GPL
release of OpenQM 2.6.6. **Solo is the personal edition of SD Core for
Windows**: one Windows user, one computer, one account, and none of the
multi-user machinery. The GettingStarted set has a page, *Differences from
multiuser SD Core for Windows W1.1-1*, that says exactly what differs.

**That lineage matters when you go looking for documentation.** Not all
the features of the *commercial* OpenQM 2.6.6 were in the GPL release,
and no documentation specific to the GPL version was ever released. The
OpenQM 2.6.6 documents can be used as a reference, but SD Core has
additions, changes and deletions — of features, of structure, of
security and of commands. This documentation set covers those changes.

If you have used OpenQM, or SD on Linux, much of SD Core will still be
familiar: the same data model, the same query processor, the same
BASIC.

**SD Core Solo for Windows is Windows only.** There are no `#ifdef` branches
keeping Linux alive in this source — Linux SD is a separate project and
this is not a build of it.

SD Core is free software under the GNU General Public Licence v3. `config gpl`
displays the licence and `config contrib` the list of contributors. The
installer carries compiled binaries; the source is a separate download.

## The four components

| | |
|---|---|
| **The command processor (TCL)** | reads what you type at the `:` prompt and dispatches it to a verb, a program, a paragraph or a query |
| **The query processor** | runs `list`, `select`, `count`, `sort` and the rest — the reporting language |
| **SDBasic** | the programming language: a compiled BASIC with dynamic arrays, file I/O, and the multivalue string functions |
| **The SDClient API** | a C client library (`sdclilib.dll`) that lets an external application connect to SD, read and write records, execute commands and call subroutines |

## Signing in

```
sd
```

**You land in the one SD account, `sduser`, after the account password** — the
one you chose when installing. Being signed in to Windows is not enough: SD
asks. If the computer was installed from a control file there is no password
yet, and `sd` asks you to choose one. The GettingStarted set's *Your first
thirty minutes* walks through it.

SD is already running. A scheduled task starts it at every Windows start-up,
so you do not type `sd -start`. Open a new window after installing: one that
was open before the install does not have SD on its PATH yet.

## Your first file and record

```
create.file customers
ed customers 1001
```

`ed` is the line editor, and it needs nothing installed. In `ed`: `i`
to insert, type your lines, a full stop on its own line to stop
inserting, then `fi` to file and exit.

You can also use `edit` (Microsoft Edit, a full-screen editor) or `micro` (a
full-screen editor with syntax highlighting). Both come with SD and both open
in your session with nothing to unlock — see *Programmer commands* in the
GettingStarted set.

```
list customers
count customers
```

Commands are lower case now. Typing `LIST` still works — SD tries what you
typed, then lower case, then upper, and finally with any hyphens changed to
dots, so `clear-select` reaches `clear.select` too.

## Writing a program

A program lives in a `bp` file — a directory file, which is an ordinary
Windows folder with one file per program. You can write it in `ed`,
in `edit`, in `micro`, or in any text editor you like (Notepad, VS Code,
etc.) — the folder is on disk at:

```
%USERPROFILE%\SDCoreSolo\user_accounts\sduser\bp
```

Compile and catalogue it from inside SD:

```
basic bp myprog
catalog bp myprog
```

Then run it by name:

```
myprog
```

## Administrator commands

**There is no separate administrator account.** A few commands — `config`,
`listu`, editing the VOC directly — refuse with *Command requires
administrator privileges* until you unlock them for the session:

```
admin
```

Type the administrator password chosen at installation, or, on a managed
computer, the global password. It lasts until you leave SD or type
`admin off`. See *Administrator commands* in the GettingStarted set.

## What is not in SD Core

The following were in OpenQM, in ScarletDME, or in SD on Linux, and
are not in SD Core Solo for Windows:

| Gone | Why |
|---|---|
| QMNet (remote files) | Removed; the API is the supported way to reach another SD server |
| Embedded Python (a Python interpreter loaded into `sd.exe` itself) | Gone permanently - `sd.exe`'s MSYS2 runtime cannot safely share a process with Python. Calling Python **from** a BASIC program is not gone: it runs as a separate helper process instead - see *SD BASIC - Python Integration* |
| `sdlnxd` daemon | Linux-only; a scheduled task starts SD on Windows |
| `ENCRYPT.FIELD` verb | Removed; `sdencrypt()` and `sddecrypt()` in SDBasic are the supported route |
| `sed`, `update.record`, `modify` editors | Gone; use `edit`, `micro` or `ed` |
| PROC language | Removed; use paragraphs instead |
| `SET.LANGUAGE`, `LOAD.LANGUAGE` | Removed; SD Core is English only |
| Accounts, groups and grants | Not in Solo: there is one account, `sduser` |

## Where the listings came from

**The listings in this set were produced by running commands on the multiuser
SD Core for Windows W1.0-0.** Solo shares its runtime — the compiler, the query
processor, the file system — so they show what Solo does too, but they were not
repeated on Solo. Where a page describes something that differs on Solo, it
says so; the account name in a listing may read `DON` or `SDSYS` where Solo
would say `sduser`.

## Document conventions

| | |
|---|---|
| **bold** | a word typed as it stands |
| *italics* | something you supply |
| braces `{ }` | an optional part |
| `code` | a command, a function name, or something you type |
