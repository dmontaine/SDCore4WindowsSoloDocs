Title: Differences from multiuser SD Core for Windows W1.1-0
Subtitle: What SD Core Solo for Windows leaves out, adds and does differently.

**SD Core Solo for Windows was made from the multiuser SD Core for Windows
W1.1-0**, and the language, the query processor, the file system and nearly
every command are the same. This page is for a reader who knows the multiuser
product. The User set applies to both.

## One user, one account

| multiuser W1.1-0 | Solo |
|---|---|
| many accounts, one per person, made by an administrator | **one account, `sduser`**, made by the installer. `WHO` and `@LOGNAME` say `sduser` on every computer, whatever the Windows user is called |
| SDSYS, entered by signing in to Windows as the `sdsys` user | **SDSYS is never entered.** Nobody logs in or `LOGTO`s to it; the administrator commands run from your own account |
| `CREATE.ACCOUNT`, `DELETE.ACCOUNT`, `MODIFY.ACCOUNT`, `GRANT`, `REVOKE`, `LIST.GRANTS`, `MODIFY.PASSWORD` | **gone** |
| Windows groups `sdusers`, `sdssh`, `sdapi`, `sdsshonly`; the `os.users` and `batch.jobs` permit lists; console and Remote Desktop denied to SD accounts | **gone.** There is one Windows user, and it is yours |

## Passwords

| multiuser W1.1-0 | Solo |
|---|---|
| a local sign-in asks for no password — Windows has authenticated you | **every session asks for the account password**: at the keyboard, over ssh, and through the API |
| a command on the command line (`sd LIST VOC`) needs an elevated window or a `batch.jobs` entry | it uses **a copy of the account password Windows keeps for you**, so scripts and scheduled jobs need no typing |
| administration is being SDSYS | **administration is `ADMIN`** and a password set at installation |
| `MODIFY.PASSWORD` | **`SET.PASSWORD`**, which also updates the kept copy |

See [The account and its passwords](05-account-types.html).

## Administration

| multiuser W1.1-0 | Solo |
|---|---|
| the administrator verbs are SDSYS's, and only SDSYS has them | the same verbs are in your account and **need `ADMIN` first** — including eight that had no check of their own because only SDSYS had them: `CONFIG`, `LISTU`, `LIST.LOCKS`, `LIST.READU`, `LOCK`, `CLEAR.LOCKS`, `SET.DATE`, `CLEAN.ACCOUNT` |
| editing the VOC directly is any account's own business | `ED VOC`, a program's `WRITE` or `DELETE` to the VOC, `COPY` into it, and saving or deleting a sentence with `.S` and `.D` **need `ADMIN`**. What SD writes to the VOC as a side effect — `CREATE.FILE`'s entry, the command stack — does not |
| an administrator can `CATALOG ... GLOBAL` | **nobody changes the global catalogue**, `ADMIN` or not. On a managed computer it holds the SD Core for Linux server's programs. See [Other hardening](13-hardening.html) |
| `ssh.server`, `remote.ssh`, `remote.api` | **gone.** The API and ssh are chosen when installing |
| `APPEND.SD.PATH` changes the system PATH | it changes **your** PATH, and needs no elevation |

See [Administrator commands](06-administrator-commands.html).

## Managed mode

**New in Solo.** A computer installed in managed mode is also managed by an
SD Core for Linux server, which signs in with a **global password** set when
the computer was installed. The server can put compiled programs into the
global catalogue (`GLOBAL.BP.OUT`, `SYNC.GLOBAL.CATALOG`) and keep a list of
commands the user may not run (`DENY.VERBS`). An installer control file,
`sd-solo-setup.conf`, sets up many computers the same way. See
[Managed mode](15-managed-mode.html).

## Installing and running

| multiuser W1.1-0 | Solo |
|---|---|
| installed for the computer: `C:\Program Files\SD` and `C:\ProgramData\SD`, by an administrator | installed for one user, **all in `%USERPROFILE%\SDCoreSolo`**, by that user, with one administrator consent prompt |
| a Windows service runs SD as LocalSystem | a **scheduled task** starts SD at Windows start-up **as you**, on an ordinary unelevated token — even when your Windows account is an administrator |
| the system programs' BASIC source is installed | **compiled programs only**; no system source is installed |
| OpenSSH installed from Windows Update, Python separately, the editors by winget | the release **carries** the OpenSSH MSI and the Python installer, and installs them from beside itself — offline, from a USB stick if need be. The two editors are in the release |
| each SD account's own Windows user lands in `sd` over ssh; administrators get a Windows shell | **your own ssh sign-in lands in `sd`**, at the account-password prompt. There is no Windows shell over ssh for you, and `scp`/`sftp` to your user do not work |

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
- **Scripts that `LOGTO SDSYS`**, or that expect administrator verbs to work
  without `ADMIN`.
- **A client that signs in with a Windows user name**, or with the old
  cleartext login, or through `SDConnectLocal`.
- **Anything that writes the global catalogue.** Catalogue programs locally
  (`CATALOG ... LOCAL`) instead.
- **Anything that expects `C:\Program Files\SD` or `C:\ProgramData\SD`.** Use
  `%USERPROFILE%\SDCoreSolo`.
