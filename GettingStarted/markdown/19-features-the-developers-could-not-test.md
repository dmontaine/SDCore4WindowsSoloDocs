Title: Known limits
Subtitle: Parts of SD Core Solo for Windows whose behavior depends on your environment, and what to check before an application relies on them.

Nothing on this page is known to be broken. Each entry is a place where what
happens depends on conditions SD cannot establish for itself: the load on your
computer, the network between two computers, the version of Windows, the program
you run beside SD. Treat the page as a list of things to try on your own system
before an application depends on them.

## How to read an entry

| | |
|---|---|
| **What to expect** | how the feature behaves, as far as SD controls it |
| **Check before relying on it** | what to try, on your own system |

## Installing and running

### An upgrade over an existing installation

**What to expect.** An installation over an existing `SDCoreSolo` replaces what
SD ships and keeps what is yours: the account, the passwords and the data. What
an upgrade replaces and keeps is described on
[Upgrading and uninstalling](01a-upgrading-and-uninstalling.html).

**Check before relying on it.** Install over a copy of your own installation
first: use it, install the next release over it, and check the account, the
passwords and the data.

### Installing from a USB stick with no network

**What to expect.** The installer needs nothing from the internet: the OpenSSH
server and Python are carried beside it, and its own steps use no network. What
the OpenSSH package leaves behind (its firewall rule, the ssh server's start-up
type, its configuration before the first start) depends on the Windows
installation it is added to.

**Check before relying on it.** Before an installation from a stick on many
computers, install on one with its network unplugged, and read
`install-summary.log`.

### The installer is not signed

**What to expect.** The installer carries no code-signing certificate. A copy
downloaded from the internet, and every file unpacked from a downloaded zip by
Explorer, carries the *"downloaded from the internet"* mark, so SmartScreen may
warn before it runs, from a USB stick too.

**Check before relying on it.** Download the zip and run the installer on a
computer that has not seen it. Choose *More info, Run anyway* if SmartScreen
warns.

### A tree moved to another Windows user or computer

**What to expect.** The `SDCoreSolo` folder holds no path, so it can be copied.
The kept copy of the account password is protected for one Windows user, so it
does not open for another, and a damaged or foreign kept copy is refused. The
startup tasks, the firewall rules and the PATH entry belong to the installer and
do not move.

**Check before relying on it.** Copy a tree to another user and to another
computer, and sign in with the account password.

### A first install from a control file, over ssh and the API

**What to expect.** A computer installed from a control file has no account
password until the user chooses one at the keyboard. Until then only the global
password is accepted, at the keyboard's first prompt, over ssh and over the API.

**Check before relying on it.** Install from a control file, then sign in over
the API and over ssh with the global password before choosing the account
password. Try the prompt's retries and refusals at the keyboard.

## The password and how SD is reached

### The password prompt at a real console

**What to expect.** A session asks for the account password and refuses a wrong
one: on input that is piped in, one try, and the kept copy lets `sd-solo
<command>` run. At a real console there are three tries before the session ends.

**Check before relying on it.** At the keyboard, try the three tries at
`Password:`; a command line when the kept copy has been moved aside; and a
restart, which starts SD with nobody signed in.

### ssh at a real terminal

**What to expect.** An ssh session lands inside SD, and SD's terminal layer
drives a Windows console. Screen handling, cursor positioning and the editing
keys all go through that layer, and over ssh the pseudo-terminal belongs to the
ssh server rather than to the Windows console host.

**Check before relying on it.** Start one interactive session from a second
computer: the password prompt, then a full-screen operation and the arrow keys.

### A domain user in the ssh setting

**What to expect.** Solo's ssh server is configured with `AllowUsers` and the
lower-case name of a local user. A domain user is named `name@domain`.

**Check before relying on it.** On a domain-joined computer, sign in over ssh on
port 4251 and check that you land in SD. See [ssh access](08-ssh-access.html).

### Solo's own ssh server: starting at boot, and reaching it

**What to expect.** Solo's ssh server runs as SYSTEM, started by a scheduled task
at Windows start-up before anyone signs in, on port 4251. It checks a Windows
password, or a key in Solo's key file, and starts the session as you. Whether
another computer can reach port 4251 depends on the firewall rule and your
network. SD Core Solo and SD Core can be installed together, each with its own
ports; the computer's own ssh server on port 22 lands in SD Core, not in Solo.

**Check before relying on it.** Restart the computer, do not sign in, and connect
from a second computer with `ssh -p 4251`, typing the Windows password and then
the SD password. If SD Core is also installed, connect to port 22 as an SD Core
user.

### The ssh lock-out

