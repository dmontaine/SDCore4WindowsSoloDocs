Title: Installing
Subtitle: What the installer asks, what it puts where, and the one administrator step.

**The installer is one file, `sd-solo-setup-WS2.0-0.exe`, and it installs for
the Windows user who runs it.** Everything goes into that user's profile, in
`%USERPROFILE%\SDCoreSolo`. Nothing is installed for other users of the
computer, and there is no Windows service.

## Before you start

| | |
|---|---|
| Windows | Windows 10 or 11, 64-bit |
| Rights | an ordinary user account. The installer asks for administrator consent **once**, for the steps that change the computer rather than your profile |
| The package | the installer, with two folders beside it: `ssh-server\` holding Microsoft's OpenSSH server MSI, and `python\` holding python.org's Python installer (`python-3.x.x-amd64.exe`) |

**The two folders are part of the package, every time.** If either is
missing the installer stops at once with *"Invalid installation package.
Missing beside this installer:"* and the name of what is missing, before it
changes anything. Whether each package is actually *installed* is decided
below; carrying them is not optional.

**The package works offline and from read-only media.** It reads the two
packages from beside itself and never downloads anything, so the same folder
serves as a download and as a USB stick for installing several computers.

**The multiuser SD Core for Windows can be installed on the same computer.** The
two products keep their own programs, ports, firewall rules and shared memory:
SD Core Solo's API is on 4249 and its ssh is on **4251**, while SD Core uses
the computer's ordinary ssh port, 22. See [ssh access](08-ssh-access.html).
Earlier releases refused to install beside SD Core.

## What you are asked

### Saved data — only where an uninstall kept some

**If `%USERPROFILE%\SDCoreSolo` holds data that an earlier uninstall kept** (the
account `sduser`, `sd.conf` and the kept settings, with a `.sdcore-kept` file),
the first question is *Reload your saved data and configuration into this new
install?* — **Yes** reloads them, **No** starts clean. Either way the old folder
is moved aside to `SDCoreSolo.kept-<date and time>`, never deleted. A reload
keeps the passwords it finds and skips the password pages below and the start
question; otherwise the passwords are asked as for any new installation. See
[Upgrading and uninstalling](01a-upgrading-and-uninstalling.html).

### 1. The passwords

Asked in this order, each typed twice, hidden as you type:

| | |
|---|---|
| **Account password** | the password every SD session asks for — at the keyboard, over ssh and through the API |
| **Administrator password** | unlocks the administrator commands, with `admin` |
| **Global password** | **optional — leave it blank if no SD Core server manages this computer.** The SD Core for Linux server signs in with it, and it also unlocks the administrator commands |

**A computer is managed if and only if it has a global password.** Leave it
blank and no SD Core server manages the computer: it is a database for this
computer alone, and there is nothing else to choose. Enter one and the computer
is managed by an SD Core for Linux server — see [Managed mode](15-managed-mode.html).

**Whether the computer has a global password cannot be changed later except by
a new installation**: no command creates or removes one, and only a session the
server signs in with can change it (`set.password global`). To add one to a computer that has none, install again as
described under *Changing any of it afterwards*. An upgrade never adds, changes
or removes it.

**Every password needs at least 8 characters, with a lower-case letter, an
upper-case letter, a digit and a symbol** — letters, digits and punctuation
only, no spaces. The global password, when there is one, must differ from both
of the others; otherwise the server, signing in with the same name, would land
in an ordinary session. See [The account and its passwords](05-account-types.html).

### 2. The API and ssh

Asked on every new installation, whether or not there is a global password.

| | |
|---|---|
| **Provide the SD Core API (port 4249)** | starts the API listener. Off by default |
| **Let other computers on your network reach it** | opens port 4249 in Windows Firewall. Without it the API answers this computer only |
| **Provide Solo's ssh server (port 4251)** | shown when an OpenSSH server is already installed. On by default. Unticked, Solo has no ssh server |
| **Let other computers on your network connect to this one over ssh (port 4251)** | Solo's ssh server listens on every IPv4 address (`0.0.0.0`) and its own firewall rule for port 4251 is open to the network. Without it, the server listens on `127.0.0.1` only and answers this computer only |
| **Install the OpenSSH server** | shown when no ssh server is installed. Installs the MSI from `ssh-server\` and sets up Solo's own ssh server. Off by default |
| **Let other computers on your network connect to this one over ssh (port 4251)** | the same, for the box above |

**As delivered the API is off and ssh answers this computer only**, as in SD
Core for Windows: leave every box as it is and nothing is reachable from
another computer.

**When Solo's ssh server is on,** it runs on port 4251 and starts with Windows.
You sign in to it with your Windows account name and password, and it starts
`sd-solo`. Windows' own ssh server and its port 22 are not changed. See
[ssh access](08-ssh-access.html).

**An SD Core server that manages the computer connects from elsewhere**, through
the API and ssh. With either off, or open to this computer only, the server
cannot reach the computer until the choice is changed. An administrator who
sets up many computers gives these two answers in the control file.

### 3. Shell and OS access — on a new installation and on an upgrade

A page of its own, **Shell and OS access**, with two boxes, both unticked:

| | |
|---|---|
| **Shell (SH)** | turns on the `sh` switch: the `sh` verb |
| **OS access (OS.EXECUTE, files outside the account, Python, editors)** | turns on the `os` switch: `OS.EXECUTE`, files and directories outside the account's own directory, the Python helper and the editors `edit` and `micro` |

**Both are off as delivered**, and the default is no. The installer is the one
place where a person who wants them says so once; the administrator can change
either later with `set.os.access`. **Either switch lets whoever signs in to SD —
at the keyboard, over ssh or through the API — do what your Windows user can
do**: read [Operating system access](06b-operating-system-access.html) before
ticking one.

**An upgrade asks the same page.** A Solo from before 2.0-0 had its shell and
OS access always on, so an existing Solo would otherwise lose them without a
word; tick each you want to keep. Unticked, an upgrade turns nothing on, and the
switches stay as the account's record has them (off, unless an earlier
`set.os.access` turned one on). A silent installation, which shows no page, turns
nothing on either. The installer writes the answer with `set.os.access`, reads
both switches back, and says what they are in `install-summary.log`.

### 4. Where sessions start — on a new installation only

A page of its own, **Where sessions start**, with one box, unticked:
**Start new sessions at the command prompt, not the home screen**.

**Unticked, a session starts at the home screen**, the full-screen menu of
tiles, as delivered; nothing is written. **Ticked, it starts at the command
prompt**: the installer runs `set.start.default terminal` and reads the file it
writes back, and `install-summary.log` says which was chosen. The administrator
can change it later with `set.start.default menu` or `set.start.default
terminal`, or on the Starts in row of `tool.menus`. **An upgrade does not ask**:
it keeps the start the installation has. A silent installation, which shows no
page, keeps the home screen.

### 5. PDF reader — only where one is installed and none is allowed

The `pdf` command shows the documentation in a PDF reader, even though your
account may not run programs, so it starts only a reader the administrator has
allowed, and **none is allowed as delivered**. When this computer's App Paths
names a reader SD knows (`msedge`, `acrobat`, `acrord32`, `foxitpdfreader`,
`sumatrapdf`, `pdfxedit`, `firefox` or `chrome`, the first found) and none is
allowed yet, a page **PDF reader** asks with one box, unticked: **Allow msedge to
show the documentation (the pdf command)**. Ticked, the installer runs
`set.viewer add pdf msedge`, which tests the program and refuses one that fails,
and `install-summary.log` says what was chosen. Asked on a new installation and
on an upgrade; a silent installation, which shows no page, allows nothing. Later:
`admin`, then `set.viewer` or the Bypass programs page. See
[Operating system access](06b-operating-system-access.html).

**Allowing a reader is a loosening:** that one program may then be started for an
account whose `os` switch is off. A graphical reader can open other files the
account's user may read and follow links. Allow only a reader you trust.

### What is not asked, because it always happens

| | |
|---|---|
| **Python** | installed for you, from `python\`, unless a Python 3.13 or later is already installed. It is a per-user install and needs no administrator rights |
| **PATH** | `%USERPROFILE%\SDCoreSolo\usr\bin` is added to your PATH, so `sd-solo` works from any new window |

## The one administrator step

**One consent prompt appears near the end.** It runs a single script,
`solo-machine.ps1`, which does the four things that are the computer's rather
than yours:

- registers the scheduled task **SD Core Solo**, which starts SD at every
  Windows start-up, as you, whether or not you are signed in (a standard
  account gets a different task, below);
- opens or restricts the firewall rules chosen above;
- installs the OpenSSH MSI, when that was chosen;
- when Solo's ssh server was chosen: makes the administrators-only folder
  `C:\ProgramData\SDCoreSolo\ssh` (the ssh server's configuration and host key,
  with its permissions checked afterwards) and registers the scheduled task
  **SD Core Solo SSH**, which starts Solo's own ssh server on port 4251 at every
  Windows start-up, as SYSTEM, so that your Windows password can sign you in;
  it also removes what an earlier release added to Windows' ssh settings.

**If you decline the prompt, SD is installed but does not start at start-up**,
and Solo's ssh server and the firewall rules are not set up. The installer says
which steps did not complete.

**A standard Windows account** — one that is not an administrator — gets the
same prompt, but Windows asks for an administrator's name and password in it
instead of a click, and the installation goes on with them. Windows refuses the
start-up task above for such an account ("Access is denied"), whoever sets it
up, so the installer registers a task that starts SD, with no window, **when you
sign in**. SD is therefore not running between a restart and your sign-in.
Signing out ends SD, and leaves it marked as not shut down cleanly; the sign-in
task clears that and starts SD again, and writes what it did to
`solo-start.log` in the `SDCoreSolo` folder. Solo's ssh server and the firewall
rules are set up as for any account, and `install-summary.log` says which of the
two tasks was made.
## What lands where

Everything is under `%USERPROFILE%\SDCoreSolo`:

| | |
|---|---|
| `usr\bin` | `sd-solo.exe` and the other programs, the client libraries, and the two full-screen editors the `edit` and `micro` verbs run |
| `usr\clients` | the client libraries for applications, 64-bit and 32-bit. See [Client distribution](10-client-distribution.html) |
| `sd.conf` | the configuration. See [Configuration](16-configuration.html) |
| `sdsys` | SD's own files: the system programs (compiled only — no source is installed), messages, the dictionaries, the credential store |
| `user_accounts\sduser` | the one SD account, and your data |
| `sd-tls` | the API's TLS key, made the first time a client connects |
| `install-summary.log` | what every installer step did — read this first when something did not work |

**The source is beside the installer, not installed.** SD Core Solo is free
software under the GNU General Public License, version 3 or later. Its complete
source - the C, the system programs' BASIC (`gpl.bp`) and the build and
installer scripts - is `SDCore4WindowsSolo-source-<version>.zip` beside
`sd-solo-setup-<version>.exe`, with a `.txt` file naming the commit it was
built from.

**Uninstalling asks to keep or delete your data and configuration.** Keep leaves
the account, `sd.conf`, the passwords, the audit trail, the TLS and ssh keys and
your settings, and removes the system files, the deny list, `global.bp.out` and
the logs; a new installation then offers them back, and a reload asks no
password. See [Upgrading and uninstalling](01a-upgrading-and-uninstalling.html).

**The folder can be moved.** SD finds its files from where its programs are,
not from a path written into it, so a copied tree works under another Windows
user or on another computer — as long as the account password is known. The
startup tasks, the firewall rules and PATH are the installer's
and do not move with it.

## Continued in

[Installing Many Computers](01c-installing-many-computers.html) — the control
file for installing many computers, what the installer checks when it
finishes, when a step fails, and changing any of it afterwards.

[Upgrading and uninstalling](01a-upgrading-and-uninstalling.html) —
upgrading an existing installation, and uninstalling.

[Differences from SD Core for Windows W2.0-0](01b-differences-from-sd-core-w2-0-0.html)
— what Solo leaves out, adds and does differently.
