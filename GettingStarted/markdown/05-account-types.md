Title: The account and its passwords
Subtitle: sduser, the account password, the kept copy, and the administrator and global passwords.

## One account: `sduser`

**SD Core Solo for Windows has one SD account, and it is always called
`sduser`** — on every computer, whatever your Windows user is called. The
installer makes it, in `%USERPROFILE%\SDCoreSolo\user_accounts\sduser`. `WHO`,
`@LOGNAME` and the audit trail all say `sduser`, and so does the user name an
API client signs in with.

**There is no way to make another account**, and none is needed: every
session — at the keyboard, over ssh, through the API, or a command from a
script — lands in `sduser`. SD's own system account, SDSYS, exists but is never
entered.

## Three passwords

| | Set | Asked | Unlocks |
|---|---|---|---|
| **Account password** | at installation, or at the first `sd` on a computer installed from a control file | by every session | the account |
| **Administrator password** | at installation | by `ADMIN` | the administrator commands, for the rest of the session |
| **Global password** | at installation, managed mode only | by `ADMIN`, and by any session in place of the account password | the account **and** the administrator commands. It is the SD Core for Linux server's |

**Every one needs at least 8 characters, with a lower-case letter, an
upper-case letter, a digit and a symbol** — letters, digits and punctuation
only. **The global password must differ from both of the others.** One
account name carries both the account password and the global password, and
the account password is tried first — so if they were the same, the server
would land in an ordinary session.

## The account password

**Every session asks for it**, and is refused without it:

| | |
|---|---|
| `sd` at a terminal | `Password:`, three tries, then the session ends |
| `sd` with its input piped | the first line of the input, one try |
| ssh | the same as a terminal, after ssh has checked your Windows password |
| the API | the client library's password, checked by SCRAM — see [API access](09-api-access.html) |
| `sd <command>` | the kept copy, below — no typing |

**A wrong one is answered `Wrong password`.** On a managed computer the global
password is accepted in its place, and that session also has the
administrator commands unlocked.

**Being signed in to Windows is not enough.** The password is the gate; anyone
who has it can use the account, and a copied `SDCoreSolo` folder works for
whoever knows it.

### The kept copy

**Windows keeps an encrypted copy of the account password for you**, protected
so that only your Windows user can open it. A command on the `sd` command line
signs in with it, which is what lets scripts and scheduled jobs use SD — see
[Scheduled jobs](04-scheduled-jobs.html). The installer writes it, and
`SET.PASSWORD` updates it.

**It proves the account password only.** It never unlocks the administrator
commands.

### Changing it: `SET.PASSWORD`

```
:admin
:set.password
New password:
Confirm the new password:
Password changed
```

**It needs `ADMIN` first**, like every administrator command. The new password
must meet the rules above and differ from the global password; otherwise it
says why and leaves the password as it was.

**`SET.PASSWORD` also updates the kept copy.** If it cannot, it says *The new
password could not be kept for commands given on the sd command line* — the
password is changed, but commands on the `sd` command line will fail until it
is set again.

### The first password on a managed computer

**A computer installed from a control file has no account password yet.** The
first `sd` typed at that computer's keyboard asks you to choose one:

```
This account has no password yet. Choose one now - SD Core Solo for Windows asks for it every time it is used.
```

**Until then, only the global password is accepted** — over ssh and the API
the answer is:

```
This account has no password yet. Set it at this computer's keyboard first; until then only the global password is accepted.
```

So the SD Core for Linux server can reach a computer it has just set up, and
nobody else can.

## The administrator password

**It unlocks the administrator commands for one session**: type `ADMIN`, then
the password. See [Administrator commands](06-administrator-commands.html).

**No command changes it after installation**, and none changes the global
password either. Both are set by the installer only.

## The global password

**Managed mode only.** The SD Core for Linux server signs in as `sduser` with
it, and every such session has the administrator commands unlocked. A few
commands need it and refuse the administrator password — the ones that are the
server's rather than the user's. See [Managed mode](15-managed-mode.html).

**The mode is fixed at installation**, and with it whether a global password
exists: nothing sets or clears it afterwards.
