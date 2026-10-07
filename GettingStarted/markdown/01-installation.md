Title: Installing
Subtitle: What the installer asks, what it puts where, and the control file for installing many computers.

**The installer is one file, `sd-solo-setup-WS1.1-3.exe`, and it installs for
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
account `sduser` and `sd.conf`, with a `.sdcore-kept` file), the first question
is *Reload your saved data and configuration into this new install?* — **Yes**
reloads them, **No** starts clean. Either way the old folder is moved aside to
`SDCoreSolo.kept-<date and time>`, never deleted, and the passwords below are
asked as for any new installation. See
[Upgrading and uninstalling](01a-upgrading-and-uninstalling.html).

### 1. The passwords

Asked in this order, each typed twice:

| | |
|---|---|
| **Account password** | the password every SD session asks for — at the keyboard, over ssh and through the API |
| **Administrator password** | unlocks the administrator commands, with `ADMIN` |
| **Global password** | **optional — leave it blank if no SD Core server manages this computer.** The SD Core for Linux server signs in with it, and it also unlocks the administrator commands |

**A computer is managed if and only if it has a global password.** Leave it
blank and no SD Core server manages the computer: it is a database for this
computer alone, and there is nothing else to choose. Enter one and the computer
is managed by an SD Core for Linux server — see [Managed computers](15-managed-mode.html).

**Whether there is a global password is fixed at installation.** Nothing
creates or clears one afterwards; the server can change an existing one with
`SET.PASSWORD GLOBAL`. To add one to a computer that has none, install again as
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
| **Let other computers reach it** | opens port 4249 in Windows Firewall. Without it the API answers this computer only |
| **Provide Solo's ssh server (port 4251)** | shown when an OpenSSH server is already installed. On by default. Unticked, Solo has no ssh server |
| **Let other computers reach it** | opens Solo's own firewall rule for port 4251 to the network. Without it, ssh answers this computer only |
| **Install the OpenSSH server** | shown when no ssh server is installed. Installs the MSI from `ssh-server\` and sets up Solo's own ssh server. Off by default |
| **Let other computers reach it** | the same, for the box above |

**When Solo's ssh server is on,** it runs on port 4251 and starts with Windows.
You sign in to it with your Windows account name and password, and it starts
`sd-solo`. Windows' own ssh server and its port 22 are not changed. See
[ssh access](08-ssh-access.html).

**An SD Core server that manages the computer connects from elsewhere**, through
the API and ssh. With either off, or open to this computer only, the server
cannot reach the computer until the choice is changed. An administrator who
sets up many computers gives these two answers in the control file.

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
two tasks was made. This was measured on one Windows 11 computer with a local
standard account; a domain account has not been tried.

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

**Uninstalling asks to keep or delete your data and configuration.** Keep leaves
the account and `sd.conf` and removes the passwords, audit trail, deny list,
`GLOBAL.BP.OUT`, the API's TLS key, your ssh key file, the system files and the
logs; a new installation then offers the account and `sd.conf` back. See
[Upgrading and uninstalling](01a-upgrading-and-uninstalling.html).

**The folder can be moved.** SD finds its files from where its programs are,
not from a path written into it, so a copied tree works under another Windows
user or on another computer — as long as the account password is known. The
startup tasks, the firewall rules and PATH are the installer's
and do not move with it.

## Installing many computers: the control file

**A file named `sd-solo-setup.conf` beside the installer answers its
questions** — it is how one USB stick sets up several computers. It answers
only what it gives; its presence does not make a computer managed.

| | |
|---|---|
| `admin-password=` | the administrator password |
| `global-password=` | the global password. **Blank or left out means no global password:** the computer is not managed and nothing is asked |
| `api=` | `off`, `local` (this computer only) or `open` (other computers too) |
| `ssh=` | `off` (no ssh server of Solo's, and the OpenSSH package is not installed), `local` or `open`. `local` and `open` install the package from `ssh-server\` when no OpenSSH server is on the computer |
| `reload-data=` | `yes` or `no`, where the folder holds data kept by an earlier uninstall: reload it, or start clean (the kept data is moved aside, never deleted). Blank: the installer asks, or reloads when the file answers everything else too. Where nothing is kept it does nothing, so one file can say it for every computer |
| `deny-verbs=` | a comma-separated list of commands the user of the computer may not run without the administrator or global password. See [Managed computers](15-managed-mode.html). Only a global-password session changes the list afterwards, so on a computer with no global password it stays as given until a new installation |

**The account password is deliberately not in it.** With a global password, the
user sets the account password **the first time they run `sd-solo` at that
computer's keyboard**. Until then ssh and the API accept only the global
password — the server can reach the computer, and nobody else can. **With no
global password the installer asks for the account password**, so an
installation with no global password cannot run unattended.

**A blank answer is asked for**, except the global password above, and so is
one the installer cannot accept: a password that breaks the rules, or an `api=`
or `ssh=` that is not `off`, `local` or `open`. When the file gives no global
password the installer says that this computer will not be managed by an SD
Core server, in `install-summary.log` and on its last page — a mistyped name
ends the same way, so read it. The release carries `sd-solo-setup.conf.sample`,
which explains each item and shows a sample answer; copy it to
`sd-solo-setup.conf` and fill it in.

**The file holds passwords in clear text.** Keep the stick safe, and do not
leave the file on a computer after installing.

**It is read only for a new installation**, kept data included (kept data holds
no passwords). On a computer that already has an `SDCoreSolo` data tree — an
earlier release's, with `sdsys` — it is ignored: the tree already has its
passwords.

## When a step fails

**The installer finishes and then lists what did not complete**:

```
These steps did not complete:
  ...
Details: %USERPROFILE%\SDCoreSolo\install-summary.log
```

The three steps are **Python**, **the account and passwords**, and **the
startup task, firewall and ssh**. The summary log holds each step's full
report, and ends each with a verdict.

## Changing any of it afterwards

| | |
|---|---|
| Whether there is a global password | uninstall and choose **Keep**, then install again: the passwords are asked again (the global one may be left blank, or entered), and you reload your saved data. See [Upgrading and uninstalling](01a-upgrading-and-uninstalling.html) |
| The API or ssh choices | the same — uninstall (Keep), install again and answer the API and ssh questions. Or change the firewall and `sd.conf` by hand |
| The passwords | `SET.PASSWORD`, `SET.PASSWORD ADMIN`, and on a managed computer `SET.PASSWORD GLOBAL` from the server. See [The account and its passwords](05-account-types.html) |
| Your PATH | `APPEND.SD.PATH`. See [Administrator commands](06-administrator-commands.html) |

**Installing over a working installation is an upgrade**, and asks nothing.
See [Upgrading and uninstalling](01a-upgrading-and-uninstalling.html).

## Continued in

[Upgrading and uninstalling](01a-upgrading-and-uninstalling.html) —
installing a new release over an existing one, and taking SD off the computer.
