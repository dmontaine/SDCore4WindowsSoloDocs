Title: Differences from SD Core for Windows W2.0-0
Subtitle: What SD Core Solo for Windows leaves out, adds and does differently.

**SD Core Solo for Windows WS2.0-0 is the single-user edition of SD Core for
Windows W2.0-0**, and the language, the query processor, the file system, the
editors, the full-screen pages and nearly every command are the same. This page
is for a reader who knows the multiuser product. The User set applies to both.

## One user, one account

| SD Core for Windows W2.0-0 | Solo |
|---|---|
| many accounts, one per person, made by an administrator | **one account, `sduser`**, made by the installer. `who` and `@LOGNAME` say `sduser` on every computer, whatever the Windows user is called |
| SDSYS, entered by signing in to Windows as the `sdsys` user | **SDSYS is never entered.** Nobody logs in to it; the administrator commands run from your own account |
| `create.account`, `delete.account`, `modify.account`, `grant`, `revoke`, `list.grants`, `modify.password`, `logto` | **gone.** Solo has one account, so `logto` had nowhere to go (removed in WS1.1-3; typing it answers `logto is not in your VOC`) |
| Windows groups `sdusers`, `sdssh`, `sdapi`, `sdsshonly`; the `os.users` and `batch.jobs` permit lists; console and Remote Desktop denied to SD accounts | **gone.** There is one Windows user, and it is yours. The shell and OS access are the exception to "yours to use": two switches, `sh` and `os`, off as delivered, turned on by the administrator with `set.os.access` (see [Operating system access](06b-operating-system-access.html)) |

## Passwords

| SD Core for Windows W2.0-0 | Solo |
|---|---|
| a local sign-in asks for no password — Windows has authenticated you | **every session asks for the account password**: at the keyboard, over ssh, and through the API |
| a command on the command line (`sd-solo LIST VOC`) needs an elevated window or a `batch.jobs` entry | it uses **a copy of the account password Windows keeps for you**, so scripts and scheduled jobs need no typing |
| administration is being SDSYS | **administration is `admin`** and a password set at installation |
| `modify.password`, run by SDSYS | **`set.password`**: your own account password with no `admin` (it asks the current one), `set.password admin` after `admin`, `set.password global` by the SD Core for Linux server only. It also updates the kept copy |

See [The account and its passwords](05-account-types.html).

## Administration

| SD Core for Windows W2.0-0 | Solo |
|---|---|
| the administrator verbs are SDSYS's, and only SDSYS has them | the same verbs are in your account and **need `admin` first** — including eight that had no check of their own because only SDSYS had them: `config`, `listu`, `list.locks`, `list.readu`, `lock`, `clear.locks`, `set.date`, `clean.account` |
| editing the VOC directly is any account's own business | `ed voc`, a program's `write` or `delete` to the VOC, `copy` into it, and saving or deleting a sentence with `.s` and `.d` **need `admin`**. What SD writes to the VOC as a side effect — `create.file`'s entry, the command stack — does not |
| an administrator can `catalog ... global` | **nobody changes the global catalog**, `admin` or not. On a managed computer it holds the SD Core for Linux server's programs. See [Other hardening](13-hardening.html) |
| `ssh.server`, `remote.ssh`, `remote.api` | **`ssh.server` is gone.** The API and ssh are chosen when installing, and `remote.ssh` and `remote.api`, each `off`, `local` or `open` (after `admin`), change them afterwards. See [ssh access](08-ssh-access.html) and [API access](09-api-access.html) |
| `append.sd.path` changes the system PATH | it changes **your** PATH, and needs no elevation |

See [Administrator commands](06-administrator-commands.html).

## Managed computers

**New in Solo.** A computer installed with a **global password** — the
installer asks for one and it may be left blank — is also managed by an
SD Core for Linux server, which signs in with that password. The server can put
compiled programs into the global catalog (`global.bp.out`,
`sync.global.catalog`) and keep a list of commands the user may not run
(`deny.verbs`). An installer control file, `sd-solo-setup.conf`, sets up many
computers the same way. See [Managed mode](15-managed-mode.html).

## Installing and running

| SD Core for Windows W2.0-0 | Solo |
|---|---|
| installed for the computer: `C:\Program Files\SD` and `C:\ProgramData\SD`, by an administrator | installed for one user, **all in `%USERPROFILE%\SDCoreSolo`**, by that user, with one administrator consent prompt |
| a Windows service runs SD as LocalSystem | a **scheduled task** starts SD at Windows start-up **as you**, on an ordinary unelevated token — even when your Windows account is an administrator |
| the system programs' BASIC source is installed | **compiled programs only**; no system source is installed |
| OpenSSH installed from Windows Update, Python separately, the editors by winget | the release **carries** the OpenSSH MSI and the Python installer, and installs them from beside itself — offline, from a USB stick if need be. The two editors are in the release |
| each SD account's own Windows user lands in `sd-solo` over ssh, on the computer's ssh port 22; administrators get a Windows shell | **your own Windows account name and password sign-in on port 4251 lands in `sd-solo`**, at the account-password prompt, on an ssh server of Solo's own (run as SYSTEM, as the system sshd is; no key to set up). There is no Windows shell over ssh for you, and `scp`/`sftp` to your user do not work. The two products can be installed on one computer |

See [Installing](01-installation.html) and [Running SD](03-running-sd.html).

## Smaller differences

- **The API user name is always `sduser`.**
- **`SDConnectLocal` is disabled.** A client connects over the network API,
  even to this computer.
- **A session reading from a pipe ends at end of input**, with or without
  `OFF`, instead of waiting at the prompt.
- **The API's TLS relay runs with a restricted copy of your Windows token**,
  so it cannot read your files.

## What might stop working

- **Anything that creates, grants or deletes accounts**, or signs in to more
  than one account.
- **Scripts that use `logto`** (to `SDSYS` or anywhere else), or that expect
  administrator verbs to work without `admin`.
- **A client that signs in with a Windows user name**, or with the old
  cleartext login, or through `SDConnectLocal`.
- **Anything that writes the global catalog.** Catalog programs locally
  (`catalog ... local`) instead.
- **Anything that expects `C:\Program Files\SD` or `C:\ProgramData\SD`.** Use
  `%USERPROFILE%\SDCoreSolo`.
