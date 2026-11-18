Title: Managed mode
Subtitle: What an SD Core for Linux server can do to a Solo computer it manages, and how.

**A managed computer is a local database that an SD Core for Linux server also
manages.** The server is the only thing that manages Solo computers (there is no
Windows management server). **This page will grow** as
management features are added to SD Core for Linux — what is here is what a
managed computer offers the server today.

## What makes a computer managed

**A global password, and nothing else.** It is entered when installing — on the
installer's page, or in the control file — and may be left blank; a computer
with no global password is not managed. Whether there is one is fixed until a
new installation.

**The server also has to be able to reach the computer.** Both of these are a
choice in every installation, on the installer's Tasks page or in the control
file, and neither is switched on for you:

| | |
|---|---|
| **the API, on and open** | port 4249, reachable from other computers — the server's way in |
| **ssh, on and open** | Solo's own ssh server on port 4251 (the OpenSSH package is installed if none was), reachable from other computers |

With either off, or open to this computer only, the server cannot use that
route. **Nothing forces either open.** If you give the installer a global
password on a new installation and either is off or local, the finish page and `install-summary.log`
say so (*This computer has a global password, but the API is local and ssh is
off*): an SD Core server reaches this computer only through what is open to the
network, so with neither open the computer is managed in name only. An administrator who sets up many computers puts `api=open` and
`ssh=open` in the control file.

See [Installing](01-installation.html), and the **control file** that sets up
many computers from one USB stick in [Installing many
computers](01c-installing-many-computers.html).

## How the server signs in

**As `sduser`, with the global password.** Over the API that is the whole of
it. Over ssh, on port 4251, the server first gets past ssh's own sign-in — its key, which
it installed through the API — and then gives SD the global password when `sd-solo`
asks. One account name carries
two passwords: SD tries the account password first and the global password
second, which is why the two must differ.

**A session signed in with the global password is a server session.** It has
the administrator commands unlocked from the start, and it is the only kind of
session that may use the commands below — the administrator password does not
open them.

**A server session can change every password on the computer** — the account
password with `set.password`, the administrator password with `set.password
admin`, and the global password with `set.password global`, which nothing else
may use. See [The account and its passwords](05-account-types.html).

**On a computer installed from a control file, the server can sign in before
the user has chosen an account password.** Until the user does, at that
computer's keyboard, the global password is the only one accepted.

## The server's ssh key

**The server installs its own ssh key.** The server knows only this computer's
address and the name `sduser` — not the Windows user name that ssh needs. So after
it signs in over the API, it asks this computer to install the server's public
key, and the computer answers with the Windows user name as ssh matches it, its
host name, the key's fingerprint (`SHA256:...`), `ADDED` or `PRESENT`, the
fingerprint of Solo's own ssh server's host key and the port to connect to
(4251). From then on the server can sign in over ssh, on that port, with that
key instead of the Windows password; the key is an extra and does not switch
the password off. The same request lists (`LIST`) and removes (`REMOVE`, answered
`REMOVED` or `ABSENT`) the server's keys. **It is API request 72** (it was 49
before 2.0-0; 49 now asks the server's revision, as in SD Core), so a program of
your own that sends 49 to install a key must send 72.

- **Only a session signed in with the global password may ask.** The account
  password and `admin` are refused with *Only the SD Core server may manage ssh
  keys*, and nothing changes.
- **The key can start `sd-solo` and nothing else** — no shell, no forwarding. What
  lands in the file is one line: `restrict`, the key and the tag
  `sdcoresolo-managed`, in Solo's own key file,
  `%USERPROFILE%\SDCoreSolo\ssh\authorized_keys`: see [ssh access](08-ssh-access.html).
- **At most four server keys are kept.** A refusal says why in one of four ways:
  the key or fingerprint is not valid, four keys are already installed, the
  request is unknown, or it could not be carried out. **Your own ssh keys are
  never added to, listed, counted or removed.**
- **Every use is in the audit trail**, with the key's fingerprint and the address
  it came from, never the key itself.
- **It does nothing on a computer with no global password**: there is no server,
  so nothing can ask.
- **It needs Solo's own ssh server**, which needs the OpenSSH package's
  `sshd.exe` — see [ssh access](08-ssh-access.html). Without it the request is
  refused. A computer installed before 2.0-0 is moved to the new arrangement by
  an upgrade: the key already added is moved into Solo's key file, and the
  server reaches the computer on port 4251 from then on, not 22.

**The server's certificate is pinned.** The first time the server's client
library connects to this computer's address and port, it remembers the
computer's TLS certificate. If a later connection to the same address presents a
different one, the library refuses it before sending anything, and says which
line to remove. Reinstalling this computer gives it a new certificate, so the
server's line for it has to be removed (the message names the file) before the
server can connect again. The very first connection is trusted. See
[API access](09-api-access.html).

## The server's programs: `global.bp.out`

**The global catalog of a managed computer holds the server's programs.**
The user can run them — `CALL name` — and cannot add, replace or remove any,
with or without `admin`. A program of the same name in the account's own
catalog is found first.

