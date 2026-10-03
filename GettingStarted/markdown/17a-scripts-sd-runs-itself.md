Title: The Scripts SD Runs For Itself
Subtitle: The scripts the installer, the verbs and SD itself call, which nobody types.

This page continues [The Installed Scripts](17-the-installed-scripts.html).

## The ones the installer runs

**You should not need to run either of these.** They are listed so that a name
in the installer's report or in an error message can be looked up. Each step's
report is appended to `%USERPROFILE%\SDCoreSolo\install-summary.log` under a
title, which is where to read what actually happened.

| | |
|---|---|
| `solo-setup.ps1` | the steps that run **as you**, with no elevation |
| `solo-machine.ps1` | the one step that needs an **administrator**, behind a single consent prompt |
| `internal-marker.ps1` | a helper the first one loads; it defines two functions and does nothing else |

### `solo-setup.ps1`

**It runs after the files are copied**, and does these in order:

1. starts SD, because sessions need a started SD;
2. makes the account `sduser`;
3. sets the passwords the installer collected — the account password, the
   administrator password and, in managed mode, the global password — and, from
   a control file, the list of denied commands;
4. **on an upgrade only**, brings the dictionaries up to the release and runs
   `UPDATE.ACCOUNTS`, because an upgrade replaces the shipped VOC records but
   does not rebuild the account's own;
5. removes the system programs' source from the VOC and, in managed mode,
   makes the global catalogue match `GLOBAL.BP.OUT` again after an upgrade has
   replaced the catalogue;
6. stops SD, so that the scheduled task — which the next script registers —
   starts it and owns it.

**The passwords never appear on a command line or in a file.** The installer
puts them in its own environment for the moment it starts the script and clears
them afterwards; the script reads them, clears them from its own environment
before it starts anything, and writes each one to SD's standard input. Each
step is judged on the line the SD program prints on success, and a password
found in a session's output fails the step.

**Exit 0** every step passed, **1** a step failed, **2** it refused before
doing anything.

### `solo-machine.ps1`

**It is started by the installer, through one consent prompt, and is not meant
to be run by hand.** It acts for the Windows user who is installing, whichever
administrator approves the prompt.

| Action | What it does |
|---|---|
| **Install** | registers and starts the scheduled task **SD Core Solo**; installs the OpenSSH package when that was chosen or managed mode needs it; opens or restricts the API and ssh firewall rules as chosen; registers and starts the task **SD Core Solo SSH**, which runs Solo's own ssh server (below) |
| **Upgrade** | registers the scheduled tasks again — an upgrade does not revisit the choices, with one exception: on a managed computer with no firewall rule for port 4251 yet, it opens one. It also removes what an earlier release added to Windows' ssh settings |
| **Remove** | takes away both tasks, stops Solo's ssh server, and takes away the API rule and the ssh port rule. **Windows' own ssh rule for port 22 is Microsoft's and is never touched** |

**The task runs `sd-solo -start` as you, at Windows start-up, whether or not you are
signed in** — that is what lets the API work before anyone signs in.
For a user who is an administrator, Task Scheduler gives the task the full
administrator token, and **`sd-solo.exe` drops it itself**, so SD runs on an
ordinary token whatever the task does. See [Running SD](03-running-sd.html).

**The task SD Core Solo SSH runs `solo-sshd.ps1 -Run` as you, at Windows
start-up, whether or not you are signed in.** It is registered without a stored
password, so it can only ever run as you and with no more rights than you have
at the keyboard. Starting it again, up to three times a minute apart, is left to
Task Scheduler if the server stops.

**An earlier release wrote a block into Windows' `sshd_config`**, between
`# BEGIN SD Core Solo` and `# END SD Core Solo` markers, with a `Match User`
line for your Windows user. **This release removes exactly that block and
nothing else**, on an install, an upgrade and an uninstall. The file is checked
with `sshd -t` afterwards and put back if the ssh server rejects it, and the
Windows ssh service is restarted only if it was running and a block was removed.
Solo no longer writes to that file.

**Exit 0** every step passed, **1** a step failed, **2** it refused — not
elevated, or an input was missing.

**If you decline the consent prompt, the installer still finishes**, and lists
the steps that did not complete. See [Installing](01-installation.html).

### `solo-sshd.ps1`

**Prepares and runs Solo's own ssh server.** It is started by the task SD Core
Solo SSH, and by `solo-sshkey.ps1`, which uses its `-Prepare`. It needs no
elevation.

```
powershell -ExecutionPolicy Bypass -File "%USERPROFILE%\SDCoreSolo\solo-sshd.ps1" -Show
```

