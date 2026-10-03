Title: ssh access
Subtitle: Reaching SD on this computer over ssh, on SD Core Solo's own port, and what it costs.

**An ssh sign-in to port 4251 lands inside SD.** SD Core Solo runs an ssh server
of its own on port **4251**. It checks your key; then, instead of a Windows
prompt, you get `sd-solo`, which asks for the account password.

```
ssh -p 4251 you@this-computer        <- your key, ssh's own check
Password:                            <- SD's account password
:
```

**Only a key can log in.** The server never accepts a password on this port, so
there is nothing to guess. The port is fixed: it is not a setting.

**ssh works while nobody is signed in to the computer**: the server is started
when Windows starts, by its own scheduled task, and SD is started at start-up by
its task.

**It does not touch port 22.** Port 22 is the computer's ordinary ssh port, the
one the Windows OpenSSH Server service listens on and the one SD Core (the
multi-user product) uses. ssh to 22 goes to whatever answers there; ssh to 4251
goes to SD Core Solo. **So SD Core Solo and SD Core can be installed on one
computer.**

## What the installer sets up

| | |
|---|---|
| **The ssh server** | the `sshd.exe` that Microsoft's OpenSSH package installs, run as **you**, without administrator rights, with a configuration of its own: `%USERPROFILE%\SDCoreSolo\ssh\sshd_config`. The file is rewritten each time the server starts, so do not edit it |
| **The task** | **SD Core Solo SSH**, which starts the server when Windows starts, whether or not you are signed in, and starts it again, up to three times a minute apart, if it stops |
| **Its host key** | `%USERPROFILE%\SDCoreSolo\ssh\ssh_host_ed25519_key`, made the first time the server starts. It is Solo's own, not the Windows OpenSSH Server's |
| **The key file** | `%USERPROFILE%\SDCoreSolo\ssh\authorized_keys`, the only place the server looks for keys |

The configuration allows **only your Windows user**, only by key, and gives that
user `sd-solo` and nothing else:

```
Port 4251
StrictModes yes
PasswordAuthentication no
AuthenticationMethods publickey
AllowUsers you
DisableForwarding yes
ForceCommand "C:\Users\you\SDCoreSolo\usr\bin\sd-solo.exe"
```

**Nothing in the Windows OpenSSH Server's own configuration, service or port 22
firewall rule is changed.** The OpenSSH package must be installed, because Solo
runs the `sshd.exe` it provides; the Windows service does not have to be
running, and need not be set to start.

**If there is no OpenSSH package**, standalone installs offer to install
Microsoft's OpenSSH server from the release's `ssh-server` folder, and managed
installs always do. See [Installing](01-installation.html).

**It is not a choice.** The installer has no box for landing in SD; that is what
ssh access to SD Core Solo is. The one choice it asks is who may reach the port
(below).

## Keys

**The SD Core for Linux server adds its own key** on a managed computer, through
the API (see [Managed mode](15-managed-mode.html)), and removes it again if it is
asked to.

**To log in from your own computer**, add your public key to
`%USERPROFILE%\SDCoreSolo\ssh\authorized_keys` as one line (the file is in your
own folder, so no administrator is needed):

```
ssh-ed25519 AAAA... you@your-computer
```

The server checks the file's permissions (`StrictModes yes`) and refuses a key
file that anyone but you, SYSTEM and the administrators may write to. A file in
your own profile folder has the right permissions as Windows creates it.

**Keys from an earlier version are moved.** Before this version, SD Core
Solo put the server's key in `%USERPROFILE%\.ssh\authorized_keys`. The first
start moves that key into the new file, keeps a copy of your old file beside it
as `authorized_keys.sdcoresolo-backup`, and leaves any key of your own in the old
file alone.

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
| **Standalone** | the installer asks. Unticked, the firewall rule for port 4251 is limited to `127.0.0.1` — this computer only |
| **Managed** | always open to other computers, because the SD Core for Linux server connects from elsewhere |

**`ssh -p 4251 localhost` works either way**, because Windows does not filter
traffic that never leaves the computer.

**The rule is Solo's own, named `SD-Solo-SSH-In-TCP`**, and it is the one step of
Solo's ssh that needs an administrator: a firewall rule is machine-wide. The
uninstaller removes it.

**To change it afterwards**, from an elevated PowerShell:

```
powershell -ExecutionPolicy Bypass -File "%USERPROFILE%\SDCoreSolo\solo-ssh-firewall.ps1" -Restrict
powershell -ExecutionPolicy Bypass -File "%USERPROFILE%\SDCoreSolo\solo-ssh-firewall.ps1" -Open
powershell -ExecutionPolicy Bypass -File "%USERPROFILE%\SDCoreSolo\solo-ssh-firewall.ps1" -Show
```

`-Restrict` limits the rule to this computer, `-Open` opens it to any address,
`-Remove` takes it away and `-Show` reports without changing anything. It exits
`0` applied, `1` failed, `2` refused (for example, not elevated). It scopes the
rule's remote address rather than disabling it, so the rule reads correctly in
`wf.msc`.

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

**These have not been measured on a real computer.** See
[Features the developers could not test](19-features-the-developers-could-not-test.html).

- **That the server starts at Windows start-up with nobody signed in.** The task
  runs as you without a stored password; that this logon type starts the server
  at boot has not been seen.
- **That another computer can reach port 4251** through the firewall rule. Only a
  connection from this computer has been tried.
- **That SD Core Solo and SD Core work together on one computer**, each answering on its own ssh port.
- **The user name for a domain user.** The configuration names a local user by the
  lower-case user name, and a domain user as `name@domain`. Whether the server
  matches a domain user by that form has not been measured. On a domain-joined
  computer, check that your ssh sign-in lands in SD before relying on it.
