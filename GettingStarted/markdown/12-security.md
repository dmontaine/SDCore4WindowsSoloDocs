Title: Security
Subtitle: Who can reach SD Core Solo for Windows, what each password guards, and what does not protect it.

SD Core Solo for Windows is built for **one Windows user on one computer**,
and its protection follows from that. Read this page before relying on it for
anything beyond that.

## The position in one paragraph

**Your Windows user owns everything**: the programs, the data, the account and
the credential files all live in `%USERPROFILE%\SDCoreSolo`, and every SD
process runs as you. **Windows' own protection of your profile is what keeps
other Windows users out.** **SD's passwords keep other *people* out of SD** —
at the keyboard, over ssh and through the API — and its administrator gate
keeps a session from doing more than it should by accident. **Neither stops
somebody who is already signed in to Windows as you**, or an administrator of
the computer, from reading or changing the files directly.

## What each thing guards

| | Guards | Does not guard |
|---|---|---|
| **Windows** | your profile from other Windows users | it from you, and from the computer's administrators, who can read `SDCoreSolo` — the credential store included |
| **The account password** | every SD session: keyboard, ssh, API, and a command line | the files themselves |
| **`admin`** | the administrator commands and direct VOC edits, in a session | a program that reads the files |
| **The `sh` and `os` switches** | the way out of SD onto the computer from inside a session: the `sh` verb, `OS.EXECUTE`, files and directories outside the account, the embedded Python, `micro` — off as delivered | what a program that already runs as you does outside SD |
| **The global password** | a managed computer, for the SD Core for Linux server | — |
| **The deny list** | commands the user may not run on a managed computer | the same files, outside SD |

**The account password is the gate SD adds.** Being signed in to Windows as you
is not enough to get an SD session: SD asks. That matters when somebody else
has your Windows sign-in for a moment, and above all for ssh and the API, which
are reached from other computers. See
[The account and its passwords](05-account-types.html).

## What ships secured

| | |
|---|---|
| Every session | asks the account password |
| Administrator commands | refused until `admin`. The eight that had no check of their own — `config`, `listu`, `list.locks`, `list.readu`, `lock`, `clear.locks`, `set.date`, `clean.account` — are gated too. See [Administrator commands](06-administrator-commands.html) |
| The VOC | direct edits need `admin`; the global catalog is changed by nobody in a session |
| The shell and OS access | refused until the administrator turns the `sh` or `os` switch on with `set.os.access`: off on a new install unless you answered yes when the installer asked, and kept as they were on an upgrade — see [Operating system access](06b-operating-system-access.html) |
| `NETDIRS` | a door in `sd.conf`, **no line as delivered**: folders outside the account, separated by semicolons, that a session with the `os` switch off may still read and write by pathname. Listing one is a loosening for every session of the account, on every route — see [Configuration](16-configuration.html) |
| The NET gate | API request 29 (open an `account;file` file for a client) is refused until the administrator sets `NETFILES=2` in `sd.conf` and turns the gate on with `set.net.access on`: both off as delivered; and a VOC pointer that leads out of the account is refused unless the account has the `os` switch on — see [API access](09-api-access.html) |
| File transfer | an sftp program on another computer can put text files into the account's file `xfer.in` and take them from `xfer.out`, over Solo's ssh (port 4251). **Off twice as delivered**: `XFER=1` in `sd.conf` (read when Solo starts) and `set.xfer.access on`. Opening it means a program that knows your Windows password can bring files into the account while you wait in `receive`; it sees only that one folder, only plain text, within the size limits (`XFERFILE`, `XFERSLOT`) and only while the transfer is open. **The SD account password is not asked of the sftp connection** (it has no terminal to give it): what stands in for it is that nothing moves unless your own session, which did give it, is waiting in `receive` or `send` at that moment — see *Moving Files In and Out* in the User documentation |
| Scheduled jobs | SD's own jobs (`set.job`) run one of your SD commands from a Windows Task Scheduler task, as your Windows user, **whether you are signed in or not**. **Off as delivered**: `set.jobs.access on`, and off refuses every job at its next run. A job is made or changed only with the account password typed at that moment. The task is made by the installer's SYSTEM helper (only an administrator can make a task that runs while you are signed out), which is told only the job's name and schedule; the command stays in SD. While you are signed out Windows will not open the kept copy of the password, so `run.job`, and nothing else, signs in by being the Windows user who owns SD's password folder. That adds no way in: while you are signed in any program of yours can sign in with the kept copy, and while you are signed out a program can run as you only from a task an administrator registered — see [Scheduled jobs](04-scheduled-jobs.html) |
| The daemon | runs as you on an ordinary token, Administrators deny-only and Medium integrity, even from an administrator's account |
| ssh | your own Windows account name and password only, on Solo's own port 4251, forced into `sd-solo`, no forwarding; the server runs as SYSTEM with its configuration in an administrators-only folder; and three wrong passwords from one address lock that address for ten minutes (a Windows Firewall rule; not enforced for this computer's own address) — see [ssh access](08-ssh-access.html) |
| The API | off unless chosen; SCRAM inside TLS 1.3; a session confined to the account's files — see [API access](09-api-access.html) |
| `sd-solo -internal` | closed once the installer has finished |

