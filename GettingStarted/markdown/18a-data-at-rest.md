Title: Encrypting the data SD keeps on disk
Subtitle: Putting SD's folder on a BitLocker drive, what that protects, what it does not, and what else must be on the encrypted drive.

This page is about the bottom layer in the table on the
[encryption page](18-encryption.html): the disk. SD itself is unchanged by it.
The encryption is done by Windows, to the drive your SD folder lives on. The
encryption page explains the other two layers, which are the application's, and
the functions and key ring they use.

SD folds case, so a command may be typed in either case. Commands are shown
here in lower case.

## The short version

**Turn BitLocker on for the drive that holds your profile**, which on an
ordinary computer is the system drive, C:. Solo keeps almost everything in one
folder, `%USERPROFILE%\SDCoreSolo`, so the simple and complete answer is the one
Windows already offers: a stolen or discarded computer, or a copy of the disk,
then holds ciphertext, and nothing about SD has to change.

**It protects nothing once the drive is unlocked.** While Windows is running,
anyone who can sign in as you, or as an administrator of the computer, reads
the data as plain text, exactly as before. The same is true of anyone who
mounts the drive.

**Encrypt the system drive, not only a data drive.** Several things SD writes
are on the system drive whatever you choose for the account: your temporary
folder (where a large sort writes its work files), the page file and the
hibernation file (which can hold SD's memory). BitLocker on C: covers all of
them at once.

## What needs protecting

Almost everything SD keeps is under `%USERPROFILE%\SDCoreSolo`, a folder
inside your own profile, which Windows keeps from other ordinary users.

| Path | What is there |
|---|---|
| `%USERPROFILE%\SDCoreSolo\user_accounts\sduser` | the account, your data |
| `%USERPROFILE%\SDCoreSolo\sdsys\$cred` | the credential store: the verifiers for the passwords, and the kept copy of the account password, which Windows protects for your Windows user |
| `%USERPROFILE%\SDCoreSolo\sd.conf` | the configuration |
| `%USERPROFILE%\SDCoreSolo\sdsys\audit` and `errlog` | the audit trail and the error log |
| `%USERPROFILE%\SDCoreSolo\sd-tls` | the API's TLS key, when you chose an API |
| `%USERPROFILE%\SDCoreSolo\ssh` | your ssh key file, when ssh is on |
| `%USERPROFILE%\SDCoreSolo\sdsys\ai` | the AI settings and keys, when you use the helper |
| `%USERPROFILE%\SDCoreSolo\sdsys\netlogins` | stored SDNet logins |
| `%USERPROFILE%\SDCoreSolo\dev\shm` | the shared-memory file, made again each time SD starts |
| `C:\ProgramData\SDCoreSolo\ssh` | Solo's own ssh server: its configuration, its host key (private), and the guard's log. Present only when ssh was chosen; readable only by administrators |
| `C:\ProgramData\SDCoreSolo\doors` | the answer and log of the helper that makes machine changes for `remote.ssh` and the other doors, when the installer set it up |

**The kept copy is not clear text, but it is not a secret from you.** Windows
protects it for your Windows user, as the [Security](12-security.html) page says,
so a copy of the folder moved to another user or computer does not open it. Anyone
signed in as you can use it. The other files in the folder, the account's data
included, are ordinary files: a copy of the folder is a copy of the data. That
alone is a reason not to leave a laptop unencrypted.

**Backups go where `BACKUPDIR` points.** It is not set until you run
`set.backup.directory`, which saves the folder in `sd.conf`
([Backup and restore](06c-backup-and-restore.html)). `backup.account` writes
plain zip files there. A backup is protected only while that folder is on an
encrypted drive, and not at all once the file is copied away.

**The sort work area is not under `SDCoreSolo`.** A sort that does not fit in
memory writes its work files to `SORTWORK`, and when that is not set, to
`TEMPDIR`, and when that is not set, to the `TMP` folder of the process that
sorts, which for you is `%LOCALAPPDATA%\Temp`
([The parameters](16c-the-parameters.html)). Those files can hold the keys being
sorted, in plain text. On an encrypted system drive they are covered. If your
profile or the account is on another drive and only that drive is encrypted, put
the work files on it. Write the path in POSIX form, as `sd.conf` does:

```
SORTWORK=/cygdrive/d/SDData/sortwork
```

Create the folder first. A folder that does not exist when SD starts is
ignored, and the default applies without a message. `TEMPDIR` itself, for SD's
other temporary files, is not covered by this page. A change to either parameter
needs a restart ([Configuration](16-configuration.html)).

**The page file and the hibernation file.** SD's shared memory can be paged out
to the page file, and a hibernated computer writes its whole memory, SD's included,
to `hiberfil.sys`. Both are on the system drive. With BitLocker on that drive they
are covered; on a computer without it, switch hibernation off (`powercfg /hibernate off`
in an elevated prompt) and consider a page file that is cleared at shutdown.

## Doing it

1. Turn BitLocker on for the system drive (Settings, *Privacy & security*,
   *Device encryption*, or Control Panel, *BitLocker Drive Encryption*). With a
   TPM, the drive unlocks at start-up without a prompt; a start-up PIN as well
   (group policy *Require additional authentication at startup*) stops a stolen
   computer from unlocking itself.
2. **Keep the recovery key away from the computer.** Without it, a drive that
   will not unlock is lost, and SD's data with it.
3. If the folder or the backups are on another drive, turn BitLocker on for
   that drive as well and let it unlock automatically with the system drive.

SD needs nothing changed. It starts after Windows has unlocked the drives, whether
that is at Windows start-up (the task **SD Core Solo**) or, on a standard
account, when you sign in. A drive that is locked at that moment would give SD an
empty place to start: unlock it with the system drive.

Turn it on before the data exists where you can. Encrypting a drive that
already holds data does encrypt it, but copies of the old blocks that the file
system has freed may stay readable until they are overwritten.

## What SD's own encryption covers, and what it does not

BitLocker, the layer described here, protects the **drive**, and SD is not aware of
it. The two other layers on the [encryption page](18-encryption.html) protect
**fields**: `sdencrypt()` and the `XE` conversion keep a value encrypted in the
file itself, so that a copy of the file, taken from a running computer, still
holds ciphertext for that field. They do not hide the record id, the other
fields, which values are empty or how long a value is, and the key is the
application's to keep. Use BitLocker for the whole folder, and SD's own
encryption for the few fields that must stay unreadable to someone who can read
the files.

## What this does not cover

- **A running computer.** Once the drive is unlocked, everything is plain text to
  anyone signed in as you, and to an administrator.
- **Copies you make.** A record exported to a file, a report printed to a file,
  or a file copied off the computer is outside the encrypted drive once it leaves it.
- **Backups.** A backup folder protects nothing unless it is on an encrypted
  drive or the backups are encrypted another way.
- **The API and ssh.** Nothing here changes how data travels over a connection;
  that is the job of TLS ([API access](09-api-access.html)) and ssh
  ([ssh access](08-ssh-access.html)).
- **The prompt at start-up.** The BitLocker PIN or recovery prompt is Windows'.

## What to do, in order

1. Turn BitLocker on for the system drive, and keep the recovery key elsewhere.
2. Do the same for any other drive that holds `SDCoreSolo`, the backup folder or the sort work files.
3. If a drive cannot be encrypted, set `SORTWORK` to a folder on one that is, and
   switch hibernation off.
4. Decide where the backup folder lives.
5. Restart the computer and start SD before trusting the arrangement.
