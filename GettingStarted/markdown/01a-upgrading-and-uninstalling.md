Title: Upgrading and uninstalling
Subtitle: Installing a new release over an existing one, and taking SD off the computer.

This page continues [Installing](01-installation.html).

## Upgrading

**Run the new installer while the old release is installed.** It recognises
the installation and upgrades it. It asks nothing — no mode, no passwords, no
API or ssh choices — and it keeps them all.

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
with a verdict. The startup task is registered again, and so is the ssh task
(**SD Core Solo SSH**, which starts Solo's own ssh server on port 4251). An
existing firewall rule is not touched.

**Upgrading from a release before WS1.1-3 changes how ssh works**, and does it
without asking: what the earlier release added to Windows' ssh settings is
removed, the key the SD Core server added to `%USERPROFILE%\.ssh\authorized_keys`
moves into Solo's own key file, and ssh to Solo is now on port 4251 instead of
22. See [ssh access](08-ssh-access.html). **A managed computer gets the firewall
rule for 4251 opened to other computers**, because the server has to reach it.
**A standalone computer does not**: its port 4251 answers this computer only
until you run `solo-ssh-firewall.ps1 -Open` from an elevated PowerShell.

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

**Your data stays.** `sdsys`, your account (`user_accounts\sduser`),
`sd.conf` and the API's TLS key are left in `%USERPROFILE%\SDCoreSolo`. Install
again later and the installer finds them: it keeps the mode and the passwords,
and asks only the API and ssh questions again.

**To remove the data too, delete `%USERPROFILE%\SDCoreSolo` yourself** after
uninstalling. Nothing else holds a copy.

**What uninstalling leaves installed:** the OpenSSH server and Python, even if
the SD installer put them there — other programs may use them — and Windows'
own ssh firewall rule for port 22, which is Microsoft's and was never Solo's.
Remove them from *Apps* if you no longer want them. Your optional ssh key file
stays with your data, in `%USERPROFILE%\SDCoreSolo\ssh`; the server's host key
goes with its folder, so a client sees a new host key after a reinstall.

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