**The account is the same one for every session**, so the gates are about *how
you arrived and what you unlocked*, not about which account you are in.

## What SD keeps

| | |
|---|---|
| **The passwords** | none is stored. `$cred` holds a verifier for each — the account, the administrator and the global — that cannot be turned back into a password |
| **The kept copy** | a copy of the account password, encrypted with Windows' own protection for your Windows user, that lets `sd-solo <command>` sign in without typing. **Any program running as you can ask Windows to decrypt it**; it keeps it from other Windows users, not from other programs of yours |

**Whoever can replace a verifier can set a password they know**, and anyone who
is your Windows user, or an administrator of the computer, can. That is the
limit stated above, not a fault: the passwords protect SD sessions, not the
files under them.

## The installer's own door

**`sd-solo -internal` is how the installer runs SD's setup steps**, and it is admitted
only by a one-shot marker file the installer writes and SD consumes — the
setup log shows *Internal session admitted (opened by solo-setup)* and the audit
trail shows *internal session admitted* with the name of the writer. It is not a
way into a running system. **SDSYS, SD's own system account, is never signed
in to**: nobody logs in to it, and Solo has no `logto` verb.

## On a managed computer

**The SD Core for Linux server can add to what is locked** — the global
catalog, the list of denied commands — and it signs in with the global
password. **All of that is SD enforcing it, and none of it is Windows
enforcing it**: the user of the computer owns the files and can change them
from outside SD. What managed mode protects is what happens *inside* SD. See
[Managed mode](15-managed-mode.html).

**The server can also put an ssh key on the computer.** Only a session signed in
with the global password may, at most four keys are kept, each can start
`sd-solo` and nothing else, and every use is audited. That is the global password
reaching one step further — into Solo's own ssh key file — so **whoever holds the
global password can now also reach this computer over ssh as your Windows user**.
It could already run the administrator commands. **On the client side, the
library pins the computer's TLS certificate on first use**, which stops someone
posing as the computer from collecting a login proof to crack offline; the first
connection is trusted. See [API access](09-api-access.html).

## What you can do further

**Lock a session into one application** by removing `basic` and `run` from the
VOC (which needs `admin`) and turning off its break key (`pterm break off`), so
it can neither compile nor run anything else and cannot interrupt out to a TCL
prompt. Neither is a setting; both are done by hand.

**Turn the API off** if you do not use it — it is off unless it was chosen; see
[API access](09-api-access.html). **Restrict ssh to this computer** if other
computers do not need it, or **leave it off** — see
[ssh access](08-ssh-access.html). When Solo's ssh server is on, it runs on
port 4251, your Windows sign-in lands in SD and the server starts with
Windows. Anyone who can reach the port can try passwords
for your Windows account, so keep the firewall rule limited to this computer
unless you need it open. **Do not give the ssh key line to anything you would
not give the account password to** — see [ssh access](08-ssh-access.html).

**Enable full-disk encryption (BitLocker) and a screen lock**, as you would for
any computer that holds data you care about: everything above assumes the
computer itself is in your hands. How to put SD's folder on an encrypted drive is
on [Encrypting the data SD keeps on disk](18a-data-at-rest.html).

## Continued in

[Security and the operating
system](12a-security-and-the-operating-system.html) — reaching the operating
system from inside SD, and the audit trail.
