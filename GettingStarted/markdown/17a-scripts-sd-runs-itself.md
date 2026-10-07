Title: The Scripts SD Runs For Itself
Subtitle: The scripts the installer and the startup tasks run, which nobody types.

This page continues [The Installed Scripts](17-the-installed-scripts.html).

## The ones the installer runs

**You should not need to run any of these** (`solo-ssh-firewall.ps1` is the one
you may, on [The Installed Scripts](17-the-installed-scripts.html)). They are
listed so that a name in the installer's report or in an error message can be
looked up. Each step's report is appended to
`%USERPROFILE%\SDCoreSolo\install-summary.log` under a title, which is where to
read what actually happened.

| | |
|---|---|
| `solo-setup.ps1` | the steps that run **as you**, with no elevation |
| `solo-machine.ps1` | the one step that needs an **administrator**, behind a single consent prompt |
| `solo-api-listener.ps1` | switches the API listener on or off in `sd.conf`; `solo-machine.ps1` runs it |
| `solo-sshd.ps1` | sets up and looks after Solo's own ssh server |
| `solo-ssh-firewall.ps1` | the firewall rule for Solo's ssh port |
| `solo-start.ps1` | what the sign-in startup task runs, on an account that cannot have a start-up task |
| `internal-marker.ps1` | a helper the first one loads; it defines two functions and does nothing else |

### `solo-setup.ps1`

**It runs after the files are copied**, and does these in order:

1. starts SD, because sessions need a started SD;
2. makes the account `sduser`;
3. sets the passwords the installer collected — the account password, the
   administrator password and, when one was given, the global password — and,
   from a control file, the list of denied commands;
4. **on an upgrade only**, brings the dictionaries up to the release and runs
   `UPDATE.ACCOUNTS`, because an upgrade replaces the shipped VOC records but
   does not rebuild the account's own;
5. removes the system programs' source from the VOC and, where there is a
   global password, makes the global catalogue match `GLOBAL.BP.OUT` again after an upgrade has
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
| **Install** | makes the API firewall rule as chosen, **then** switches the API listener on or off with `solo-api-listener.ps1`, **then** registers and starts the scheduled task **SD Core Solo** — in that order, so the first start of SD that listens finds its rule already there and Windows shows no alert — and, when the API was chosen, waits until port 4249 is listening and says so; installs the OpenSSH package when that was chosen; opens or restricts the ssh port rule as chosen; when Solo's ssh server was chosen, has `solo-sshd.ps1 -Install` make the administrators-only ssh folder, then registers and starts the task **SD Core Solo SSH**, which runs Solo's own ssh server as SYSTEM (below) |
| **Upgrade** | registers the scheduled tasks again — an upgrade does not revisit the choices, and registers the ssh task only where Solo's ssh server is already set up. It moves an API firewall rule an earlier release left on port 4243 to 4249, keeping who may reach it, and leaves `sd.conf` alone. It also removes what an earlier release added to Windows' ssh settings |
| **Remove** | takes away both tasks, stops Solo's ssh server and deletes its administrators-only folder, and takes away the API rule and the ssh port rule. **Windows' own ssh rule for port 22 is Microsoft's and is never touched** |

**The task runs `sd-solo -start` as you, at Windows start-up, whether or not you are
signed in** — that is what lets the API work before anyone signs in.
For a user who is an administrator, Task Scheduler gives the task the full
administrator token, and **`sd-solo.exe` drops it itself**, so SD runs on an
ordinary token whatever the task does. See [Running SD](03-running-sd.html).
**Windows refuses that start-up task for a standard (non-administrator) account**,
and then the installer registers a task of the same name that runs
`solo-start.ps1` when you **sign in**, below. `install-summary.log` says which
of the two was made.

**The task SD Core Solo SSH runs `sshd.exe` itself, as SYSTEM, at Windows
start-up, whether or not you are signed in,** against the configuration in
`C:\ProgramData\SDCoreSolo\ssh`. **It never runs a script of yours as SYSTEM**:
everything it reads is in that administrators-only folder, which the installer
makes and whose permissions it reads back before going on. Starting it again, up
to three times a minute apart, is left to Task Scheduler if the server stops.

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

**Sets up and looks after Solo's own ssh server.** The installer's administrator
step runs `-Install` and `-Uninstall`; `solo-sshkey.ps1` uses `-Prepare`, which
needs no elevation.