**What to expect.** Three wrong passwords from one address in ten minutes lock
that address for ten minutes. A wrong password counts, a key does not, a sign-in
forgets the failures, and the address is read from the end of the log line. The
guard reads Solo's ssh server's lines in the Windows event log
(**OpenSSH/Operational**, level VERBOSE) and blocks the address with a Windows
Firewall rule. **A lock on this computer's own address (`127.0.0.1`) ends the
connection but cannot refuse the next one**, because Windows Firewall does not
filter loopback. A guess spread over many addresses is not stopped.

**Check before relying on it.** Install with ssh on and the firewall rule open,
give three wrong passwords at `ssh -p 4251` from a second computer, and see the
third end the connection, the next connection be refused, `-List` name the address
and its end time, `-Unlock` let it in again, and the lock lift by itself after ten
minutes. Then try a correct password and a key that the server does not know.
See [ssh access](08-ssh-access.html).

## Scheduled tasks

### A task that does not store your password

**What to expect.** A scheduled task that runs as you, when you are signed in,
runs `sd-solo <command>` with the kept copy of the account password. A task set
to *Run whether user is logged on or not* with *Do not store password* ticked is
signed in by Windows without your password, and the kept copy is protected by
it, so it may not open.

**Check before relying on it.** Create such a task and see whether the command
runs. See [Scheduled jobs](04-scheduled-jobs.html).

## Locking and contention

### Semaphores under heavy contention

**What to expect.** A semaphore is used on every record lock, so ordinary use
exercises it all the time.

**Check before relying on it.** If an application will have many sessions waiting
for the same lock, try that load on your own computer first and watch how long
the waiters wait.

### A lock held by a terminal session and wanted by an API connection

**What to expect.** Two local sessions compete correctly: record, update and
file locks are reported against the right holder, a waiting read is released
when the holder lets go, and a task lock refuses a second taker.

**Check before relying on it.** If an application mixes terminal sessions and API
clients on the same records, try an API client and a terminal session competing
for one record.

### Task locks taken twice by the same session

**What to expect.** Taking a task lock you already hold succeeds, and one
`unlock` releases it however many times you locked it: the test is "unowned or
mine", and `unlock` clears the slot outright. A program that takes the same lock
twice and expects to hold it after one `unlock` will be wrong.

## Application data

### Large and long-established data

**What to expect.** SD creates, writes, reads and deletes files, records,
indexes, select lists and sequential files, and the system files it starts with
are ordinary files of the same kind.

**Check before relying on it.** Before moving a production application, restore a
copy of its account and run it: a file of hundreds of thousands of records, a
deep dictionary or a schema built over years by someone else is the case worth
trying first.

## Sockets

### UDP and ICMP

**What to expect.** TCP is the supported protocol: listening, connecting,
accepting, reading, writing, the blocking and non-blocking modes and the error
codes. The flags `0x00010000` and `0x00020000` for UDP and ICMP exist in the
language, but SD makes no promise about sockets opened with them.

**Check before relying on it.** Use TCP. If you need a datagram socket, try a
datagram to a listener and back first.

## The MCP door

### An agent's MCP client through the door

**What to expect.** The launcher `solo-mcp.py` speaks MCP on its standard input
and output, and the rules inside `set.mcp` and `mcp.call` decide what a call may
do. See [The MCP door](25-the-mcp-door.html).

**Check before relying on it.** Turn the door on with `set.mcp on`, add the
launcher to your own MCP client, call each tool once, then `set.mcp off` and see
the next call refused. The audit trail shows each call and each refusal.

## SD BASIC statements that depend on something outside SD

| | needs |
|---|---|
| `sendmail` | a mail relay configured on the computer |
| `chgphant()` | a phantom process to change |
| `ccall()` | a C function registered into the executable |

**What to expect.** All three compile, and none of them does anything useful
without what it needs. Nothing else in the documentation depends on them.

**Check before relying on it.** Try each with what it needs, on your own system.

## The terminal editors' key bindings

**The key bindings of Microsoft Edit are the editor's own, unmodified.** SD only
installs it and launches it. If a binding surprises you, that is the program's
own documented behavior.

## What is not on this page

The places where SD Core Solo for Windows deliberately differs from OpenQM,
ScarletDME, upstream `sdb64` or the multiuser SD Core for Windows are documented
as differences, in the pages that describe the feature. This page is only about
behavior that depends on your own environment.

## See also

[Sessions and locks](06a-sessions-and-locks.html) covers the locking model that
some of the entries above qualify. [Scheduled jobs](04-scheduled-jobs.html)
covers the Task Scheduler entry. [System limits](16a-system-limits.html) states
the figures SD works within.
