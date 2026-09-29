Title: API access
Subtitle: The client API, the port it answers on, its login, and what an API session is.

**The client API lets a program on this computer, or another one, use SD as a
back-end data store.** Application code uses `SDConnect()` and the rest of the
client library as with any SD Core — see [Client distribution](10-client-distribution.html).

## Is it on?

| | |
|---|---|
| **Standalone** | only if **Provide the SD Core API** was ticked when installing. Off by default |
| **Managed** | always on, and open to other computers, because the SD Core for Linux server connects through it |

**Declined at installation, there is no listener at all**: the installer
writes an `sd.conf` with no `APIPORT` line, and SD opens no socket.

## Signing in

| | |
|---|---|
| **User name** | always `sduser` |
| **Password** | the account password — or, on a managed computer, the global password, which the server uses |
| **Account** | `sduser` |

**The login is SCRAM-SHA-256, inside TLS 1.3.** The password is never sent in
any form; the server sets a challenge only someone who knows the password can
answer, and then proves itself back, so a program that grabbed the port before
SD started cannot collect passwords by pretending to be SD.

**A client that sends a password in clear is refused** with *"Cleartext login
is no longer supported; this server requires SCRAM authentication"*.

**A wrong password is refused**, and the refusal is written to the audit trail
— for example `API REFUSED user=sduser reason=wrong password`.

**On a computer installed from a control file**, until the account password
has been chosen at the keyboard, only the global password is accepted.

**`SDConnectLocal` is disabled.** It signed in with no password, which Solo
does not allow; a client calling it gets *SDConnectLocal is not available in
SD Core Solo for Windows - connect with SDConnect and the account password*.
Use `SDConnect` to this computer's own address instead.

## The port

**`APIPORT=4243` in `sd.conf`.** Whether other computers can reach it is a
Windows Firewall rule:

| | |
|---|---|
| **Standalone** | the installer asks; unticked, the rule allows this computer only |
| **Managed** | open to other computers |

**To change it afterwards**, from an elevated PowerShell:

```
powershell -ExecutionPolicy Bypass -File "%USERPROFILE%\SDCoreSolo\api-firewall.ps1" -Open
powershell -ExecutionPolicy Bypass -File "%USERPROFILE%\SDCoreSolo\api-firewall.ps1" -Restrict
powershell -ExecutionPolicy Bypass -File "%USERPROFILE%\SDCoreSolo\api-firewall.ps1" -Show
```

`-Open` allows any address, `-Restrict` this computer only, and `-Show`
reports without changing anything (and needs no elevation). It exits `0`
applied, `1` failed, `2` refused.

**To turn the API off or on** after installing, remove or add the `APIPORT`
line in `%USERPROFILE%\SDCoreSolo\sd.conf`, then `sd -stop` and `sd -start`.
The listener is read only at start-up.

> **`APILOGIN` is not an off switch.** It decides whether the API demands a
> password. `APILOGIN=0` is the **weaker** setting, not the safer one. Do not
> reach for it.

## An API session is you

**It runs as your Windows user**, in the account `sduser`, with the same
rights as a session at the keyboard — including `sh` and `OS.EXECUTE`, which
a remote client can therefore use. The part of SD that handles the network
encryption runs on a restricted copy of your Windows token that cannot read
your files; the session itself is ordinary and unelevated.

**A session signed in with the global password is a server session**, with
the administrator commands unlocked. See [Managed mode](15-managed-mode.html).

### A session is confined to its own account

| | |
|---|---|
| **Allowed** | everything inside the account, and the SD system files every account needs — messages, `syscom`, the dictionaries, `sd.voclib` |
| **Refused** | opening, renaming, deleting or listing anything else |

**A refused `OPEN` takes the `ELSE` branch and `STATUS()` is 3035** — *not
permitted*, not *not found*. That distinction matters when debugging: 3035 is
this confinement, not a missing file.

**If your data lives outside the account**, name the directory in the
**`NETDIRS`** setting in `sd.conf`, separating several with a semicolon.
`config('NETDIRS')` prints what is in force. SD's password file, its program
catalogue and its account register are never reachable from an API session,
and cannot be added to `NETDIRS`. A server session may reach `GLOBAL.BP.OUT`.

## Client libraries

| | |
|---|---|
| 64-bit | `sdclilib.dll`, in `%USERPROFILE%\SDCoreSolo\usr\clients\client64` |
| 32-bit | in `...\usr\clients\client32` |

**Use a client library from this release or later.** One that predates SCRAM
is refused. See [Client distribution](10-client-distribution.html).

**Programs using the `!sdclient` class** — SD BASIC reaching another SD over
the API — speak the same login, from this release at both ends.