| Switch | What it does |
|---|---|
| `-Prepare` | makes `%USERPROFILE%\SDCoreSolo\ssh` with its host key, its configuration and its key file, moves the SD Core server's key out of `%USERPROFILE%\.ssh\authorized_keys` if an earlier release put it there, and stops |
| `-Run` | does `-Prepare`, checks the configuration with `sshd -t`, and runs `sshd.exe` in the foreground on port 4251. It refuses if something else already holds the port |
| `-Stop` | ends the `sshd.exe` that was started from this configuration, found by its command line — never the Windows ssh service. **Run it from an elevated PowerShell.** The process the startup task started cannot be inspected from an ordinary window, even your own; there `-Stop` says *"cannot inspect"*, exits `1` and stops nothing, rather than report that there was nothing to stop |
| `-Show` | reports the port, whether it is listening and which process holds it, and changes nothing |

**The port is fixed at 4251 and is not a parameter.** The configuration is
written fresh each time, so editing it by hand does not last.

### `solo-ssh-firewall.ps1`

**Decides whether other computers may reach port 4251.** It is the one step of
Solo's ssh that needs an administrator, so it is run from the installer's
consent prompt, or by you from an elevated PowerShell. See
[ssh access](08-ssh-access.html) for the four switches and their exit codes.

### `internal-marker.ps1`

**`sd-solo -internal` is how the installer runs SD's setup steps, and it is admitted
only while a marker file exists.** This script writes that file immediately
before each internal session, and SD deletes it on admission, so an
un-used marker authorises exactly one later session and then expires. It is
loaded by the installer's other scripts, never run on its own.

**It is a speed bump, not a boundary**, and says so: anyone who can write into
the `sdsys` folder can write the file by hand, and in Solo that is you. See
[Security](12-security.html#the-installers-own-door).

## The ones a verb calls

**Prefer the verb.** It reports what happened in the product's own words and
refuses when it cannot act; the script does the work and assumes the caller
knew what they were doing.

| | Called by |
|---|---|
| `sd-path.ps1` | `append.sd.path on` \| `off` — puts SD's program folder on **your** PATH, or takes it off |
| `micro-home.ps1` | the editor programs, before `micro` starts — see below |
| `solo-sshkey.ps1` | the API's ssh key request (49), on a managed computer — see below |

### `sd-path.ps1`

```
powershell -ExecutionPolicy Bypass -File "%USERPROFILE%\SDCoreSolo\sd-path.ps1" -Show
powershell -ExecutionPolicy Bypass -File "%USERPROFILE%\SDCoreSolo\sd-path.ps1" -Add
powershell -ExecutionPolicy Bypass -File "%USERPROFILE%\SDCoreSolo\sd-path.ps1" -Remove
```

**It changes your own PATH, not the computer's, and needs no elevation.** Exit
**0** applied (or `-Show` succeeded), **1** failed, **2** refused. It reads and
writes the same registry value the installer does, and it keeps a PATH entry
that refers to a variable (`%SystemRoot%`) as a reference rather than expanding
it. Every install and upgrade puts SD on your PATH again, so `-Remove` lasts
until the next one. See [Administrator commands](06-administrator-commands.html).

### `micro-home.ps1`

**`micro` writes to a configuration folder when it saves**, and this script
gives you one you can write to, in your profile, and prints where it is:

```
MICROHOME=C:\Users\you\.micro
```

The editor program reads that line and nothing else from it. It exits 0 with
that line, or 1 without it. It exists because `micro` printed a false
*"Permission denied"* on every save when its configuration folder was read-only;
**the folder has to be per-user and writable only by its owner**, because
`micro` loads and runs plug-ins from it. Nothing about it is a setting — see
[Programmer commands](07-programmer-commands.html#editors).

### `solo-sshkey.ps1`

**Run by SD itself, in your own session, when the SD Core server asks to add,
remove or list its ssh key** — see [Managed mode](15-managed-mode.html). It is
not for typing: it takes a request verb (`ADD`, `REMOVE` or `LIST`) and a key or
fingerprint, and prints lines SD reads back (`RESULT=ADDED`, `FPR=SHA256:...`,
`ERROR=...`). **It needs no elevation**, because it writes only Solo's own key
file, `%USERPROFILE%\SDCoreSolo\ssh\authorized_keys`, and it refuses unless
`sshd.exe` is installed. It runs `solo-sshd.ps1 -Prepare` first, which also
tells it Solo's ssh server key and port. It touches only lines it wrote itself,
tagged `sdcoresolo-managed`, and keeps at most four. Exit **0** with a `RESULT=` line,
**1** with an `ERROR=` line and nothing changed.

## See also

[Installing](01-installation.html) covers what the installer puts on the
computer and what an upgrade replaces.
[Security](12-security.html) covers what each password guards.
