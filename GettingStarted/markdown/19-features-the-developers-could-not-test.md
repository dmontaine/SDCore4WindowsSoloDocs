Title: Features the Developers Could Not Test
Subtitle: The parts of SD Core Solo for Windows that were built and reasoned about but never exercised, what is known about each, and what it would take to settle it.

Everything else in this documentation describes behaviour that was run and
watched. **This page is the exception, and it exists so that the exception is
visible in one place rather than scattered through the reference as
footnotes.**

Nothing here is known to be broken. Each entry is something that **compiles,
or exists, or follows from the source, and was never put under load or into
the condition that would prove it.** Treat them as the list of things to pilot
before you depend on them.

**Two kinds of entry.** The first three sections are about SD's runtime —
locks, sockets, real data — which Solo shares with the multiuser SD Core for
Windows; they were established there, and Solo has not repeated them. The rest
are about Solo itself: its installer, its password, and the way it is reached.

## How to read an entry

| | |
|---|---|
| **Known** | what was actually run and observed |
| **Not known** | the specific gap — usually narrower than the heading suggests |
| **To settle it** | what would have to be done |

## Locking and contention

### Semaphores under contention

**Known.** The semaphores are exercised on every record lock, and two sessions
competing for the same record ran through them at once without misbehaving.

**Not known.** **No semaphore has ever been observed blocking.** What is
unmeasured is the waiting path, not the code path — the difference matters
only under a load heavier than anything yet run.

**To settle it.** Enough concurrent sessions to make one wait, and a watch on
what it does while it waits.

### Contention between an API session and a local one

**Known.** Two *local* sessions compete correctly: record, update and file
locks are all reported against the right holder, a waiting read is released
when the holder lets go, and a task lock refuses a second taker.

**Not known.** The same contest with one side arriving through the API server
rather than at a terminal.

**To settle it.** An API client and a terminal session competing for one
record.

### Task locks taken twice by the same session

**Known.** From the source: taking a task lock you already hold succeeds, and
one `unlock` releases it however many times you locked it — the ownership test
is *"unowned or mine"*, and `unlock` clears the slot outright.

**Not known.** It was **read rather than run**. No program has taken the same
task lock twice and then released it once.

**To settle it.** Four lines of SD BASIC.

## Application data

### A real application's data

**Known.** SD creates, writes, reads and deletes files, records, indexes,
select lists and sequential files, and the system files it bootstraps with are
real ones.

**Not known.** **No production application's data has been loaded into this
port.** Nothing here has met a file of hundreds of thousands of records, a
deep dictionary, or a schema built over years by somebody else.

**To settle it.** Restore an existing account and run it.

## Sockets

### UDP and ICMP

**Known.** TCP works: listening, connecting, accepting, reading, writing, the
blocking and non-blocking modes, and the error codes.

**Not known.** The `0x00010000` and `0x00020000` flags are named in the
documentation because they are **in the compiler**, not because a datagram was
ever sent. No UDP or ICMP socket has been opened.

**To settle it.** A datagram to a listener and back.

## Installing and upgrading

### An upgrade over an existing installation

**Known.** A new installation has been run from scratch many times, each on a
computer with no `SDCoreSolo` folder.

**Not known.** **The upgrade path has not been exercised.** The installer's
tests always uninstall and then install afresh, so an installation over an
existing `SDCoreSolo` has not been run start to finish. What an upgrade
replaces and keeps is described on
[Upgrading and uninstalling](01a-upgrading-and-uninstalling.html) from the
installer's own rules, not from having watched it.

**To settle it.** Install one release, use it, then install the next over it
and check the account, the passwords and the data.

### Installing from a USB stick with no network

**Known.** The installer needs nothing from the internet: the OpenSSH server
and Python are carried beside it, and its own steps use no network.

**Not known.** **A quiet install from a stick, on a computer with its network
unplugged, has not been run**, and neither has each of the two packages'
installs offline. What the OpenSSH package leaves behind — its firewall rule,
the ssh server's start-up type, its configuration before the first start — is
read by the installer's ssh steps but was not measured.

**To settle it.** Install from a stick on a computer with the network
unplugged, and read `install-summary.log`.

### The installer is not signed

**Known.** The installer carries no code-signing certificate.

**Not known.** **What Windows does with it has not been measured.** A copy
downloaded from the internet, and every file unpacked from a downloaded zip by
Explorer, carries the *"downloaded from the internet"* mark, so SmartScreen may
warn before it runs — from a USB stick too.

**To settle it.** Download the zip and run the installer on a computer that has
not seen it. A code-signing certificate is the fix; the free alternative is to
tell the user to choose *More info, Run anyway*.

### A tree moved to another Windows user or computer

**Known.** The `SDCoreSolo` folder holds no path, so it can be copied, and a
damaged kept copy of the account password is refused.

