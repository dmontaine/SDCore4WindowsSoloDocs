Title: Managed mode
Subtitle: What an SD Core for Linux server can do to a Solo computer it manages, and how.

**A computer installed in managed mode is a local database that an SD Core for
Linux server also manages.** The server is the only thing that manages Solo
computers; there is no Windows management server. **This page will grow** as
management features are added to SD Core for Linux — what is here is what a
managed computer offers the server today.

## What makes a computer managed

**The choice made when installing**, and fixed until a new installation. It
comes with:

| | |
|---|---|
| **a global password** | set at installation, on the installer's page or in the control file. The server signs in with it |
| **the API, on and open** | port 4243, reachable from other computers — the server's way in |
| **ssh, on and open** | the OpenSSH server installed if none was, reachable from other computers |

See [Installing](01-installation.html), including the **control file** that
sets up many computers from one USB stick.

## How the server signs in

**As `sduser`, with the global password.** Over the API that is the whole of
it. Over ssh the server first signs in to the computer's ssh server as its
Windows user — ssh's own sign-in, with that user's Windows password or key —
and then gives SD the global password when `sd` asks. One account name carries
two passwords: SD tries the account password first and the global password
second, which is why the two must differ.

**A session signed in with the global password is a server session.** It has
the administrator commands unlocked from the start, and it is the only kind of
session that may use the commands below — the administrator password does not
open them.

**A server session can change every password on the computer** — the account
password with `SET.PASSWORD`, the administrator password with `SET.PASSWORD
ADMIN`, and the global password with `SET.PASSWORD GLOBAL`, which nothing else
may use. See [The account and its passwords](05-account-types.html).

**On a computer installed from a control file, the server can sign in before
the user has chosen an account password.** Until the user does, at that
computer's keyboard, the global password is the only one accepted.

## The server's programs: `GLOBAL.BP.OUT`

**The global catalogue of a managed computer holds the server's programs.**
The user can run them — `CALL *name` — and cannot add, replace or remove any,
with or without `ADMIN`.

| | |
|---|---|
| `GLOBAL.BP.OUT` | a file of **compiled programs only** — no source is installed. Empty after installation; the server fills it |
| `SYNC.GLOBAL.CATALOG` | makes the global catalogue match `GLOBAL.BP.OUT` |

**To add or replace a program**, a server session copies its compiled object
into `GLOBAL.BP.OUT` — for example from a `BP.OUT` it has written it to — and
runs `SYNC.GLOBAL.CATALOG`:

```
:copy from bp.out to global.bp.out myprog overwriting
:sync.global.catalog
catalogued *myprog
SYNC GLOBAL CATALOG DONE 1 catalogued 0 removed 0 refused
```

**To remove one**, delete it from `GLOBAL.BP.OUT` and run
`SYNC.GLOBAL.CATALOG` again; the `*myprog` entry goes:

```
:delete global.bp.out myprog
:sync.global.catalog
removed *myprog
SYNC GLOBAL CATALOG DONE 0 catalogued 1 removed 0 refused
```

**What `SYNC.GLOBAL.CATALOG` does:** every object in `GLOBAL.BP.OUT` is
catalogued as `*<name>`, in lower case, replacing any older copy; every `*`
entry with no object left in `GLOBAL.BP.OUT` is removed. SD's own system
programs in the catalogue are never touched. An object it cannot load is
refused by name and the rest still go in. The last line always reads
`SYNC GLOBAL CATALOG DONE <n> catalogued <n> removed <n> refused`.

**An upgrade catalogues them again for you.** It replaces the global catalogue
with the new release's, then runs `SYNC.GLOBAL.CATALOG`; `GLOBAL.BP.OUT` itself
is kept.

**Everyone else is refused**:

| | |
|---|---|
| *The global catalogue can only be changed by the SD Core server* | `SYNC.GLOBAL.CATALOG`, or writing `GLOBAL.BP.OUT`, from a session that did not sign in with the global password |
| *The global catalogue holds the SD Core server's programs from GLOBAL.BP.OUT and is changed only by SYNC.GLOBAL.CATALOG* | `CATALOG ... GLOBAL`, a `CATALOG` name beginning `*`, `!`, `_` or `$`, or `DELETE.CATALOG` of a global entry — from any session |

**On a standalone computer** there is no server: `SYNC.GLOBAL.CATALOG` says
so and changes nothing, and the global catalogue holds only SD's own programs.

## Commands the user may not run: `DENY.VERBS`

**The server keeps a list of commands the user of the computer may not run
without the administrator or global password.** A command on the list behaves
like the administrator commands: refused with *Command requires administrator
privileges* until `ADMIN`.

```
deny.verbs                       list them
deny.verbs add listf,create.file add to the list
deny.verbs remove listf          take one off
deny.verbs set listf,copy        replace the list
```

**Every form answers with the list as it now stands:**

```
DENY.VERBS 2: LISTF,COPY
```

| | |
|---|---|
| **Who may use it** | a server session only. Anyone else, `ADMIN` included, is told *The denied verbs can only be listed or changed by the SD Core server* |
| **Never denied** | `ADMIN`, `OFF`, `QUIT` and `LO` — a list naming one says it is dropped |
| **Set at installation** | the control file's `deny-verbs=` line, on a new installation only |
| **Kept by an upgrade** | yes |

**It only adds.** It cannot lift the check an administrator command carries in
its own code; a command already needing `ADMIN` needs it whatever the list
says.

## The limit of all this

**SD enforces these rules; Windows does not.** The whole `SDCoreSolo` folder
belongs to the computer's Windows user, who can change any file in it from
outside SD — `GLOBAL.BP.OUT`, the catalogue, the list of denied commands. What
managed mode protects is what happens inside SD. A computer whose user must
not be able to change these needs that user to lack the Windows rights to the
folder, and Solo does not set that up.

## Coming later

**Further management features are to be added to the SD Core for Linux
server**, and each will have its counterpart here. This page will say what
each one does to a managed computer as it arrives.
