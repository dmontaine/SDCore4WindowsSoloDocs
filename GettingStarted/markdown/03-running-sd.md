Title: Running SD
Subtitle: How SD starts, stopping it, the command line, and what to do when the last shutdown was not a clean one.

**SD is started by a scheduled task, and it is already running.** You do not
type `sd-solo -start`.

| | |
|---|---|
| Task name | **SD Core Solo**, in Task Scheduler's library |
| When | at every Windows start-up |
| As whom | you — whether or not you are signed in, so ssh and the API work before anyone signs in |
| With what rights | an ordinary, unelevated token, even when your Windows account is an administrator. SD drops the administrator rights itself |
| Created by | the installer, in its one administrator step; registered again by every upgrade |
| Removed by | the uninstaller |

**There is no Windows service.** The multiuser SD Core for Windows ran SD as
a service under LocalSystem; Solo runs it as the one user who owns it.

## Starting and stopping

```
sd-solo -stop
sd-solo -start
```

**Stopping SD ends every session**, including API and ssh sessions. **Neither
needs an elevated window**: SD, its data and its programs are yours, and
Windows' own protection of your profile is the gate.

**Starting it by hand runs it until you stop it or Windows restarts**; the
task starts it again at the next start-up. Running the task from Task
Scheduler does the same thing.

## `sd-solo -start` and `sd-solo -stop` tell the truth

**"SD is already started" is only said when the daemon really is running**,
and it gives the Windows process id — the number Task Manager and
`Stop-Process` use.

**If SD's shared memory is there but the daemon is not** — what a killed or
crashed SD leaves behind — `sd-solo -start` says so and tells you to run `sd-solo -stop`
first. It does not clear it for you, because that would end any sessions
still attached; the count of those is printed so you can decide.

**`sd-solo -stop` checks that the daemon actually stopped**, and warns with its
process id if it did not.

**A session still running ten seconds after the stop is ended.** `sd-solo -stop`
and `sd-solo -restart` end it with SIGKILL and say how many they ended, for
example `1 process(es) still running after ten seconds were ended with SIGKILL`.
A process whose program is not in SD's own folder is not touched. A session
ended this way does not run its own logout.

> **Known limit.** If the shared memory has already gone, `sd-solo -stop` has
> nowhere left to read the daemon's process id from and cannot report on it.
> Check by hand:
>
> ```
> Get-Process sdwind
> ```

## After an unclean shutdown

If SD is stopped abruptly — the power goes, or the process is killed — it
leaves its shared memory behind, and **on Windows that survives a restart.**
Nothing from before a restart can still be using it, so SD discards it at the
next start and starts normally, printing:

```
Discarding the shared segment left by the previous boot -
SD did not shut down cleanly.
```

A segment belonging to a running SD is never touched.

## SD will not start a second time inside itself

If you leave SD with **`sh`** and then type `sd-solo` in that shell, it says so and
returns you to the session you already have. `sh` is yours to use, with no
`admin` — see [Operating system access](06b-operating-system-access.html).

## The command line

```
sd-solo                  enter the account, after the account password
sd-solo <command>        run one command and return, using the kept password
sd-solo -quiet           suppress the displays on entry
sd-solo -u               list current sessions
sd-solo -k <n> | -k all  end session n, or every session
sd-solo -start           start SD
sd-solo -stop            stop SD
sd-solo --version        report the version
sd-solo --help           a summary of these
```

**`sd-solo <command>` runs and exits, with no prompt.** It uses a copy of the
account password Windows keeps for you (see
[The account and its passwords](05-account-types.html)), which is what lets a
script or a scheduled job use SD. **If that copy is missing or no longer
matches**, what happens depends on where the input comes from:

| | |
|---|---|
| typed at a terminal | refused: *A command given on the sd command line needs the account password on its input* |
| piped in | the first line of the input is taken as the password, once — so a job can supply it itself |

`set.password` keeps the copy up to date when you change the password.

**`sd-solo -a` and `sd-solo -a<name>` have nothing to choose between**: there is one
account, `sduser`, and every session lands in it.

**`sd-solo -u` and `sd-solo -k` need no `admin`.** They are switches on the program,
outside any SD session, and like `-start` and `-stop` they are the business
of the user who owns SD. Inside a session, the same jobs are `LISTU` and
`LOGOUT`, which do need `admin` — see [Sessions and locks](06a-sessions-and-locks.html).

## Where things are

Everything is under `%USERPROFILE%\SDCoreSolo`:

| | |
|---|---|
| Programs | `usr\bin\` |
| The changelog | `changelog` |
| Configuration | `sd.conf` |
| SD's own files | `sdsys\` |
| Your account | `user_accounts\sduser\` |
| Audit trail | `sdsys\audit` |
| Error log | `sdsys\errlog` |
| What the installer did | `install-summary.log` |

**Do not move the programs out of `usr\bin`.** The runtime DLLs ship beside
`sd-solo.exe` deliberately — Windows searches the program's own folder before
`PATH`, which keeps Git for Windows's rival `msys-2.0.dll` from being picked
up. **That failure makes SD report "SD has not been started" while it is
running**, which is worth recognising because it looks like nothing else.
The whole `SDCoreSolo` folder can be moved; its parts cannot be separated.

**The Start Menu has an entry, SD Core Solo**, in your own Start Menu (the
uninstaller removes it). `sd-solo` is also on your PATH: open a window and type
it.

**SD Core Solo always runs in a window of its own**, as a Windows application
does. The Start Menu entry opens it; typing `sd-solo` at a PowerShell or cmd
prompt opens it too, and the prompt comes back at once, as it does after
`notepad`. The window is a classic console window that SD controls: it starts
at 140 columns by 43 lines (make it bigger whenever you like) and is titled
**SD Core Solo**. It is designed for Windows 11 on a screen of at least 1920 by
1080.

When it ends normally (`OFF`) its window closes. **When it ends with an error,
for example a refused sign-in, the window stays open** with "SD ended with
status *n*. Press Enter to close this window." so the reason can be read.

These still run where you type them, because they open no session:
`sd-solo` with a command, the switches such as `sd-solo -start` and
`sd-solo -stop`, anything whose input or output is redirected, and an ssh
session.

**The full-screen pages need a window of at least 140 columns by 43 lines**,
which is the size SD's own window starts at. For full screen, press
**Alt+Enter** (in SD's window F11 goes to SD).
