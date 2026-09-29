Title: Security and the operating system
Subtitle: Reaching the computer from inside SD, how privileged work is done, and the audit trail.

This page continues [Security](12-security.html).

## Reaching the operating system from inside SD

**There are three ways out of SD onto the computer, and all three are open to
your session with no `ADMIN`.**

| | What it is |
|---|---|
| **`sh`** and `!` | a shell at the `:` prompt |
| `OS.EXECUTE` | the operating system from inside a BASIC program |
| **`edit`** and **`micro`** | a text editor, running outside SD |

**They run as your Windows user, on an ordinary token.** Nothing in SD stands
between a session and the computer beyond what Windows itself allows that
user — see [Operating system access](06b-operating-system-access.html).

**That applies to every kind of session**, ssh and the API included. Anyone
who has the account password — and, over ssh, your Windows sign-in — can run
Windows commands as you. That is why the account password matters, and why it
should be one only you know.

**On a managed computer**, the SD Core for Linux server can put `sh` on the
list of denied commands, and `!` goes with it. **A program's `OS.EXECUTE` is
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
2026-09-29 00:15:33 user=sduser uid=105 pid=1043 LOGIN PASSWORD account=SDUSER via=global
2026-09-29 00:15:33 user=sduser uid=105 pid=1043 LOGIN account=SDUSER
2026-09-29 00:15:34 user=sduser uid=105 pid=1043 GLOBAL PASSWORD SET
```

| Recorded | |
|---|---|
| **Sign-ins** | every one, with how the password was proved — `via=account`, `via=global`, or `via=stored` for a command-line `sd <command>` — and every refusal |
| **`ADMIN`** | every unlock and every refusal |
| **Passwords** | a change of the account, administrator or global password, and a refused change |
| **The API** | every login, every refused request, and every failed login with its reason — `API REFUSED user=sduser reason=wrong password`. **The address is not recorded** |
| **Managed mode** | `DENY.VERBS` changes and `SYNC.GLOBAL.CATALOG` runs, and the installer's own internal sessions |

**The refusals are the interesting half.** An `ADMIN REFUSED`, a
`LOGIN REFUSED` or an `API REFUSED` is somebody trying something that did not
work.

**Nothing is ever discarded.** At 1 MB the file is **renamed with the date and
time and a new one started**. Removing the old ones is your decision; SD will
not, and they accumulate.

**This is not the error log and does not behave like it.** `sdsys\errlog`
throws away its oldest half when it fills, and holds diagnostics, not a record
of who did what.

**The audit trail does not protect itself from you.** The file belongs to your
Windows user, who can edit or delete it, and so can any administrator of the
computer. It is a record for reading, not evidence that survives somebody with
your Windows sign-in. Windows keeps its own separate record in the Security
event log; **read the two together.**

## What is still not true

**SD has no file-level access control of its own** on the keyboard and ssh
paths: a session opens files as you, so Windows' permissions on your profile
are the only boundary. The one place a path gate exists is the API, where a
session is confined to the account it stands in — see
[API access](09-api-access.html).
