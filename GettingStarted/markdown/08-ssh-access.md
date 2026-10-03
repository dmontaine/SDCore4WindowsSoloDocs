Title: ssh access
Subtitle: Reaching SD on this computer over ssh, on SD Core Solo's own port, with your Windows account name and password.

**An ssh sign-in to port 4251 lands inside SD.** SD Core Solo runs an ssh server
of its own on port **4251**. You sign in with your Windows account name and its
Windows password, as you would to any computer; then, instead of a Windows
prompt, you get `sd-solo`, which asks for the SD account password.

```
ssh -p 4251 you@this-computer
you@this-computer's password:        <- your Windows password, ssh's own check
Password:                            <- SD's account password
:
```

**There is no key to set up first.** A key is an optional extra (below), never
the only way in. The port is fixed: it is not a setting.

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
| **The ssh server** | the `sshd.exe` that Microsoft's OpenSSH package installs, started by a scheduled task, **SD Core Solo SSH**, as **SYSTEM**, when Windows starts, and again up to three times a minute apart if it stops |
| **Its folder** | `C:\ProgramData\SDCoreSolo\ssh`: the server's configuration (`sshd_config`) and its host key. Only administrators can change it, and **the installer reads the permissions back after setting them and stops if anyone else could write there** |
| **The key file** | `%USERPROFILE%\SDCoreSolo\ssh\authorized_keys`, in your own folder, for the optional keys |

The configuration allows **only your Windows user**, gives that user `sd-solo`
and nothing else, and takes a password or a key:

```
Port 4251
StrictModes yes
PasswordAuthentication yes
PubkeyAuthentication yes
AllowUsers you
DisableForwarding yes
ForceCommand "C:\Users\you\SDCoreSolo\usr\bin\sd-solo.exe"
```

**Why the server runs as SYSTEM.** A server run as an ordinary user can check
your Windows password but then cannot start your session: Windows refuses with
*"a required privilege is not held"* (error 1314). This was measured. The
Windows OpenSSH Server service runs as SYSTEM too. **Because SYSTEM
trusts the configuration, the configuration and host key are in an
administrators-only folder, made by the installer's one administrator step,**
not in your own folder where a program of yours could change what runs as
SYSTEM. Your `sd-solo.exe` still runs as **you**: the server starts it under
your account after you sign in.

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

## Warnings and things to know

**Anyone who can reach the port can try passwords for your Windows account, and
Windows can then lock you out of it.** Wrong passwords count against the
account like wrong passwords at the sign-in screen. On the computer this was
developed on, the policy locks an account after 10 wrong tries within 10
minutes (`net accounts` shows yours), and that includes you. So **leave the firewall rule limited to this computer unless other
computers need to reach SD**, and use a long Windows password. (Windows OpenSSH
on port 22 has the same property.)

**The password is not sent in the clear.** ssh encrypts the connection before
it asks for the password, so it crosses the network inside the encrypted
channel; the server receives it decrypted, as any ssh server does. **Check the
host key the first time you connect**: ssh shows its fingerprint and asks. It
is the one in `C:\ProgramData\SDCoreSolo\ssh\ssh_host_ed25519_key.pub`, and
`ssh-keygen -l -f` prints it.

**A Windows account with no password cannot sign in over ssh**, because Windows
refuses a network sign-in for a blank password by default. Give the account a
password first.

## Keys, as an optional extra

**A key never replaces the password; it is another way in.** The SD Core for
Linux server adds its own key on a managed computer, through the API (see
[Managed mode](15-managed-mode.html)), and removes it again if it is asked to.

**To sign in from your own computer with a key**, add your public key to
`%USERPROFILE%\SDCoreSolo\ssh\authorized_keys` as one line (the file is in your
own folder, so no administrator is needed):

```
ssh-ed25519 AAAA... you@your-computer
```

The server checks the file's permissions (`StrictModes yes`) and refuses a key
file that anyone but you, SYSTEM and the administrators may write to. A file in
your own profile folder has the right permissions as Windows creates it.

**Keys from an earlier version are moved.** Before this version, SD Core Solo
put the server's key in `%USERPROFILE%\.ssh\authorized_keys`. Installing moves
that key into the new file, keeps a copy of your old file beside it as
`authorized_keys.sdcoresolo-backup`, and leaves any key of your own in the old
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

**Use `127.0.0.1` rather than `localhost` for a connection from this computer**:
Windows' ssh client tries the IPv6 address first and does not fall back to
IPv4. `ssh -p 4251 you@127.0.0.1` works either way the rule is set, because
Windows does not filter traffic that never leaves the computer.

**The rule is Solo's own, named `SD-Solo-SSH-In-TCP`**, and creating it needs an
administrator: a firewall rule is machine-wide. The uninstaller removes it.

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
the keyboard. SD drops administrator rights for an ssh session as it does for
itself, so it runs on an ordinary, unelevated token even when your Windows
account is an administrator.

**On a computer installed from a control file**, before the account password
has been chosen at the keyboard, an ssh session is told:

```
This account has no password yet. Set it at this computer's keyboard first; until then only the global password is accepted.
```

## Not measured

**These have not been measured on a real computer.** See
[Features the developers could not test](19-features-the-developers-could-not-test.html).

- **A sign-in made while nobody is signed in to the computer.** After a restart,
  the server was seen running before the first sign-in succeeded (the task
  launched eight seconds before it), but nobody tried to sign in to it in that
  window. (A Windows password sign-in through this server reached SD from the
  same computer and from a second one, typed by the owner himself, and a key
  sign-in reached SD from the same computer.)
- **That SD Core Solo and SD Core work together on one computer**, each answering on its own ssh port.
- **The user name for a domain user.** The configuration names a local user by the
  lower-case user name, and a domain user as `name@domain`. Whether the server
  matches a domain user by that form has not been measured. On a domain-joined
  computer, check that your ssh sign-in lands in SD before relying on it.
