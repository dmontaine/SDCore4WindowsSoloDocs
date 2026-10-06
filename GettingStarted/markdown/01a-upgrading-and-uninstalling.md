Title: Upgrading and uninstalling
Subtitle: Installing a new release over an existing one, and taking SD off the computer.

This page continues [Installing](01-installation.html).

## Upgrading

**Run the new installer while the old release is installed.** It recognises
the installation and upgrades it. It asks nothing — no passwords, no API or ssh
choices — and it keeps them all. It never adds, changes or removes the global
password.

**SD is stopped first**, because an upgrade replaces `sd-solo.exe`. It starts again
at the next Windows start-up, or with `sd-solo -start` (see
[Running SD](03-running-sd.html)).

**What an upgrade replaces, and what it keeps:**

| | |
|---|---|
| **replaced** | the programs, the system programs and their catalogue, SD's messages, the VOC templates and the other files the release ships |
| **kept** | your account and its data, the passwords, `sd.conf`, the API's TLS key, and on a managed computer the server's programs in `GLOBAL.BP.OUT` and the list of denied commands |

**Then it brings your account up to the release.** Replacing files is not
enough on its own: your account's VOC and SD's dictionaries were built by the
release that installed them. So an upgrade also, for you:

- adds the new release's commands to your account's VOC — it never takes
  anything away, and a record you keep your own version of is left alone;
- merges and recompiles SD's own dictionaries;
- catalogues the server's programs in `GLOBAL.BP.OUT` again, on a managed
  computer, because the global catalogue is one of the files replaced.

Each step reports in `%USERPROFILE%\SDCoreSolo\install-summary.log`, ending
with a verdict. The startup task is registered again. So is the ssh task
(**SD Core Solo SSH**, which starts Solo's own ssh server on port 4251), but
only where Solo's ssh server is already set up: an upgrade never turns ssh on
for a computer that chose none. An existing firewall rule is not touched.

**Upgrading from a release before WS1.1-3 changes how ssh works**, and does it
without asking: what the earlier release added to Windows' ssh settings is
removed, the key the SD Core server added to `%USERPROFILE%\.ssh\authorized_keys`
moves into Solo's own key file, and ssh to Solo is now on port 4251 instead of
22. See [ssh access](08-ssh-access.html). **An upgrade does not open the
firewall rule for 4251 to other computers**, not even on a managed computer
whose server has to reach it: its port 4251 answers this computer only until
you run `solo-ssh-firewall.ps1 -Open` from an elevated PowerShell.

**Python is installed if none is there**, as on a new installation, and PATH
gets SD's program folder if it lost it.

## Uninstalling

**Uninstall from Windows Settings, *Apps*, *SD Core Solo for Windows*.** It
stops SD, then — after the one administrator consent prompt — removes:

- the startup task **SD Core Solo** and the ssh task **SD Core Solo SSH**,
  which also stops Solo's ssh server and deletes its administrators-only folder,
  `C:\ProgramData\SDCoreSolo`;
- the firewall rules for the API and for Solo's ssh port, 4251;

and takes `%USERPROFILE%\SDCoreSolo\usr\bin` off your PATH.

**Then it asks what to do with your data and configuration, and Keep comes
first:**

| | |
|---|---|
| **Keep** | leaves your account `sduser` with its data (`user_accounts\sduser`) and `sd.conf` in `%USERPROFILE%\SDCoreSolo`, with a small file, `.sdcore-kept`, that says so. **Everything else in the folder is removed:** the passwords, the audit trail, the list of denied commands, `GLOBAL.BP.OUT`, the API's TLS key and your ssh key file |
| **Delete** | removes the whole folder, for good |

**A silent uninstall never deletes your data**; it keeps, as above. Whichever you
choose, the uninstaller says what it did and where.

**Install again later and the installer finds what Keep left.** It asks *Reload
your saved data and configuration into this new install?* (Yes is the default):

| | |
|---|---|
| **Yes** | copies the account's files into the new account, tries the saved `sd.conf` and brings the account's commands up to date, as an upgrade does. If SD will not start on the saved `sd.conf` — a line this release no longer knows, `STARTUP=` for one — the default is kept, the installer's summary says so, and your saved copy is left untouched |
| **No** | starts clean |

**Either way the old folder is moved aside, never deleted:** it becomes
`%USERPROFILE%\SDCoreSolo.kept-<date and time>`, and stays until you delete it.
**The passwords are asked again** — they are not in the kept data — and the
global password may be left blank, so this is also how to add a global password
to a computer that has none, or to drop one. See [Installing](01-installation.html).

**A folder left by an earlier release's uninstaller** still holds `sdsys` and
the passwords, and is reinstalled over as it always was: nothing is asked and
everything is kept.

**What uninstalling leaves installed:** the OpenSSH server and Python, even if
the SD installer put them there — other programs may use them — and Windows'
own ssh firewall rule for port 22, which is Microsoft's and was never Solo's.
Remove them from *Apps* if you no longer want them. Your optional ssh key file
goes with the rest of the folder (Keep removes it), and so does the server's host
key, so a client sees a new host key after a reinstall.

**If the administrator step cannot run**, the uninstaller says *"The startup
task, firewall rule or ssh setting could not be removed"* and names the log.
The programs are still removed. Delete the tasks **SD Core Solo** and **SD Core
Solo SSH** in Task Scheduler, end the `sshd.exe` that is listening on port 4251,
delete the folder `C:\ProgramData\SDCoreSolo`, and delete the rules
`SD-Solo-API-In-TCP` and `SD-Solo-SSH-In-TCP` in `wf.msc`, by hand, from an
elevated prompt.

## Continued in

[Differences from multiuser SD Core for Windows W1.1-1](01b-differences-from-multiuser-w1-1-1.html)
— what Solo leaves out, adds, and does differently.
