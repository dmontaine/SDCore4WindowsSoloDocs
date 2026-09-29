Title: Installing
Subtitle: What the installer asks, what it puts where, and the control file for installing many computers.

**The installer is one file, `sd-solo-setup-WS1.1-0.exe`, and it installs for
the Windows user who runs it.** Everything goes into that user's profile, in
`%USERPROFILE%\SDCoreSolo`. Nothing is installed for other users of the
computer, and there is no Windows service.

## Before you start

| | |
|---|---|
| Windows | Windows 10 or 11, 64-bit |
| Rights | an ordinary user account. The installer asks for administrator consent **once**, for the steps that change the computer rather than your profile |
| The package | the installer, with two folders beside it: `ssh-server\` holding Microsoft's OpenSSH server MSI, and `python\` holding python.org's Python installer (`python-3.x.x-amd64.exe`) |

**The two folders are part of the package, in every mode.** If either is
missing the installer stops at once with *"Invalid installation package.
Missing beside this installer:"* and the name of what is missing, before it
changes anything. Whether each package is actually *installed* is decided
below; carrying them is not optional.

**The package works offline and from read-only media.** It reads the two
packages from beside itself and never downloads anything, so the same folder
serves as a download and as a USB stick for installing several computers.

**The multiuser SD Core for Windows must not be installed.** If it is, the
installer stops with *"SD Core is installed on this computer. Uninstall it
first."* The two cannot share a computer.

## What you are asked

### 1. The mode

| | |
|---|---|
| **Standalone** | a database for this computer only. The default |
| **Managed client of an SD Core server** | a computer an SD Core for Linux server also manages. See [Managed mode](15-managed-mode.html) |

**The mode cannot be changed later except by a new installation.** It is the
one choice that is fixed: it decides whether a global password exists, and
nothing sets or clears that afterwards. The installer asks it only when there
is no `SDCoreSolo` data yet.

### 2. The passwords

Asked in this order, each typed twice:

| | |
|---|---|
| **Account password** | the password every SD session asks for — at the keyboard, over ssh and through the API |
| **Administrator password** | unlocks the administrator commands, with `ADMIN` |
| **Global password** | managed mode only. The SD Core for Linux server signs in with it, and it also unlocks the administrator commands |

**Every password needs at least 8 characters, with a lower-case letter, an
upper-case letter, a digit and a symbol** — letters, digits and punctuation
only, no spaces. The global password must differ from both of the others;
otherwise the server, signing in with the same name, would land in an ordinary
session. See [The account and its passwords](05-account-types.html).

### 3. The API and ssh — standalone only

| | |
|---|---|
| **Provide the SD Core API (port 4243)** | starts the API listener. Off by default |
| **Let other computers reach it** | opens port 4243 in Windows Firewall. Without it the API answers this computer only |
| **Let other computers reach this computer's ssh server** | shown when an ssh server with a firewall rule is already installed. Opens that rule to the network |
| **Install the OpenSSH server** | shown when no ssh server is installed. Installs the MSI from `ssh-server\` |

**In managed mode none of this is asked**: the API is on and reachable from
other computers, and so is ssh — the OpenSSH MSI is installed if there is no
ssh server. The server has to reach the computer from elsewhere, so managed
mode leaves nothing to choose.

### What is not asked, because it always happens

| | |
|---|---|
| **Python** | installed for you, from `python\`, unless a Python 3.13 or later is already installed. It is a per-user install and needs no administrator rights |
| **PATH** | `%USERPROFILE%\SDCoreSolo\usr\bin` is added to your PATH, so `sd` works from any new window |
| **ssh lands in SD** | wherever an OpenSSH server is present, it is set so that your ssh sign-in starts `sd`, and the ssh server is set to start with Windows. See [ssh access](08-ssh-access.html) |

## The one administrator step

**One consent prompt appears near the end.** It runs a single script,
`solo-machine.ps1`, which does the four things that are the computer's rather
than yours:

- registers the scheduled task **SD Core Solo**, which starts SD at every
  Windows start-up, as you, whether or not you are signed in;
- opens or restricts the firewall rules chosen above;
- installs the OpenSSH MSI, when that was chosen or managed mode needs it;
- writes the ssh setting that starts `sd` for your ssh sign-in.

**If you decline the prompt, SD is installed but does not start at start-up**,
and ssh and the firewall are left as they were. The installer says which steps
did not complete.

## What lands where

Everything is under `%USERPROFILE%\SDCoreSolo`:

| | |
|---|---|
| `usr\bin` | `sd.exe` and the other programs, the client libraries, and the two full-screen editors the `edit` and `micro` verbs run |
| `usr\clients` | the client libraries for applications, 64-bit and 32-bit. See [Client distribution](10-client-distribution.html) |
| `sd.conf` | the configuration. See [Configuration](16-configuration.html) |
| `sdsys` | SD's own files: the system programs (compiled only — no source is installed), messages, the dictionaries, the credential store |
| `user_accounts\sduser` | the one SD account, and your data |
| `sd-tls` | the API's TLS key, made the first time a client connects |
| `install-summary.log` | what every installer step did — read this first when something did not work |

**Uninstalling keeps `sdsys`, the account and `sd.conf`.** See
[Upgrading and uninstalling](01a-upgrading-and-uninstalling.html).

**The folder can be moved.** SD finds its files from where its programs are,
not from a path written into it, so a copied tree works under another Windows
user or on another computer — as long as the account password is known. The
startup task, the firewall rules, the ssh setting and PATH are the installer's
and do not move with it.

## Installing many computers: the control file

**A file named `sd-solo-setup.conf` beside the installer answers its
questions.** It is for **managed mode only** — its presence makes the install
managed — and it is how one USB stick sets up several computers.

| | |
|---|---|
| `admin-password=` | the administrator password |
| `global-password=` | the global password |
| `deny-verbs=` | a comma-separated list of commands the user of the computer may not run without the administrator or global password. See [Managed mode](15-managed-mode.html) |

**The account password is deliberately not in it.** On a computer installed
from a control file, the user sets the account password **the first time they
run `sd` at that computer's keyboard**. Until then ssh and the API accept only
the global password — the server can reach the computer, and nobody else can.

**A blank answer is asked for**, and so is one that breaks the password rules.
The release carries `sd-solo-setup.conf.sample`, which explains each item and
shows a sample answer; copy it to `sd-solo-setup.conf` and fill it in.

**The file holds passwords in clear text.** Keep the stick safe, and do not
leave the file on a computer after installing.

**It is read only for a new installation.** On a computer that already has an
`SDCoreSolo` data tree it is ignored — the tree already has its passwords.

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
| The mode | a new installation. The mode is read from the data, so an install over kept data keeps it: uninstall, **move `%USERPROFILE%\SDCoreSolo` aside** (it holds your data — keep it), then install |
| The API or ssh choices | uninstall, then install again — the data and passwords are kept, and the API and ssh questions are asked again. Or change the firewall and `sd.conf` by hand |
| The account password | `SET.PASSWORD`. See [The account and its passwords](05-account-types.html) |
| Your PATH | `APPEND.SD.PATH`. See [Administrator commands](06-administrator-commands.html) |

**Installing over a working installation is an upgrade**, and asks nothing.
See [Upgrading and uninstalling](01a-upgrading-and-uninstalling.html).

## Continued in

[Upgrading and uninstalling](01a-upgrading-and-uninstalling.html) —
installing a new release over an existing one, and taking SD off the computer.