| | |
|---|---|
| `global.bp.out` | a file of **compiled programs only** — no source is installed. Empty after installation; the server fills it. It is `%USERPROFILE%\SDCoreSolo\sdsys\global.bp.out` |
| `sync.global.catalog` | makes the global catalog match `global.bp.out` |

**To add a program**, a server session copies its compiled object into
`global.bp.out` — for example from a `BP.OUT` it has written it to — and runs
`sync.global.catalog`:

```
:copy from bp.out to global.bp.out myprog
1 record(s) copied.
:sync.global.catalog
cataloged myprog
sync global catalog done 1 cataloged 0 removed 0 refused
```

A session with no administrator rights can then `CALL myprog(x)`. **A program
cataloged in the account under the same name is found first**, as for any
`CALL`. The server's programs have plain names and no prefix: a left-over
`*name` entry from an older release is removed by the first `sync.global.catalog`,
which catalogs the programs again under their plain names.

**To remove one**, delete it from `global.bp.out` and run
`sync.global.catalog` again; the `myprog` entry goes:

```
:delete global.bp.out myprog
:sync.global.catalog
removed myprog
sync global catalog done 0 cataloged 1 removed 0 refused
```

**What `sync.global.catalog` does:** every object in `global.bp.out` is
cataloged under its plain name, **in lower case** — every name in the catalog is
lower case — replacing any older copy; a program whose name begins with `$`,
`!`, `_`, `*` or `-` is refused, since those characters are SD's own. Every
catalog entry with no `$`, `!` or `_` prefix and no object left in
`global.bp.out` is removed. SD's own system programs in the catalog are never
touched, because every one of them begins with `$`, `!` or `_`, and nothing else
can put an entry without one there. An object it cannot load is refused by name
and the rest still go in. The last line always reads
`sync global catalog done <n> cataloged <n> removed <n> refused`.

**An upgrade catalogs them again for you.** It replaces the global catalog
with the new release's, then runs `sync.global.catalog`; `global.bp.out` itself
is kept.

**Everyone else is refused**:

| | |
|---|---|
| *The global catalog can only be changed by the SD Core server* | `sync.global.catalog`, or writing `global.bp.out`, `gcat` or the deny list, from a session that did not sign in with the global password — `admin` included |
| *The global catalog holds the SD Core server's programs from global.bp.out and is changed only by sync.global.catalog* | `catalog ... global`, a `catalog` name beginning `!`, `_` or `$`, or `delete.catalog` of a global entry — from any session |

**A user's own program cannot write to them either.** A BASIC program that
opens `global.bp.out` and `WRITE`s to it is refused by SD itself, with
`STATUS()` saying so, whatever the caller's rights.

**On a computer with no global password** there is no server:
`sync.global.catalog` says *this computer has no global password, so no SD Core
server manages it and there is nothing to manage*, changes nothing, and the
global catalog holds only SD's own programs.

## Commands the user may not run: `deny.verbs`

**The server keeps a list of commands the user of the computer may not run
without the administrator or global password.** A command on the list behaves
like the administrator commands: refused with *Command requires administrator
privileges* until `admin`.

```
deny.verbs                       list them
deny.verbs add sh,create.file    add to the list
deny.verbs remove sh             take one off
deny.verbs set sh,copy           replace the list
```

**Every form answers with the list as it now stands:**

```
deny.verbs 2: sh,copy
```

**A command is denied under every name that runs it.** Some names that look
different share one program: `edit` and `micro` both run SD's editor program,
so denying one denies the other, and the answer says what else was taken
(`deny.verbs also denies, as the same command: ...`). **The case a record spells
its program in does not matter:** a verb whose VOC record runs the same program
as a listed verb is denied too, however the program's name is written. For example:

```
:deny.verbs add sh
deny.verbs 1: sh
```

| | |
|---|---|
| **Who may use it** | a server session only. Anyone else, `admin` included, is told *The denied verbs can only be listed or changed by the SD Core server* |
| **Never denied** | `admin`, `off`, `quit` and `lo` — a list naming one says it is dropped |
| **Set at installation** | the control file's `deny-verbs=` line, on a new installation only. **On a computer with no global password no session can change the list afterwards** — only the server can — so it is fixed until a new installation |
| **Kept by an upgrade** | yes — it lives in `%USERPROFILE%\SDCoreSolo\sdsys\solo.policy` |

**It only adds.** It cannot lift the check an administrator command carries in
its own code; a command already needing `admin` needs it whatever the list
says.

**It is checked where every command is dispatched** — typed, run from a
paragraph, `EXECUTE`d by a program, or sent over the API — so no route skips it.

## The limit of all this

**SD enforces these rules; Windows does not.** The whole `SDCoreSolo` folder
belongs to the computer's Windows user, who can change any file in it from
outside SD — `global.bp.out`, the catalog, the list of denied commands. What
managed mode protects is what happens inside SD. A computer whose user must
not be able to change these needs that user to lack the Windows rights to the
folder, and Solo does not set that up.

## Coming later

**Further management features are to be added to the SD Core for Linux
server**, and each will have its counterpart here. This page will say what
each one does to a managed computer as it arrives.
