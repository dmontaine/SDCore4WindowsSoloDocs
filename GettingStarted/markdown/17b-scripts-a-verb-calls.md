Title: The Scripts a Verb Calls
Subtitle: The scripts SD runs when a verb needs the Windows side - PATH, the editor, the ssh key, backups, the settings report and a restore.

This page continues [The Scripts SD Runs For
Itself](17a-scripts-sd-runs-itself.html).

## The ones a verb calls

**Prefer the verb.** It reports what happened in the product's own words and
refuses when it cannot act; the script does the work and assumes the caller
knew what they were doing.

| | Called by |
|---|---|
| `sd-path.ps1` | `append.sd.path on` \| `off` — puts SD's program folder on **your** PATH, or takes it off |
| `micro-home.ps1` | the editor programs, before `micro` starts — see below |
| `solo-sshkey.ps1` | the API's ssh key request (49), on a managed computer — see below |
| `sd-account-archive.ps1` | `BACKUP.ACCOUNT` and `RESTORE.ACCOUNT` — see below |
| `sd-settings-os.ps1` | `SETTINGS.REPORT` — see below |
| `sd-backupdir.ps1` | `SET.BACKUP.DIRECTORY` — see below |

**One more is run by SD itself, when it starts:** `solo-restore-swap.ps1`, also
below.

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
remove or list its ssh key** — see [Managed computers](15-managed-mode.html). It is
not for typing: it takes a request verb (`ADD`, `REMOVE` or `LIST`) and a key or
fingerprint, and prints lines SD reads back (`RESULT=ADDED`, `FPR=SHA256:...`,
`ERROR=...`). **It needs no elevation**, because it writes only Solo's own key
file, `%USERPROFILE%\SDCoreSolo\ssh\authorized_keys`, and it refuses unless
`sshd.exe` is installed. It runs `solo-sshd.ps1 -Prepare` first, which also
tells it Solo's ssh server key and port. It touches only lines it wrote itself,
tagged `sdcoresolo-managed`, and keeps at most four. Exit **0** with a `RESULT=` line,
**1** with an `ERROR=` line and nothing changed.

### `sd-account-archive.ps1`

**The file work behind `BACKUP.ACCOUNT` and `RESTORE.ACCOUNT`**: it writes the
backup zip, unpacks one into a staging folder, counts what is there, and puts a
restored account's files in place. It is not for typing. Whatever it reports
ends in one line, `ACC-ARCHIVE <mode> OK` or `ACC-ARCHIVE ERROR <reason>`, and
the verb reads only that. Exit **0** done, **1** refused or failed, **2** it
could not run.

**The zip is the same shape SD Core for Linux writes**: a `manifest.txt` at the
top and the account under `accounts/`, every folder stored even when empty, so a
backup made on one product can be read on the other. **It refuses a junction or
symbolic link inside the account** rather than follow it into somewhere else.
See [Backing up and restoring the account](06c-backup-and-restore.html).

### `sd-settings-os.ps1`

**The Windows sections of `SETTINGS.REPORT`**: `sd.conf`, ssh, the API's
certificate, the firewall rules and the start-up task. Every line is printed as
`REPORT <text>` and the last is `SETTINGS-OS OK SECTIONS <n>`. **It never
prints a password or a private key**; of the API's key-and-certificate file it
decodes the certificate only. A section it cannot read says so in its own lines
and the rest still print. Exit **0** done, **1** failed.

### `sd-backupdir.ps1`

**What `SET.BACKUP.DIRECTORY` runs.** It saves the folder `BACKUP.ACCOUNT` and
`RESTORE.ACCOUNT` use when none is typed, as one line of `sd.conf`,
`BACKUPDIR=<full path>`, which is read at every use, so a change needs no
restart. It makes the folder if it is not there and proves it can write to it
before it saves. **The path must be a full Windows path, plain ASCII, at most 240
characters.** Only that one line of `sd.conf` changes, and the file is put back
if it does not read back. Exit **0** done, **1** refused or failed. See
[Backing up and restoring the account](06c-backup-and-restore.html).

### `solo-restore-swap.ps1`

**Puts a restore in place while SD starts.** `RESTORE.ACCOUNT` cannot replace
`sduser` from inside a session, because every session is in it, so the verb
unpacks and checks the backup and leaves a marker, `.sdrestore.pending`, in the
`SDCoreSolo` folder. The next `sd-solo -start` or `-restart` finds it, runs this
script before SD's shared memory is made, and starts SD afterwards whatever the
script answered. **The account's old contents are kept in `.sdrestore.previous`**
until the next restore, so a bad one can be undone by hand. What it did is
appended to `sdrestore.log`. Exit **0** applied, **1** failed (put back, the
marker kept), **2** not run because an SD process of this installation is still
running (the marker kept), **3** nothing was pending.

## See also

[Installing](01-installation.html) covers what the installer puts on the
computer and what an upgrade replaces.
[Security](12-security.html) covers what each password guards.
