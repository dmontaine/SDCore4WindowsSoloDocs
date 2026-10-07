Title: API access
Subtitle: The client API, the port it answers on, its login, and what an API session is.

**The client API lets a program on this computer, or another one, use SD as a
back-end data store.** Application code uses `SDConnect()` and the rest of the
client library as with any SD Core — see [Client distribution](10-client-distribution.html).

## Is it on?

**Only if it was chosen when installing** — **Provide the SD Core API** ticked,
or `api=local` or `api=open` in the control file. Off by default, with or
without a global password. A managed computer needs it on and open, because the
SD Core for Linux server connects through it; nothing switches it on for you.

**Declined at installation, there is no listener at all**: `sd.conf` has its
`APIPORT` line commented out, and SD opens no socket.

**Chosen, the listener is switched on after the firewall rule exists.** The
installer's `sd.conf` always starts with `APIPORT` commented out, so the
installer's own first start of SD opens no port and Windows shows no firewall
alert. The administrator step then makes the API's firewall rule, switches
`APIPORT` on with `solo-api-listener.ps1`, and starts SD from the startup task.
The install report says whether the API is listening on 4249.

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

**On a computer installed from a control file that gave a global password**,
until the account password has been chosen at the keyboard, only the global
password is accepted.

**`SDConnectLocal` is disabled.** It signed in with no password, which Solo
does not allow; a client calling it gets *SDConnectLocal is not available in
SD Core Solo for Windows - connect with SDConnect and the account password*.
Use `SDConnect` to this computer's own address instead.

## The port

**The API port is 4249, and nothing can change it.** It used to be 4243, which
OpenQM and ScarletDME also use, and the full SD Core for Windows uses 4247, so
the two products can be installed on one computer. `APIPORT` in `sd.conf` now only
switches the API on: any number above zero means on, and SD listens on 4249
whatever the number is. A file that says `APIPORT=4243` keeps working and means
4249. A program that names port 4243 must name 4249 instead; one that names no
port needs no change. Whether other computers can reach it is a Windows Firewall
rule, named `SD-Solo-API-In-TCP`:

| | |
|---|---|
| **Let other computers reach it** | the installer asks, or `api=open` in the control file. Unticked, or `api=local`, the rule allows this computer only |
| **A managed computer** | needs it open. Nothing opens it for you |

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
line in `%USERPROFILE%\SDCoreSolo\sd.conf`, then `sd-solo -stop` and `sd-solo -start`.
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
the administrator commands unlocked. See [Managed computers](15-managed-mode.html).

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

## The client library remembers each server's certificate

**The first time a client library connects to a server, it keeps that server's
certificate; after that it connects only to a server that presents the same
one.** This is *first use* trust: the first connection is taken on trust, and
every later one is checked. It closes the one gap SCRAM leaves — someone
posing as the server could otherwise collect a login and try to guess the
password offline.

| | |
|---|---|
| **What is kept** | the SHA-256 of the server's whole certificate, as 64 lower-case hex digits |
| **Where** | `%USERPROFILE%\.sdcore\known_servers`, or the file named by the environment variable `SD_KNOWN_SERVERS`; one line per server, `host:port` and the digits. A different port is a different server |
| **When it is checked** | right after the secure connection is made and before anything is sent — no login byte, no password |

**A changed certificate is refused**, with *"THE SERVER'S CERTIFICATE HAS CHANGED
since this client first connected to host:port (pinned ..., now ...). The
connection was refused before any password was sent. If the server was
reinstalled, remove the line for host:port from ... and connect again"*. Nothing
updates the file by itself and nothing asks you; **removing the line is the
remedy**, and only if you know why the certificate changed. **A file that cannot
be opened or written refuses the connection** ("cannot pin the server: cannot
open ...") rather than connecting unchecked.

**Installing SD Core Solo again makes a new certificate**, because the
installation is replaced. A client that already knew the computer will refuse it
until its line is removed.

**Programs using the `!sdclient` class** — SD BASIC reaching another SD over
the API — speak the same login, from this release at both ends.
