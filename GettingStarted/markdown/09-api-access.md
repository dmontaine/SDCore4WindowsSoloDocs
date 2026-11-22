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

**The client library pins the server's certificate.** The first connection from a
client to an address and port is trusted, and its certificate is remembered — the
SHA-256 of the whole certificate — in `%USERPROFILE%\.sdcore\known_servers` (or the
file named by the environment variable `SD_KNOWN_SERVERS`), one line,
`<host>:<port> <64 hex digits>`, per server. A different port is a different
server. Any later connection to the same address and port that presents another
certificate is **refused before a single byte of the login is sent**, with a
message that names both fingerprints and the line to remove. A reinstalled
computer has a new certificate: remove its line and connect again. **Removing the
line is the remedy**, and only if you know why the certificate changed. **A store
the client cannot read or write refuses the connection** rather than connecting
unchecked. This is done in the client library, so it applies to every program that
uses `SDConnect`. **The very first connection is trusted**, as with ssh: someone in
the middle at that moment would be remembered instead of the computer.

**A managed computer's server can ask this computer to install its ssh key** over
the API, once it has signed in with the global password: see
[Managed mode](15-managed-mode.html).

**A client that sends a password in clear is refused** with *"Cleartext login
is no longer supported; this server requires SCRAM authentication"*.

**A wrong password is refused**, and the refusal is written to the audit trail
— for example `api refused user=sduser reason=wrong password`. It is written
**before** the three-second wait SD adds to slow down guessing, so a program that
hangs up during the wait is recorded as well.

**The only account an API session may enter is `sduser`.** Asking for SDSYS, or any
other name, is answered *User not allowed in requested account* — the same wording
for an account that does not exist, so the API cannot be used to find out what
accounts there are. **The administrator password is not an API login**: it is
refused.

**On a computer installed from a control file**, until the account password has
been chosen at the keyboard, only the global password is accepted.

**`SDConnectLocal` is disabled.** It sends no password, which Solo does not
allow. A client that calls it gets, at once, *SDConnectLocal is not available in
SD Core Solo for Windows - connect with SDConnect and the account password*. Use
`SDConnect` to this computer's own address instead (`127.0.0.1`).

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
| **Let other computers on your network reach it** | the installer asks, or `api=open` in the control file. Unticked, or `api=local`, the rule allows this computer only |
| **A managed computer** | needs it open. Nothing opens it for you |

**To change it afterwards from SD**, after `admin`: `remote.api open`, `remote.api local` or
`remote.api off` (or the A key of `tool.remote`: the word is chosen from a list that starts on
`off`, and `open` wants the word `open` typed, in either case). With no word, `remote.api` reports what is
installed. A change asks for the administrator password again and is made by the doors
helper the installer set up. It works from any session, at this computer or over ssh or the API,
and needs no elevated window. Off also turns the listener off, and an API change to or from off
takes effect at the next `sd-solo -stop` and `sd-solo -start`. Both the change and its result are
on the audit trail.

> **The doors helper is a deliberate trade.** It does machine-wide changes (the firewall rule,
> the listener) for anyone who knows the administrator password, from any session, without a
> Windows prompt. Solo keeps that password in your own folder, so a program running as you could
> replace it; that is the security Solo has by design, and less than SD Core's.

**Or, from an elevated PowerShell:**

```
powershell -ExecutionPolicy Bypass -File "%USERPROFILE%\SDCoreSolo\api-firewall.ps1" -Open
powershell -ExecutionPolicy Bypass -File "%USERPROFILE%\SDCoreSolo\api-firewall.ps1" -Restrict
powershell -ExecutionPolicy Bypass -File "%USERPROFILE%\SDCoreSolo\api-firewall.ps1" -Show
```

**A Windows firewall Block rule wins over Solo's own rule, and Solo reports one but never removes
it.** Windows makes such a rule when somebody presses Cancel at its "allow this program through
the firewall?" alert for `sdwind.exe`, and you or another tool may make one on purpose. While an
enabled inbound Block rule covers Solo's `sdwind.exe` (the program the API listens in) or TCP port
4249, other computers cannot connect, whatever `remote.api open` set. So `remote.api open`,
`remote.api` with no word, `-Show` above and the `tool.remote` page each name the rule:

