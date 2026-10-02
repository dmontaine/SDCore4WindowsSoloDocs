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
with a verdict. The startup task is registered again; the firewall and ssh
settings are not touched.

**Python is installed if none is there**, as on a new installation, and PATH
gets SD's program folder if it lost it.

## Uninstalling

**Uninstall from Windows Settings, *Apps*, *SD Core Solo for Windows*.** It
stops SD, then — after the one administrator consent prompt — removes:

- the startup task **SD Core Solo**;
- the API's firewall rule;
- the ssh setting that starts `sd-solo` for your ssh sign-in;

and takes `%USERPROFILE%\SDCoreSolo\usr\bin` off your PATH.

**Your data stays.** `sdsys`, your account (`user_accounts\sduser`),
`sd.conf` and the API's TLS key are left in `%USERPROFILE%\SDCoreSolo`. Install
again later and the installer finds them: it keeps the mode and the passwords,
and asks only the API and ssh questions again.

**To remove the data too, delete `%USERPROFILE%\SDCoreSolo` yourself** after
uninstalling. Nothing else holds a copy.

**What uninstalling leaves installed:** the OpenSSH server and Python, even if
the SD installer put them there — other programs may use them — and the ssh
server's firewall rule. Remove them from *Apps* if you no longer want them.

**If the administrator step cannot run**, the uninstaller says *"The startup
task, firewall rule or ssh setting could not be removed"* and names the log.
The programs are still removed. Delete the task **SD Core Solo** in Task
Scheduler, and the `SD Core Solo` block in
`C:\ProgramData\ssh\sshd_config`, by hand.

## Continued in

[Differences from multiuser SD Core for Windows W1.1-1](01b-differences-from-multiuser-w1-1-1.html)
— what Solo leaves out, adds, and does differently.
