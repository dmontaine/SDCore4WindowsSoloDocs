Title: Your first thirty minutes
Subtitle: From a finished install to a file with data in it, an administrator command, and a command run from a script.

This page assumes SD Core Solo for Windows is installed. It is a walkthrough,
not a reference — every step links to the page that explains it properly.

## 1. Start SD

**Open a new PowerShell or Command Prompt window** — a window opened before
the install does not have SD on its PATH yet — and type:

```
sd-solo
```

**SD is already running.** A scheduled task starts it at every Windows
start-up, so you do not type `sd-solo -start`. See [Running SD](03-running-sd.html).

**It asks for the account password**, the one you chose when installing. On a
computer installed from a control file there is none yet, and it asks you to
choose one now:

```
This account has no password yet. Choose one now - SD Core Solo for Windows asks for it every time it is used.
```

The sign-on banner names the product and its version, `WS1.1-3`.

## 2. Look around

```
who
listf
term
```

| | |
|---|---|
| **`who`** | the account — always `sduser` |
| `listf` | the files in it |
| **`term`** | your terminal type and page size — should say `Device : windows` |

**If `term` says something else and your arrow keys do not work**, run
`term windows` for this session and see
[Other hardening](13-hardening.html#the-terminal).

**`term` also reports the page size, and SD's default is 120 × 36 — not
80 × 24.**

## 3. Make a file and put something in it

```
create.file customers
ed customers 1001
```

**`ed`** is the **line** editor. If you would rather have a full screen,
**`edit`** opens the same record in Microsoft Edit and **`micro`** opens it in
micro; both come with SD. See
[Development and file commands](07-programmer-commands.html#editors).

In **`ed`**: `i` to insert, type your lines, a full stop on its own line to stop
inserting, then `fi` to file and exit.

> **You do not have to write programs in `ed`.** The account's `bp` file is a
> **directory file** — an ordinary Windows folder with one file per program —
> so Notepad++, VS Code or any text editor works on it just as well:
>
> ```
> %USERPROFILE%\SDCoreSolo\user_accounts\sduser\bp
> ```
>
> Save the file, then **`basic`** and **`catalog`** it from inside SD as usual.
> SD folds CR+LF line endings on the way in — see
> [Other hardening](13-hardening.html#line-endings).

```
list customers
count customers
```

**Commands are lower case now.** Typing `LIST` still works — SD tries what you
typed, then lower case, then upper. See [Lower case](11-lower-case.html).

**THIS IS THE POINT AT WHICH MOST THINGS SHOULD FEEL LIKE OpenQM.** If
anything in ordinary data work behaves differently and is not described in this
set, that is worth reporting.

## 4. An administrator command

```
listu
```

```
Command requires administrator privileges
```

**That is the administrator gate.** Unlock it for this session:

```
admin
```

Type the administrator password — or, on a managed computer, the global
password — and `listu` works, and so does every other administrator command,
until you leave. See [Administrator commands](06-administrator-commands.html).

## 5. Leave

```
off
```

## 6. A command from a script

**From a PowerShell or Command Prompt window**, not from inside SD:

```
sd-solo list customers
```

**It runs the one command and returns, with no password prompt.** A command on
the command line uses a copy of the account password that Windows keeps for
you — which is what lets a script or a scheduled job use SD. See
[Scheduled jobs](04-scheduled-jobs.html).

## What to try next

1. **Your own application data.** **There is no restore utility**, so the
   way in is a short BASIC program that reads your exported data and writes the
   records. Then query it — the query processor is where most of the surface
   area is.
2. **A client program against the API**, if you chose it when installing. It
   signs in as `sduser` with the account password, on port 4249, and needs a
   client library from this release. **The DLL goes in the same directory as
   the application that loads it**, and its architecture must match. See
   [API access](09-api-access.html) and
   [Client distribution](10-client-distribution.html).
3. **ssh**, if the OpenSSH package is installed: `ssh -p 4251 <your Windows
   user>@127.0.0.1` asks for your Windows password, then lands in `sd-solo`, which
   asks for the account password. There is no key to set up first. See
   [ssh access](08-ssh-access.html).

## When something goes wrong

| | |
|---|---|
| Something SD did, and who did it | `audit`, in `%USERPROFILE%\SDCoreSolo\sdsys` |
| Diagnostics, and API connections | `errlog`, same place |
| The installation itself | `%USERPROFILE%\SDCoreSolo\install-summary.log` |

[Other hardening](13-hardening.html#the-logs) explains which log answers which
question.

**When you report something, say which build.** The release stamp is on the
sign-on banner, in `sd-solo --version`, and in
`%USERPROFILE%\SDCoreSolo\changelog`.
