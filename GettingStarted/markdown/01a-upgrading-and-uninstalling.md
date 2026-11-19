Title: Upgrading and uninstalling
Subtitle: Replacing an existing installation, and taking SD off the machine.

This page continues [Installing](01-installation.html).

## Upgrading

**Run the new installer while the old release is installed.** It recognizes
the installation and upgrades it. It asks no passwords and no API or ssh
choices — and it keeps them all. It never adds, changes or removes the global
password. **It does not show the *Shell and OS access* page**: an upgrade keeps
the shell (`sh`) and OS access (`os`) switches the installation has, so it
neither shuts what the administrator opened nor opens what was shut (see
[Installation](01-installation.html)). The administrator changes them later
with `admin`, then `set.os.access`.

**SD is stopped first**, because an upgrade replaces `sd-solo.exe`. It starts again
at the next Windows start-up, or with `sd-solo -start` (see
[Running SD](03-running-sd.html)).

**What an upgrade replaces, and what it keeps:**

| | |
|---|---|
| **replaced** | the programs, the system programs and their catalog, SD's messages, the VOC templates and the other files the release ships |
| **kept** | your account and its data, the passwords, `sd.conf`, the API's TLS key, and on a managed computer the server's programs in `global.bp.out` and the list of denied commands |

**Then it brings your account up to the release.** Replacing files is not
enough on its own: your account's VOC and SD's dictionaries were built by the
release that installed them. So an upgrade also, for you:

- adds the new release's commands to your account's VOC (`update.accounts all`),
  and takes out the records on SD's deletion list - the ones a release stopped
  shipping - but only a record that is still exactly what SD shipped, after
  saving a copy (`restore.voc` puts it back); a record you keep your own
  version of, marked `[locked]`, is left alone;
- takes the `sd.conf` parameters SD no longer has out of your kept `sd.conf` (SD
  would refuse to start on one), and says which it removed;
- converts your account's files to case-insensitive record ids (below);
- merges and recompiles SD's own dictionaries;
- catalogs the server's programs in `global.bp.out` again, on a managed
  computer, because the global catalog is one of the files replaced.

**Your files are converted to case-insensitive record ids for you.** Every file
`create.file` makes treats ids as one whatever the case (see
[Lower case](11-lower-case.html)), and an upgrade keeps your existing files, which
were built case sensitive. After the account's VOC is brought up to the release, the
upgrade reads each file the account's VOC names, **and converts it only if no two of
its ids differ only by case**, so a conversion cannot lose a record. A file is **left
exactly as it was** and named in `nocase-upgrade.log` in the Solo folder when:

- it holds two ids that differ only by case (rename or delete one of each pair, then
  run `configure.file` *name* `no.case` on it);
- it has alternate key indexes (convert it by hand with `configure.file`);
- it lies under the SD system folder, which the upgrade makes new itself
  (convert one yourself with `configure.file` *name* `no.case` if you need to);
- it could not be read or rebuilt.

A walk that did not finish is reported as a failure; what was not converted keeps
working, and ids are still found typed in any case. When some files were left, the
install report says **WARNING** and names the log.