```
powershell -ExecutionPolicy Bypass -File "%USERPROFILE%\SDCoreSolo\solo-sshd.ps1" -Show
```

| Switch | What it does |
|---|---|
| `-Prepare` | **ordinary user.** Makes your optional key file `%USERPROFILE%\SDCoreSolo\ssh\authorized_keys` if it is missing, deletes what the first build of this version left in that folder, moves the SD Core server's key out of `%USERPROFILE%\.ssh\authorized_keys` if an earlier release put it there, and stops. It reads the server's host-key fingerprint from the administrators-only folder and writes nothing there |
| `-Install` | **elevated.** Makes `C:\ProgramData\SDCoreSolo\ssh` (administrators and SYSTEM may change it; users may read it), the host key (kept across upgrades) and the configuration, **reads the permissions back and stops with `PROBLEM=` lines if anyone else could write there or read the private key**, and checks the configuration with `sshd -t`. It refuses a login name or folder name that could add a line to the configuration |
| `-Stop` | **elevated.** Ends the `sshd.exe` that was started from this configuration, found by its command line — never the Windows ssh service. The process a SYSTEM task started cannot be inspected from an ordinary window; there `-Stop` says *"cannot inspect"*, exits `1` and stops nothing, rather than report that there was nothing to stop |
| `-Uninstall` | **elevated.** `-Stop`, then deletes `C:\ProgramData\SDCoreSolo` |
| `-Show` | reports the port, whether it is listening and which process holds it, and changes nothing |
| `-PrintConfig -OsUser <name>` | prints the configuration that would be written, and writes nothing |

**The port is fixed at 4251 and is not a parameter.** The installer rewrites the
configuration, so editing it by hand does not last.

### `solo-ssh-firewall.ps1`

**Decides whether other computers may reach port 4251.** It is the one step of
Solo's ssh that needs an administrator, so it is run from the installer's
consent prompt, or by you from an elevated PowerShell. See
[ssh access](08-ssh-access.html) for the four switches and their exit codes.

### `solo-api-listener.ps1`

**Switches the API listener, `APIPORT` in `sd.conf`, on or off.**
`solo-machine.ps1` runs it with `-On` after it has made the API firewall rule
and before it starts the startup task, or with `-Off` when the API was not
chosen. You can run it yourself; [API access](09-api-access.html) has the
commands.

| Switch | What it does |
|---|---|
| `-On` | makes `APIPORT=4249` the one active line: it un-comments the commented line, rewrites an older `APIPORT=4243`, or adds the line where there is none |
| `-Off` | comments the active line out |
| `-Show` | reports whether the listener is on, and changes nothing |
| `-ConfPath <file>` | works on that file instead of the `sd.conf` beside the script |

**It changes the file, not the running SD**: the listener is read once, as SD
starts. It keeps the file's own line endings, reads the file again before it
says it is done, and says so on every run which file it used. Exit **0** the
file now says what was asked, **1** it could not be written or did not read back
as asked, **2** it could not tell — no such file, or not exactly one of
`-On`, `-Off` and `-Show`.

**It is the listener, not the firewall.** `APIPORT` decides whether SD opens a
socket at all; `api-firewall.ps1` decides who may reach it.

### `solo-start.ps1`

**What the sign-in startup task runs**, as you, with no window and no
elevation. It exists because Windows refuses the start-up task for a standard
account, and because signing out ends SD and leaves its shared memory behind, so
a plain `sd-solo -start` at the next sign-in would refuse to start. It starts SD;
if that failed and no SD is running, it clears the leftover with `sd-solo -stop`
and starts once more. **It never stops an SD that is running, and it does not
loop.** What it did is written to `solo-start.log` beside it. Exit **0** SD is
running when it ends, **1** it is not.

### `internal-marker.ps1`

**`sd-solo -internal` is how the installer runs SD's setup steps, and it is admitted
only while a marker file exists.** This script writes that file immediately
before each internal session, and SD deletes it on admission, so an
un-used marker authorises exactly one later session and then expires. It is
loaded by the installer's other scripts, never run on its own.

**It is a speed bump, not a boundary**, and says so: anyone who can write into
the `sdsys` folder can write the file by hand, and in Solo that is you. See
[Security](12-security.html#the-installers-own-door).

## Continued in

[The Scripts a Verb Calls](17b-scripts-a-verb-calls.html) — the scripts the
verbs call, and the one SD runs as it starts
