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
| **the API, on and open** | port 4249, reachable from other computers — the server's way in |
| **ssh, on and open** | the OpenSSH server installed if none was, reachable from other computers |

See [Installing](01-installation.html), including the **control file** that
sets up many computers from one USB stick.

## How the server signs in

**As `sduser`, with the global password.** Over the API that is the whole of
it. Over ssh the server first signs in to the computer's ssh server as its
Windows user — ssh's own sign-in, with that user's Windows password or key —
and then gives SD the global password when `sd-solo` asks. One account name carries
two passwords: SD tries the account password first and the global password
second, which is why the two must differ.

## The server's ssh key

**The server cannot know which Windows user this computer's SD belongs to,** and
ssh needs that name. So over the API, a server session can install the server's
ssh public key in your own Windows user's `.ssh\authorized_keys`, and is told
the user name in reply. From then on the server signs in over ssh with its key
instead of a Windows password.

| Request | What it does |
|---|---|
| **ADD** | puts the key in `%USERPROFILE%\.ssh\authorized_keys`. The reply carries your Windows user name as ssh matches it, this computer's name, the key's fingerprint (`SHA256:...`), `ADDED` or `PRESENT`, and the fingerprint of this computer's ssh server key, so the server can check it is talking to the right computer — empty when that is not known |
| **REMOVE** | takes the key with that fingerprint out; the reply is `REMOVED` or `ABSENT` and the number of the server's keys left |
| **LIST** | the fingerprints of the server's keys, one per field |

**Only a server session may ask.** A session signed in with the account
password is refused with *"Only the SD Core server may manage ssh keys"*, and
nothing changes. A standalone computer has no global password, so nothing can
ask.

**What lands in the file is one line:** `restrict`, the key, and the tag
`sdcoresolo-managed`. The key can start SD and nothing else — it gives no shell
and no port forwarding. **At most four** of these lines are kept, and **your
own keys in the file are never added to, listed or removed.** A refusal says
why in one of four ways: the key or fingerprint is not valid, four keys are
already installed, the request is unknown, or it could not be carried out. Every
use is written to the audit trail, and a successful one carries the key's
fingerprint and the address the request came from.

**It needs the installer's ssh block.** A managed install writes it with a key
file line and in the position that makes ssh read your own file even when your
Windows user is an administrator — see [ssh access](08-ssh-access.html). A
computer installed before that was done refuses the request until it is
installed again; an upgrade does not rewrite the block.

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

**A command is denied under every name that runs it.** Denying `SH` denies
`!`, and denying `CATALOG` denies `CATALOGUE`, because each pair runs the same
command. Some names that look different share one: `EDIT`, `NANO` and
`MICRO` all run SD's editor program, so denying one denies all three. The
answer says what else was taken:

```
:deny.verbs add sh
DENY.VERBS also denies, as the same command: !
DENY.VERBS 1: SH
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
