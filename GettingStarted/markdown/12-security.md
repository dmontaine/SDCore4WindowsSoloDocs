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
| **`ADMIN`** | the administrator commands and direct VOC edits, in a session | a program that reads the files |
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
| Administrator commands | refused until `ADMIN`. The eight that had no check of their own — `CONFIG`, `LISTU`, `LIST.LOCKS`, `LIST.READU`, `LOCK`, `CLEAR.LOCKS`, `SET.DATE`, `CLEAN.ACCOUNT` — are gated too. See [Administrator commands](06-administrator-commands.html) |
| The VOC | direct edits need `ADMIN`; the global catalogue is changed by nobody in a session |
| The daemon | runs as you on an ordinary token, Administrators deny-only and Medium integrity, even from an administrator's account |
| ssh | your own Windows account name and password only, on Solo's own port 4251, forced into `sd-solo`, no forwarding; the server runs as SYSTEM with its configuration in an administrators-only folder — see [ssh access](08-ssh-access.html) |
| The API | off unless chosen (always on in managed mode); SCRAM inside TLS 1.3; a session confined to the account's files — see [API access](09-api-access.html) |

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
setup log shows *Internal session admitted (opened by solo-setup)*. It is not a
way into a running system. **SDSYS, SD's own system account, is never signed
in to**: nobody logs in to it, and Solo has no `LOGTO` verb.

## On a managed computer

**The SD Core for Linux server can add to what is locked** — the global
catalogue, the list of denied commands — and it signs in with the global
password. **All of that is SD enforcing it, and none of it is Windows
enforcing it**: the user of the computer owns the files and can change them
from outside SD. What managed mode protects is what happens *inside* SD. See
[Managed mode](15-managed-mode.html).

## What you can do further

**Lock a session into one application** by removing `basic` and `run` from the
VOC (which needs `ADMIN`) and turning off its break key (`pterm break off`), so
it can neither compile nor run anything else and cannot interrupt out to a TCL
prompt. Neither is a setting; both are done by hand.

**Turn the API off** if you do not use it — in standalone mode it is off
unless it was chosen; see [API access](09-api-access.html). **Restrict ssh to
this computer** if other computers do not need it — see
[ssh access](08-ssh-access.html). Wherever the OpenSSH package is installed,
Solo runs its own ssh server on port 4251, your Windows sign-in lands in SD and
the server starts with Windows. Anyone who can reach the port can try passwords
for your Windows account, so keep the firewall rule limited to this computer
unless you need it open.

## Continued in

[Security and the operating
system](12a-security-and-the-operating-system.html) — reaching the operating
system from inside SD, and the audit trail.