**Not known.** **A tree actually moved to another Windows user, or to another
computer, has not been run.** The kept copy is protected for one Windows user,
so it will not open for another, and the damaged-copy test stands in for that
case rather than being it. The startup tasks, the firewall rules and the PATH
entry are the installer's and do not move.

**To settle it.** Copy a tree to another user and to another computer, and sign
in with the account password.

## The password and how SD is reached

### The password prompt at a real console

**Known.** A session asks for the account password and refuses a wrong one — on
input that is piped in, one try; the kept copy lets `sd-solo <command>` run.

**Not known.** Three things at a real terminal, by hand: **the three tries** at
`Password:` before the session ends; **a command line refused** when the kept
copy has been moved aside; and **a restart starting SD with nobody signed in**.

**To settle it.** Try each at the keyboard.

### ssh at a real terminal, and its password prompt

**Known.** An ssh session lands inside SD, and SD's terminal layer was watched
driving a real Windows console.

**Not known.** Those two at once, and the password prompt on top of them.
Nobody has run an interactive session at a terminal *reached over ssh* and
been asked for the account password there, where the pseudo-terminal belongs
to the ssh server rather than to the Windows console host. Screen handling,
cursor positioning and the editing keys all go through that layer.

**To settle it.** One interactive session from a second computer: the password
prompt, then a full-screen operation and the arrow keys.

### A domain user in the ssh setting

**Known.** Solo's ssh server is configured with `AllowUsers` and the lower-case
name of a local user.

**Not known.** **Whether the ssh server matches a *domain* user by
`name@domain`, the form the configuration uses for one.**

**To settle it.** On a domain-joined computer, sign in over ssh, on port 4251,
and check that you land in SD.

### Solo's own ssh server: starting at boot, and reaching it

**Known.** The first build of this version ran Solo's ssh server as an ordinary
user. With it: a scheduled task started the server at Windows start-up before
anyone signed in; a key sign-in worked; a second computer reached port 4251
through the firewall rule; SD Core Solo and SD Core were installed and running
together, each with its own ports; and **the server checked a Windows password
and then could not start the session** (Windows error 1314), which is why this
version runs the server as SYSTEM instead.

**Also known.** The SYSTEM server this version installs, on the developer's
computer: a Windows password sign-in from the same computer reached SD (the
owner typed his own password), and a key sign-in did too.

**Not known.** The SYSTEM server has not been signed in to **from a second
computer**, nor **after a restart with nobody signed in** (that was seen for the
first build only), and **ssh to port 22 still landing in SD Core**, with Solo
installed beside it, has not been tried.

**To settle it.** Restart the computer, do not sign in, and connect from a
second computer with `ssh -p 4251`, typing the Windows password and then the SD
password; and connect to port 22 as an SD Core user.

### The first password on a computer installed from a control file

**Known.** A computer installed from a control file has no account password
until the user sets one at the keyboard, and the server's global password is
accepted meanwhile.

**Not known.** The prompt's retries and refusals, the refusal a first-password
request gets over ssh, and what the API does before a first password has been
set.

**To settle it.** Install from a control file and try each.

## Scheduled tasks

### A task that does not store your password

**Known.** A scheduled task that runs as you, when you are signed in, runs
`sd-solo <command>` with the kept copy of the account password. That is the case the
design was built around.

**Not known.** **A task set to *Run whether user is logged on or not* with *Do
not store password* ticked.** Windows signs such a task in without your
password, and the kept copy is protected by it, so it may not open. The
question is whether a task that stores no password can open a copy protected
for your Windows user. See [Scheduled jobs](04-scheduled-jobs.html).

**To settle it.** Create such a task and see whether the command runs.

## SD BASIC statements that compile but were never run

| | why not |
|---|---|
| `sendmail` | needs a mail relay configured |
| `chgphant()` | needs a phantom process to change |
| `ccall()` | needs a C function registered into the executable |

**Known.** All three compile in an ordinary account.

**Not known.** What any of them does. Nothing else in the documentation
depends on them.

## Third-party editors

**The key bindings documented for Microsoft Edit are the editor's own, read
from its source rather than driven at a keyboard.** SD installs it and calls
it; it does not implement it. If a binding differs from what is written, the
editor is right and the page is wrong.

## What is NOT on this page, and why

**Anything that was tested and failed is a defect, not a gap**, and does not
belong here — it is either fixed or it is a known issue.

**Anything a reader might merely find surprising is not a gap either.** The
places where this port deliberately differs from OpenQM, ScarletDME or SD on
Linux are documented as differences, in the pages that describe the feature.
This page is only about what nobody has watched happen.

## See also

[Sessions and Locks](06a-sessions-and-locks.html) covers the locking model that
two of the entries above qualify.
[Scheduled jobs](04-scheduled-jobs.html) covers the Task Scheduler entry.
[System limits](16a-system-limits.html) states which of its figures come from
the source.
