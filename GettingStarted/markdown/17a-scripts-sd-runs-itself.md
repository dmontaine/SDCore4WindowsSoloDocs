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
| **Install** | registers and starts the scheduled task **SD Core Solo**; installs the OpenSSH server when that was chosen or managed mode needs it; opens or restricts the API and ssh firewall rules as chosen; writes the ssh setting that starts `sd` for your ssh sign-in, and sets the ssh server to start with Windows |
| **Upgrade** | registers the scheduled task again, and nothing else — an upgrade does not revisit the choices |
| **Remove** | takes away the task, the API rule and the ssh setting. **The ssh firewall rule is Microsoft's and is left as it is** |

**The task runs `sd -start` as you, at Windows start-up, whether or not you are
signed in** — that is what lets ssh and the API work before anyone signs in.
For a user who is an administrator, Task Scheduler gives the task the full
administrator token, and **`sd.exe` drops it itself**, so SD runs on an
ordinary token whatever the task does. See [Running SD](03-running-sd.html).

**The ssh setting is a block appended to `sshd_config`, between markers**, so
that the uninstaller can remove exactly it and nothing else:

```
# BEGIN SD Core Solo - added by its installer, removed by its uninstaller
Match User <your Windows user>
    ForceCommand "%USERPROFILE%\SDCoreSolo\usr\bin\sd.exe"
    DisableForwarding yes
# END SD Core Solo
```

**Only your user is matched**; sign-in is the ssh server's own, with your
Windows password or key. `DisableForwarding` is there because `ForceCommand`
does not by itself stop port forwarding. The file is checked with `sshd -t`
afterwards and put back if the ssh server rejects it. See
[ssh access](08-ssh-access.html).

**Exit 0** every step passed, **1** a step failed, **2** it refused — not
elevated, or an input was missing.

**If you decline the consent prompt, the installer still finishes**, and lists
the steps that did not complete. See [Installing](01-installation.html).

### `internal-marker.ps1`

**`sd -internal` is how the installer runs SD's setup steps, and it is admitted
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
`ERROR=...`). **It needs no elevation**, because it writes only
`%USERPROFILE%\.ssh\authorized_keys`, and it refuses unless the installer's ssh
block is in place. It touches only lines it wrote itself, tagged
`sdcoresolo-managed`, and keeps at most four. Exit **0** with a `RESULT=` line,
**1** with an `ERROR=` line and nothing changed.

## See also

[Installing](01-installation.html) covers what the installer puts on the
computer and what an upgrade replaces.
[Security](12-security.html) covers what each password guards.
