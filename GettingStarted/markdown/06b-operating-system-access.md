Title: Operating System Access
Subtitle: The `sh` and `!` verbs, OS.EXECUTE, and what they reach on a Solo computer.

`sh` runs a Windows command from the SD prompt. `!` is the same verb under a
shorter name. **They are the only way out of SD to the operating system from
TCL**; `OS.EXECUTE` is the way from a program.

**On a Solo computer they are yours to use, with no `ADMIN`** — at the keyboard,
over ssh and through the API. SD, its data and the Windows user it runs as are
all one person's, so there is nobody for a gate to keep out. The multiuser SD
Core for Windows decided this per person with an `os.users` list; Solo has no
such list.

**What they run as:** your Windows user, on an ordinary unelevated token —
even when your Windows account is an administrator, because SD drops those
rights (see [Running SD](03-running-sd.html)). A command that needs elevation
fails as it would in any unelevated window.

SD folds case, so a command may be typed in either case. Commands are shown here
in lower case.

> **The listings on this page were produced on the multiuser SD Core for
> Windows**, whose shell code Solo shares; only who may use it differs.

## The two verbs

```
sh command
! command
```

Everything after the verb is handed to the shell as typed:

```
:sh echo hello-from-the-shell
hello-from-the-shell
:! echo via-the-bang-form
via-the-bang-form
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

## The shell you get is Windows PowerShell 5.1

Not `pwsh`, and the difference bites:

```
:sh echo delta && echo epsilon
At line:1 char:12
+ echo delta && echo epsilon
+            ~~
The token '&&' is not a valid statement separator in this version.
```

**`&&` and `||` do not exist there.** Nor do the ternary, null-coalescing or
null-conditional operators. Chain with `;`, and test with `if ($?) { … }`.

**Pipes and redirection work:**

```
:sh echo alpha-beta | findstr alpha
alpha-beta
:sh echo gamma > zzsh.txt
:sh Get-Content zzsh.txt
gamma
```

The second and third lines are separate `sh` invocations, so **the working
directory persists between them** — the file written by one was read by the
next.

## SD refuses to nest

```
:sh sd
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

**A program's `OS.EXECUTE` is open to you in the same way**, and so are the
screen editors `edit` and `micro`, which run as Windows programs. The form
`execute 'sh …'` from a program goes through the `sh` verb itself.

## Over ssh and the API

**The same is true of a remote session.** An ssh or API session is you, and
may reach the operating system as you. Anyone who has the account password —
and, over ssh, your Windows sign-in — can therefore run Windows commands as
your Windows user. That is one reason every session asks for the account
password; see [Security](12-security.html).

## On a managed computer

**The SD Core for Linux server can put `sh` on the list of denied commands**
(see [Managed mode](15-managed-mode.html)); it then needs `ADMIN` first.
**That does not close the operating system off today:** `!` — the same verb
under another name — cannot be named on the list, and a program's
`OS.EXECUTE` is not a command the list can hold.

## See also

[Administrator commands](06-administrator-commands.html) ·
[Sessions and locks](06a-sessions-and-locks.html) ·
[Security and the operating system](12a-security-and-the-operating-system.html).