Each step reports in `%USERPROFILE%\SDCoreSolo\install-summary.log`, ending
with a verdict. The startup task is registered again. So is the ssh task
(**SD Core Solo SSH**, which starts Solo's own ssh server on port 4251), but
only where Solo's ssh server is already set up: an upgrade never turns ssh on
for a computer that chose none. An existing firewall rule is not touched. **Solo's ssh server
keeps its reach**: one that was open (its rule open to other computers) listens on every IPv4
address, `0.0.0.0`, and `::1`; any other listens on `127.0.0.1` and `::1` only from this release on (an earlier
release listened on every address and left the firewall rule to keep other computers out).

**Upgrading from a release before WS1.1-3 changes how ssh works**, and does it
without asking: what the earlier release added to Windows' ssh settings is
removed, the key the SD Core server added to `%USERPROFILE%\.ssh\authorized_keys`
moves into Solo's own key file, and ssh to Solo is now on port 4251 instead of
22. See [ssh access](08-ssh-access.html). **An upgrade does not open the
firewall rule for 4251 to other computers**, not even on a managed computer
whose server has to reach it: its port 4251 answers this computer only until
you type `remote.ssh open` in SD, after `admin`, which opens both the listen address and the rule.

**Python is installed if none is there**, as on a new installation, and PATH
gets SD's program folder if it lost it.

**Limits.** An upgrade replaces every file it lists, so a system file you edited
by hand is replaced; edit `sd.conf` (kept) or your account (kept), not SD's own
files.

**What changed since the previous release**, and what an upgrade might break,
is in the *Changes* document that comes with the release: every addition,
change and removal, listed by section.

## Uninstalling

**Uninstall from Windows Settings, *Apps*, *SD Core Solo for Windows*.** It
first asks what to do with your data and configuration, and Keep comes first:

| | |
|---|---|
| **Keep** | leaves in `%USERPROFILE%\SDCoreSolo` your account `sduser` with its data (`user_accounts\sduser`), `sd.conf`, and what SD Core keeps with its accounts: the passwords (`sdsys\$cred`), the AI helper's settings and keys (`sdsys\ai`; see [The AI helper](24-the-ai-helper.html)), the audit trail and its older copies, the API's TLS key (`sd-tls`), your ssh key file (`ssh`), the SDNet logins, your own VOC changes (`newvoc.mods`) and saved VOC records, your scheduled jobs' definitions (`sdsys\$jobs`; see [Scheduled jobs](04-scheduled-jobs.html)), and where sessions start (`start.default`); a small file, `.sdcore-kept`, says so. Solo's ssh server keeps its host key in its administrators-only folder, `C:\ProgramData\SDCoreSolo\ssh`. **These are removed, and the dialog lists them:** the system files, the list of denied commands, `global.bp.out` and the install logs |
| **Delete** | removes the whole folder, for good, and the ssh host key with it |

**A silent uninstall never deletes your data**; it keeps, as above.

It then stops SD and — after the one administrator consent prompt — removes:

- first, every scheduled job's task (the folder **SD Core Solo Jobs**), so that
  no job starts while SD is being removed; their definitions stay with your data
  when you chose Keep;
- the startup task **SD Core Solo** and the ssh task **SD Core Solo SSH**,
  which also stops Solo's ssh server and deletes its administrators-only folder,
  `C:\ProgramData\SDCoreSolo` (all but the host key, when you chose Keep);
- the firewall rules for the API and for Solo's ssh port, 4251;

and takes `%USERPROFILE%\SDCoreSolo\usr\bin` off your PATH. Last, it keeps or
deletes your data as you chose, and says what it did and where.

**Install again later and the installer finds what Keep left.** It asks *Reload
your saved data and configuration into this new install?* (Yes is the default):

| | |
|---|---|
| **Yes** | copies the account's files into the new account and puts back everything else Keep kept — the passwords, the audit trail (the kept lines first, then the new installation's), the TLS key, your ssh key file, the SDNet logins, your VOC changes, your scheduled jobs (each given its task again by the installer's administrator step; the last page says *Scheduled jobs:* how many were, and how many were NOT) and where sessions start — then tries the saved `sd.conf` and brings the account's commands up to date, as an upgrade does. If SD will not start on the saved `sd.conf` — a line this release no longer knows, `STARTUP=` for one — the default is kept, the installer's summary says so, and your saved copy is left untouched |
| **No** | starts clean |

**For an installation nobody sits at**, the control file's `reload-data=yes` or `no`
answers the question; left blank, the installer asks, or reloads when the file
answers everything else too. Where nothing was kept it does nothing, so one control
file can serve many computers (see [Installing many
computers](01c-installing-many-computers.html)).

**Either way the old folder is moved aside, never deleted:** it becomes
`%USERPROFILE%\SDCoreSolo.kept-<date and time>`, and stays until you delete it.

**A reload keeps the passwords and asks none** (the summary says *passwords:
kept from the previous install - not changed*), and does not ask where sessions
start. Whether the computer is managed is whatever the kept passwords say. **A
control file that gives passwords replaces the kept ones**: that is how to add a
global password to a computer that has none, or to replace a forgotten one. Kept
data from an earlier release has no passwords, so they are asked as for a new
installation. See [Installing](01-installation.html).

**A folder left by an uninstaller older than SOLO 37's** still holds the system
files and the passwords, and is reinstalled over as it always was: nothing is
asked and everything is kept.

**What uninstalling leaves installed:** the OpenSSH server and Python, even if
the SD installer put them there — other programs may use them — and Windows'
own ssh firewall rule for port 22, which is Microsoft's and was never Solo's.
Remove them from *Apps* if you no longer want them. Keep keeps your optional ssh
key file and the server's host key, so a client that trusted the host key still
does after a reinstall; Delete removes both, and a client then sees a new one.

**If the administrator step cannot run**, the uninstaller says *"The startup
task, firewall rule or ssh setting could not be removed"* and names the log.
The programs are still removed. Delete the tasks **SD Core Solo** and **SD Core
Solo SSH** in Task Scheduler, end the `sshd.exe` that is listening on port 4251,
delete the folder `C:\ProgramData\SDCoreSolo`, and delete the rules
`SD-Solo-API-In-TCP` and `SD-Solo-SSH-In-TCP` in `wf.msc`, by hand, from an
elevated prompt.

## Moving the account to another computer

Moving the account is a backup on one computer, a copy of one file, and a restore on the other.
There is no verb that does it in one go; the two verbs are described under
[Administrator commands](06-administrator-commands.html) and in
[Backing up and restoring the account](06c-backup-and-restore.html).

1. On the old computer, after `admin`: `backup.account`. It writes one zip file to the saved
   backup directory and says its name.
2. Copy that file to the new computer (a USB stick, a network share, anything that copies a file).
3. On the new computer, install SD Core Solo for Windows, sign in, type `admin`, and run
   `restore.account` with the file's full path. It says what will be replaced and asks first.
4. **The restore happens the next time SD starts** (`sd-solo -restart`, or stop SD and start
   it), because it replaces the account you are using.

**What does not come across:** a password (the new computer keeps the passwords it was installed
with), SD's own system files and settings (`sd.conf`, the global catalog, the ssh and API choices),
and links in the account. **The backup is an ordinary, unencrypted zip** of every file in
the account: keep it where only you can read it, and delete it when the move is done. A Solo backup
restores only to Solo (on Linux or on Windows), never onto the multi-user SD Core.

## Continued in

[Differences from SD Core for Windows W2.0-0](01b-differences-from-sd-core-w2-0-0.html)
— what Solo leaves out, adds, and does differently.
