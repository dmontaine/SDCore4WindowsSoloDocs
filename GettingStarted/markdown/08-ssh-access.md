Title: ssh access
Subtitle: Reaching SD on this computer over ssh, what the installer sets up, and what it costs.

**An ssh sign-in as your Windows user lands inside SD.** The ssh server checks
your Windows password (or key) as it would for any sign-in; then, instead of a
Windows prompt, you get `sd`, which asks for the account password.

```
ssh you@this-computer
you@this-computer's password:        <- your Windows password, ssh's own check
Password:                            <- SD's account password
:
```

**ssh works while nobody is signed in to the computer**: the ssh server is set
to start with Windows, and SD is started at start-up by its task.

## What the installer sets up

**Wherever an OpenSSH server is present, the installer writes one block at the
end of its configuration**, `C:\ProgramData\ssh\sshd_config`:

```
# BEGIN SD Core Solo - added by its installer, removed by its uninstaller
Match User "you"
    ForceCommand "C:\Users\you\SDCoreSolo\usr\bin\sd.exe"
    DisableForwarding yes
# END SD Core Solo
```

| | |
|---|---|
| `Match User` | **only your Windows user** is affected. Other Windows users of the computer sign in over ssh as before |
| `ForceCommand` | your ssh session runs `sd` and nothing else |
| `DisableForwarding` | no port forwarding for you, which `ForceCommand` alone would not stop |

**The change is checked before it stays**: `sshd -t` must accept the new
configuration, or the original is put back and the installer says so. The
block is removed by the uninstaller, and the rest of the file is left as it was.

**It is not a choice.** The installer has no box for it; landing in SD is what
ssh access to SD Core Solo is.

**If there is no ssh server**, standalone installs offer to install
Microsoft's OpenSSH server from the release's `ssh-server` folder, and managed
installs always do. See [Installing](01-installation.html).

## The cost: no scp or sftp to your user, and no Windows prompt

**Your ssh sign-in cannot copy files in or give you a Windows prompt.** The
command is forced, so there is no file-transfer subsystem left to run and no
shell. That is the accepted cost of landing in SD.

**The cost is inbound only.** `scp` or WinSCP running **on** this computer,
connecting outward, is an ssh *client* and is not affected. So copy files by
pulling them from here.

**To reach Windows from an ssh session, use `sh`** inside SD — see
[Operating system access](06b-operating-system-access.html).

## Reaching the computer from the network

| | |
|---|---|
| **Standalone** | the installer asks. Unticked, the ssh server's firewall rule is limited to `127.0.0.1` — this computer only |
| **Managed** | always open to other computers, because the SD Core for Linux server connects from elsewhere |

**`ssh localhost` works either way**, because Windows does not filter traffic
that never leaves the computer.

**To change it afterwards**, from an elevated PowerShell:

```
powershell -ExecutionPolicy Bypass -File "%USERPROFILE%\SDCoreSolo\ssh-firewall.ps1" -Installed -Restrict
powershell -ExecutionPolicy Bypass -File "%USERPROFILE%\SDCoreSolo\ssh-firewall.ps1" -Installed -Open
powershell -ExecutionPolicy Bypass -File "%USERPROFILE%\SDCoreSolo\ssh-firewall.ps1" -Show
```

`-Restrict` limits the rule to this computer, `-Open` opens it to any address,
and `-Show` reports without changing anything. It exits `0` applied, `1`
failed, `2` refused or the rule is not there yet. It scopes the rule's remote
address rather than disabling it, so the rule reads correctly in `wf.msc`.

## What an ssh session is

**It is you**: the same account, `sduser`, and the same rights as a session at
the keyboard. It runs on an ordinary, unelevated token even when your Windows
account is an administrator — SD drops the rights for an ssh session as it
does for itself.

**On a computer installed from a control file**, before the account password
has been chosen at the keyboard, an ssh session is told:

```
This account has no password yet. Set it at this computer's keyboard first; until then only the global password is accepted.
```

## Not measured

**The `Match User` line names a local user by the lower-case user name, and a
domain user as `name@domain`.** Whether Win32-OpenSSH matches a **domain** user
by that form has not been measured. On a domain-joined computer, check that
your ssh sign-in lands in SD before relying on it.