```
WARNING - the Windows firewall rule "<display name>" (<name>) BLOCKS the API,
so other computers cannot connect until it is disabled or removed
```

Solo leaves the rule alone because it may be somebody's deliberate choice. To let other computers
in, disable or remove it in Windows Defender Firewall with Advanced Security (`wf.msc`, Inbound
Rules), or from an elevated PowerShell with the name the warning gives:
`Disable-NetFirewallRule -Name "<name>"`. Doing so opens the API port to every computer the network
lets reach this one, subject to Solo's own rule and its sign-in.

`-Open` allows any address, `-Restrict` this computer only, and `-Show`
reports without changing anything (and needs no elevation). It exits `0`
applied, `1` failed, `2` refused.

**To turn the API off or on** after installing, run `solo-api-listener.ps1`
with `-Off`, `-On` or `-Show`, then `sd-solo -stop` and `sd-solo -start`. The
listener is read only at start-up.

```
powershell -ExecutionPolicy Bypass -File "%USERPROFILE%\SDCoreSolo\solo-api-listener.ps1" -Show
powershell -ExecutionPolicy Bypass -File "%USERPROFILE%\SDCoreSolo\solo-api-listener.ps1" -On
powershell -ExecutionPolicy Bypass -File "%USERPROFILE%\SDCoreSolo\solo-api-listener.ps1" -Off
```

It comments `APIPORT` out or puts it back in `sd.conf` and reads the file again
before it says it is done; `-Show` changes nothing. It exits `0` done, `1` the
file could not be written, `2` it could not tell (no file, or none of the three
switches given). **Make the firewall rule first** — `-Restrict` or `-Open`, above —
because Windows asks whether to allow a program the first time it listens with
no rule, and an *Allow* there makes rules open to any address.

> **The API always demands the account's password.** To turn the API off, take
> away its listener, `APIPORT`.

### Everything that reaches SD from outside, on one screen: `tool.remote`

```
tool.remote
```

A full-screen page for the administrator (`admin` first; a window of 140 columns by 43 lines or
more) of four views, chosen with the keys `1` to `4`:

| View | What it shows and does |
|---|---|
| `1` doors | what `remote.api` and `remote.ssh` say with no keyword: **the machine's own answer**, read back from `sd.conf`, Solo's ssh configuration (its listen address) and the firewall rules. `A` changes the API and `S` ssh (`off`, `local` or `open`, from a list that starts on `off`); `open` wants the word `open` typed. After the box the terminal goes to the verb, which asks for the administrator password again |
| `2` servers | the SDNet servers you have defined. `X` deletes the one under the cursor (`delete.server`); `N` asks every field and gives **the terminal to `set.server`**, whose `connect` step shows the key the server presents and asks whether to trust it |
| `3` connected | every session now by the door it came through: user number, kind (user, api, sdnet, phantom), login user, account and origin |
| `4` TLS | this installation's TLS identity as `settings.report` gives it. Never the private key |

Each change runs only after a box showing the command was answered Run (**Cancel is
what Enter chooses**); the verbs write their own audit lines, and opening the page
writes `tool start app=tool.remote`. `F5` reads again, `F1` explains, `Ctrl-Q` leaves.

### How big a request may be

A request starts with a 4-byte length counting the whole packet and a 2-byte code,
so the smallest packet is 6 bytes. **Before the login, a packet may be 6 to 65,536
bytes; after it, 6 to 1,073,741,832** (the largest SD string plus the 10-byte header
the client library allows for). Anything else is not read: the server ends the
session and writes one line to its log (`API packet length ... refused (before
login)` or `(after login)`). The client library never breaks the rule; it matters
to a program that speaks the protocol itself.
## An API session is you

**It runs as your Windows user**, in the account `sduser`, with the same
rights as a session at the keyboard — including `sh` and `OS.EXECUTE` **when
the administrator has turned those switches on** (`set.os.access`; both are off
as delivered, and the API server reads them when it enters the account), which
a remote client can then use. The part of SD that handles the network
encryption runs on a restricted copy of your Windows token that cannot read
your files; the session itself is ordinary and unelevated.

**A session signed in with the global password is a server session**, with
the administrator commands unlocked. See [Managed mode](15-managed-mode.html).

### A session is held to its own account

