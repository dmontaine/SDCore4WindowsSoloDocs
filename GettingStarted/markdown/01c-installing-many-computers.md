Title: Installing Many Computers
Subtitle: The control file, what the installer checks when it finishes, when a step fails, and changing the choices afterwards.

This page continues [Installing](01-installation.html).

## Installing many computers: the control file

**A file named `sd-solo-setup.conf` beside the installer answers its
questions** — it is how one USB stick sets up several computers. It answers
only what it gives; its presence does not make a computer managed.

| | |
|---|---|
| `admin-password=` | the administrator password |
| `global-password=` | the global password. **Blank or left out means no global password:** the computer is not managed and nothing is asked |
| `api=` | `off`, `local` (this computer only) or `open` (other computers too), port 4249. Blank: the installer asks, and a silent install takes `off`; a managed computer needs `open` |
| `ssh=` | `off` (no ssh server of Solo's, and the OpenSSH package is not installed), `local` or `open`, port 4251. `local` and `open` install the package from `ssh-server\` when no OpenSSH server is on the computer. Blank: the installer asks, and a silent install takes `local` (`off` where no OpenSSH server is on the computer); a managed computer needs `open`. `off` with `ssh-public-key-file=` is refused |
| `ssh-public-key-file=` | a public key file on this computer whose owner may ssh straight into `sd-solo`, on port 4251 (turns ssh on, `local`, when `ssh=` is blank); the key goes into Solo's own key file. Optional: ssh also takes the user's Windows account name and password. A key file the installer cannot read is refused |
| `reload-data=` | `yes` or `no`, where the folder holds data kept by an earlier uninstall: reload it, or start clean (the kept data is moved aside, never deleted). Blank: the installer asks, or reloads when the file answers everything else too. Where nothing is kept it does nothing, so one file can say it for every computer |
| `os-access=` | `none`, `sh`, `os` or `both`: which of the two switches to turn on (see *Shell and OS access* above). Blank: the installer asks. Used on a new installation only; an upgrade keeps the switches the installation has |
| `start=` | `menu` (the home screen) or `terminal` (the command prompt): where new sessions start (see *Where sessions start* above). Blank: the installer asks. Used on a new installation only; an upgrade keeps what the installation has |
| `viewer=` | `none`, or the name of a PDF reader to allow for `pdf` (see *PDF reader* above), such as `msedge`; it must pass `set.viewer`'s test. Blank: the installer asks when it finds a known reader. Used on a new installation only |
| `deny-verbs=` | a comma-separated list of commands the user of the computer may not run without the administrator or global password. See [Managed mode](15-managed-mode.html). Only a global-password session changes the list afterwards, so on a computer with no global password it stays as given until a new installation |

**The account password is deliberately not in it.** With a global password, the
user sets the account password **the first time they run `sd-solo` at that
computer's keyboard**. Until then ssh and the API accept only the global
password — the server can reach the computer, and nobody else can. **With no
global password the installer asks for the account password**, so an
installation with no global password cannot run unattended.

**A blank answer is asked for**, except the global password above, and so is
one the installer cannot accept: a password that breaks the rules, an `api=`
or `ssh=` that is not `off`, `local` or `open`, an `os-access=` that is not
`none`, `sh`, `os` or `both`, or a `start=` that is not `menu` or `terminal`. When the file gives no global
password the installer says that this computer will not be managed by an SD
Core server, in `install-summary.log` and on its last page — a mistyped name
ends the same way, so read it. The release carries `sd-solo-setup.conf.sample`,
which explains each item and shows a sample answer; copy it to
`sd-solo-setup.conf` and fill it in.

**The file holds passwords in clear text.** Keep the stick safe, and do not
leave the file on a computer after installing.

**It is read only for a new installation**, kept data included. **Passwords it
gives replace the ones kept data holds**; without them a reload keeps the kept
passwords. On a computer that already has an `SDCoreSolo` data tree — an
earlier release's, with its system files — it is ignored: the tree already has
its passwords.

## What the installer checks when it finishes

Every install, a new one, a reload and an upgrade, ends by confirming that
**`sd-solo -internal` is closed**: the door the installer itself uses is a
one-shot, opened only by a marker file the installer writes immediately before
each of its own steps, so with no marker it must be refused (*Connection
terminated*) and leave none behind. If it is admitted, the install fails and
`install-summary.log` says so.

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
| Whether there is a global password | uninstall and choose **Keep**, then install again with a control file that gives the passwords (`admin-password=`, and `global-password=` filled in or left blank), and reload your saved data: the passwords it gives replace the kept ones. See [Upgrading and uninstalling](01a-upgrading-and-uninstalling.html) |
| The API or ssh choices | the same — uninstall (Keep), install again and answer the API and ssh questions. Or change the firewall and `sd.conf` by hand |
| The passwords | `set.password`, `set.password admin`, and on a managed computer `set.password global` from the server. See [The account and its passwords](05-account-types.html) |
| Your PATH | `append.sd.path`. See [Administrator commands](06-administrator-commands.html) |

**Installing over a working installation is an upgrade**, and does not show the
*Shell and OS access* page: it keeps the switches the installation has. See [Upgrading and uninstalling](01a-upgrading-and-uninstalling.html).

## Continued in

[Upgrading and uninstalling](01a-upgrading-and-uninstalling.html) —
upgrading an existing installation, and uninstalling.
