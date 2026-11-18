Title: Upgrading and uninstalling
Subtitle: Installing a new release over an existing one, and taking SD off the computer.

This page continues [Installing](01-installation.html).

## Upgrading

**Run the new installer while the old release is installed.** It recognizes
the installation and upgrades it. It asks no passwords and no API or ssh
choices — and it keeps them all. It never adds, changes or removes the global
password. **It asks one question: *Shell and OS access*.** A Solo from before 2.0-0
had its shell and OS access always on; now they are off unless the administrator
turns them on, so an upgrade asks which you want to keep, with both boxes
unticked (see [Installation](01-installation.html)). Unticked, nothing is turned
on and nothing already turned on is turned off. A silent upgrade shows no page
and turns nothing on.

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
- merges and recompiles SD's own dictionaries;
- catalogs the server's programs in `global.bp.out` again, on a managed
  computer, because the global catalog is one of the files replaced.

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
| **Yes** | copies the account's files into the new account and puts back everything else Keep kept — the passwords, the audit trail (the kept lines first, then the new installation's), the TLS key, your ssh key file, the SDNet logins, your VOC changes, your scheduled jobs (each given its task again by the installer's administrator step) and where sessions start — then tries the saved `sd.conf` and brings the account's commands up to date, as an upgrade does. If SD will not start on the saved `sd.conf` — a line this release no longer knows, `STARTUP=` for one — the default is kept, the installer's summary says so, and your saved copy is left untouched |
| **No** | starts clean |

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

## Continued in

[Differences from SD Core for Windows W2.0-0](01b-differences-from-sd-core-w2-0-0.html)
— what Solo leaves out, adds, and does differently.