| | |
|---|---|
| **Allowed** | everything inside its own account, and the shipped system files every account needs — messages, `syscom`, the dictionaries — which the account's VOC names. Ordinary programs are unaffected |
| **Not allowed** | opening, renaming, deleting or listing a file by a pathname outside the account |

**This is the account's `os` switch, the same one a session at the keyboard is held by.**
While it is off, as delivered, a refused `OPEN` takes the `ELSE` branch and
`STATUS()` is 3035, which means *not permitted* rather than *not found*: 3035 is a
containment refusal, not a missing file. Once the administrator has turned the `os`
switch on (`set.os.access os on`) the session has no such limit, which is why the
switch is the thing to weigh — see
[Operating system access](06b-operating-system-access.html).

**If your data lives outside the account**, name the directory in the
**`NETDIRS`** setting in `sd.conf`, separating several with a semicolon.
An entry that is not an absolute path, not an existing directory, a drive root, or that
overlaps the Solo folder or any account is ignored and named in the error log when SD starts
(see [The parameters](16c-the-parameters.html#netdirs-directories-outside-the-account)).
`config('NETDIRS')` does not read it: read the file. SD's password file, its program
catalog and its account register are never reachable from an API session,
and cannot be added to `NETDIRS`. A server session may reach `global.bp.out`.
`SDCLIENT` in `sd.conf` is the further control: any non-zero value stops files being
opened by an API session, and `2` also refuses a subroutine that is not compiled as
callable from a client (`0`, the default, allows everything).

## Request 29, the NET gate and `NETFILES`

**Request 29 opens an `account;file` file for a client** — the one door of SDNet
that is still in the server. **It is shut as delivered, and two things have to be
turned on, by the administrator, before it opens anything:** bit 2 of `NETFILES`
in `sd.conf` (`NETFILES=2`; the default is 0) and **the account's NET gate**,
which `set.net.access` turns on (`set.net.access on` after `admin`; with no
argument it shows where the gate stands). The gate is off as delivered, is kept in field
15 of the register record, is read when the request is asked (so a change reaches
a client that is already signed in), and only the word `yes` means on.

With both on, the request still refuses, with no file number and a line in the
audit trail that says why: when `SDCLIENT` is not 0 (as an ordinary open is);
when the account is named by a **path** (a path names no register record, so
nothing says who may enter it); when it names **another account or SDSYS** (the
one account a session may enter is its own); and when the account is
**suspended**. A refusal of the account rule is *User not allowed in requested
account*, whatever the reason, as for request 3.

**Where the file is.** The request asks the account's `os` switch
(`set.os.access os on`) as `dir`, `openseq`, `openpath` and the OSCOPY verbs do.
A file the account's VOC names with an F-pointer must lead **inside the account's
own folder**, with `..` and links resolved, unless the account has the `os` switch
on: a pointer at an absolute path outside the account, a relative path that climbs
out, a link to the outside, the folder next door whose name merely begins with the
account's own, a system folder, or a dictionary half that leads out (though its data
half is inside), is refused. The refusal is error ELSE with status 3035 (permission),
no file number comes back, and the audit trail gets
`api refused request=29 reason=os gate off user=... account=... path=...`. The switch
is read from the register record when the request is asked, so taking it off reaches a
client that is already signed in. The pointers SD itself puts in the VOC (`syscom`,
`newvoc` and the like) lead out of the account and are refused here too:
a client of request 29 has no use for them. A pointer that opened before and leads out
of an account whose `os` switch is off stops opening: turn the switch on if that was
intended. Solo has no SDSYS exemption, because SDSYS is never a target of the request.
**The `NETFILES` bit alone is not the gate**: without the `os` switch a VOC file in a
folder outside the account is not opened.
## Client libraries

| | |
|---|---|
| 64-bit | `sdclilib.dll`, in `%USERPROFILE%\SDCoreSolo\usr\clients\client64` |
| 32-bit | none; a 32-bit application must be rebuilt as 64-bit |

**The library must speak SCRAM.** A client library that sends the password in clear
is refused by the server — check which one your application is actually
loading. See [Client distribution](10-client-distribution.html).

**Programs using the `!sdclient` class** — SD BASIC reaching another SD over
the API — speak the same login, from this release at both ends.
