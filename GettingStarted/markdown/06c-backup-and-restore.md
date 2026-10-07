Title: Backing up and restoring the account
Subtitle: Writing your account to a zip file, putting it back, remembering the backup folder, and recording the system's settings.

This page continues [Administrator commands](06-administrator-commands.html). **All four verbs need `ADMIN` first.**

## The backup folder: `set.backup.directory`

```
set.backup.directory folder
set.backup.directory
```

With a folder, the verb makes it if it does not exist, proves SD can write in
it, and remembers it in `sd.conf` as `BACKUPDIR=`. With no folder it shows the one
that is remembered and changes nothing.

**The folder must be a full path**, such as `C:\Backups` or `P:\backups`. A path
that depends on the current folder is refused.

**You do not have to run it first.** If no folder is remembered, `backup.account`
and `restore.account` ask for one and remember the answer exactly as this verb
would. `restore.account` with `NO.QUERY` has nobody to ask and refuses instead.

**An older SD Core Solo will not start on an `sd.conf` that carries a `BACKUPDIR`
line.** SD stops at start-up on any key it does not know. Remove the line before
going back to an older release.

## Backing up: `backup.account`

```
backup.account {TO folder}
```

Writes the account to **one zip file** in the folder, named after the computer,
the account and the time, for example `SD-ace-sduser-20261001-144419.zip`.

**There is one account, so no name is needed.** With none, the command fills in the
name `sduser`. `backup.account sduser` and `backup.account ALL` are accepted too.

* **Without `TO`** the remembered folder is used. **With `TO`** the folder is used
  for that one backup only; nothing is remembered.
* The zip holds the account's files and what is needed to make the account
  again: its type, routes and description.
* **No password is in it.** The system's settings are not backed up; use
  `settings.report` for them.
* **The account is counted before it is packed** and the counts are written in the
  zip. If what was written differs from what was counted, the zip is deleted and
  the verb says by how much. A backup that reports success has been checked
  against the account.

```
:backup.account
sduser: 4 files, 78848 bytes, 6 directories
```

**There is no globally catalogued program in a Solo backup**, and the zip says
the product is `windows-solo`.

## Restoring: `restore.account`

```
restore.account zipfile {NO.QUERY}
restore.account LATEST {NO.QUERY}
```

**With no account name the command fills in `sduser`**, as `backup.account` does. A
name (`sduser`) and `ALL` are accepted too.

* **A bare file name** is looked for in the remembered folder. A name that
  includes a folder is used as given.
* **`LATEST` takes the place of the file name** and chooses the newest backup in
  the remembered folder that was made **on this computer** and **holds** the
  account. It finds the computer and the time from the file name, then opens each
  candidate and reads its record of the accounts it holds (nothing is unpacked), so
  a backup renamed by hand may not be found. It says which file it chose before it
  asks. If a newer backup made on this computer does not hold the account, or
  cannot be read, it says so and names both files. If no backup holds the account
  it says so and changes nothing. The zip is still checked against its own counts,
  below, before anything changes.
* **The zip is checked against its own counts before anything is changed.** The
  product (a Solo backup restores only into a Solo), the number of accounts, and
  the account's files, bytes and directories must match what was unpacked. Any
  difference stops the restore with nothing changed.
* **It says what will be replaced, and asks once.** The default answer is no.
  `NO.QUERY` skips the question.
* A VOC entry that holds the account's old path is rewritten to the new one. A
  pointer to somewhere outside the account is listed, not followed.

**The restore does not happen while you are in the account.** The account being
restored is your own, so the restored copy is put aside and SD says it is waiting.
**Nothing has changed until SD is restarted:**

```
sd-solo -restart
```

When SD starts it swaps the restored account in, before any session exists. The
account you had is **kept**, in a folder beside it named `.sdrestore.previous`, so
a restore can be undone by hand. What happened is written to `sdrestore.log` in
`%USERPROFILE%\SDCoreSolo`; it ends with `SOLO-RESTORE APPLIED` when the swap
worked. **The audit trail** gets a line when the restore has put the copy aside,
`RESTORE.ACCOUNT account=sduser archive=<backup file name>`, naming the file and
not its folder; the swap itself is in `sdrestore.log`, not in the audit trail.

**Restoring needs SD to be the only session.** Nobody else may be signed in while
a backup or restore runs, and new sign-ins are refused until it finishes.

## What the system looks like: `settings.report`

```
settings.report {folder}
```

Writes a text report of the system's settings for you to keep. **Nothing reads it
back**: it is a record for you, not something that can be restored.

## See also

[Administrator commands](06-administrator-commands.html) ·
[Configuration](16-configuration.html) for `BACKUPDIR` ·
[Features the developers could not test](19-features-the-developers-could-not-test.html).
