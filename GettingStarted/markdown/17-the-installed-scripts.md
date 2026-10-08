Title: The Installed Scripts
Subtitle: The fifteen PowerShell scripts installed beside SD - the execution policy they need, what their exit codes mean, and the two you may need to run yourself.

SD's installer does part of its work in PowerShell rather than inside the
installer script, and **it leaves those scripts on the computer**. They are in
`%USERPROFILE%\SDCoreSolo`, beside `sd.conf`, and they are there for two
reasons: so that a step which failed during the installation can be looked up
and understood, and so that a choice made in the wizard can be changed
afterwards.

**They are Windows scripts, not SD verbs.** Nothing here is typed at an `sd-solo`
prompt. The two you may run yourself need a PowerShell prompt started with
**Run as administrator**, because they change the firewall. The rest are run by
the installer or by SD.

*Italics* mark something you supply, **bold** a word typed as it stands, and
braces an optional part.

## What is here and what is not

**Fifteen scripts ship.**

| | |
|---|---|
| `api-firewall.ps1` | who may reach the API port — [below](#who-may-reach-the-api-from-other-computers) |
| `solo-ssh-firewall.ps1` | who may reach Solo's ssh port, 4251 — [below](#who-may-reach-ssh-from-other-computers) |
| `solo-api-listener.ps1` | switches the API listener on or off in `sd.conf` — [API access](09-api-access.html) |
| `solo-machine.ps1` | the installer's one administrator step |
| `solo-setup.ps1` | the installer's steps that run as you |
| `solo-sshd.ps1` | sets up and looks after Solo's own ssh server |
| `solo-start.ps1` | what the sign-in startup task runs, for an account Windows will not give a start-up task |
| `internal-marker.ps1` | a helper the installer's steps load |
| `sd-path.ps1` | what `append.sd.path` runs |
| `micro-home.ps1` | what the `micro` editor verb runs |
| `solo-sshkey.ps1` | what the API's ssh key request runs, on a managed computer |
| `sd-account-archive.ps1` | the file work of `backup.account` and `restore.account` |
| `sd-settings-os.ps1` | the Windows sections of `settings.report` |
| `sd-backupdir.ps1` | what `set.backup.directory` runs |
| `solo-restore-swap.ps1` | puts a restore in place while SD starts |

The rest are described on
[The Scripts SD Runs For Itself](17a-scripts-sd-runs-itself.html) and
[The Scripts a Verb Calls](17b-scripts-a-verb-calls.html).

Everything else in the project's `gplbld` directory — the verifiers, the
probes, the build and test cycle — is development tooling and **is deliberately
not installed**. If you have read about `cycle.ps1`, `assert-current.ps1` or a
`verify-` script and cannot find it, that is why: they compare an install
against the source tree it was built from, and they are destructive.

## PowerShell execution policy — you do not need to change it

Windows will not run a PowerShell script unless its **execution policy**
allows it. On Windows desktop editions the default is `Restricted`, which
allows no script at all; on Windows Server it is `RemoteSigned`, which allows
a script written on the computer itself.

**SD does not depend on that setting, and you should leave it alone.** The
scripts are launched with an explicit `-ExecutionPolicy Bypass` on their own
command line — by the installer, and by the SD verbs that call one (through the
`SH1` line of `sd.conf`). That switch applies to **that one PowerShell process,
for that one script**. It changes nothing on the computer and nothing about any
other script.

**What happens if you change it anyway:**

| what you do | what happens to SD |
|---|---|
| **Tighten it** to `Restricted` or `AllSigned`, for the computer or for your own account | **Nothing. SD carries on working.** The switch SD passes takes precedence over both of those settings |
| **Loosen it** to `RemoteSigned`, `Unrestricted` or `Bypass` | **Nothing — and you have gained nothing.** SD was already unaffected. You have made the computer more permissive for every *other* script on it, which is a real cost for no benefit |
| **Set it through Group Policy** | **This one stops SD.** See below |

**One thing it does not cover.** The `sh` verb opens an ordinary PowerShell
prompt for you, and that prompt gets **no** such switch — it runs under
whatever policy your computer sets. That is deliberate: SD lifts the
restriction for the scripts it installed itself, and never for a shell you
type into. **The scripts you run by hand from the list below are in the same
position**: you start them from a prompt of your own, so they need
`-ExecutionPolicy Bypass` on the command line, as every command on this page
shows.

### Group Policy is the exception, and it is the one to know about

A **Group Policy** setting — *Turn on Script Execution*, under
`Computer Configuration` or `User Configuration` → `Administrative Templates`
→ `Windows Components` → `Windows PowerShell` — **outranks the switch SD
passes.** Group Policy sits above the per-process setting in PowerShell's
order of precedence, so on a computer where a policy sets the execution policy,
SD cannot override it.

**On a domain-joined or otherwise managed computer, check this before
installing.** In an ordinary PowerShell prompt:

```
Get-ExecutionPolicy -List
```

If the `MachinePolicy` or `UserPolicy` row says anything other than
`Undefined`, a policy is in force. **`RemoteSigned`, `Unrestricted` or
`Bypass` there is fine** — SD's scripts are written on the computer by the
installer, not downloaded. **A policy of `Restricted` or `AllSigned` will stop
the installer's steps and the verbs that call a script**, and the symptom is
the *"running scripts is disabled on this system"* message from the installer,
`append.sd.path` or an editor verb. **That needs your Windows administrator to
relax the policy.** SD has no way around it, deliberately: a program that could
defeat Group Policy would be a worse thing to have installed than an
inconvenience.

## The exit codes are a convention

The scripts that exit with a code print what they did and then use the same
three-value convention:

| | |
|---|---|
| **0** | it is done. That includes *"it was already done"* - the scripts are written to be run twice |
| **1** | it failed, and the line above the exit says why |
| **2** | **it neither did the work nor failed.** It refused, or it could not run |

**2 is the one worth reading.** It is not an error code; it means the script
declined to act and is telling you the condition. Each script below says what
its own 2 means, because they differ. `internal-marker.ps1` exits with nothing:
it only defines two functions. `micro-home.ps1` has no 2: it prints one line
and exits 0, or exits 1 without it.

## The ones you may need to run

These change a decision the installer made. Every command below is complete as
written; run it from an **elevated** PowerShell prompt — creating or changing a
firewall rule is a change to the computer, not to you.

### Who may reach the API from other computers

```
powershell -ExecutionPolicy Bypass -File "%USERPROFILE%\SDCoreSolo\api-firewall.ps1" -Show
powershell -ExecutionPolicy Bypass -File "%USERPROFILE%\SDCoreSolo\api-firewall.ps1" -Open
powershell -ExecutionPolicy Bypass -File "%USERPROFILE%\SDCoreSolo\api-firewall.ps1" -Restrict
powershell -ExecutionPolicy Bypass -File "%USERPROFILE%\SDCoreSolo\api-firewall.ps1" -Remove
```

Exit **0** applied, **1** failed, **2** refused. `-Show` changes nothing and
needs no elevation. `-Open` allows any address, `-Restrict` this computer only;
The port is always 4249; the script takes no port. `-Retarget` moves a rule an
older release left on port 4243 to 4249 under the name `SD-Solo-API-In-TCP`, and
keeps who may reach it as it was (an upgrade runs it). **This script owns its
rule** — it created it, and `-Remove` takes it away.

**It says who may reach the port, not whether there is one.** Whether anything
is listening is `APIPORT` in `sd.conf` — see
[Configuration](16-configuration.html) and [API access](09-api-access.html).
A rule for a port nothing has opened admits nothing.

### Who may reach ssh from other computers

```
powershell -ExecutionPolicy Bypass -File "%USERPROFILE%\SDCoreSolo\solo-ssh-firewall.ps1" -Show
powershell -ExecutionPolicy Bypass -File "%USERPROFILE%\SDCoreSolo\solo-ssh-firewall.ps1" -Restrict
powershell -ExecutionPolicy Bypass -File "%USERPROFILE%\SDCoreSolo\solo-ssh-firewall.ps1" -Open
powershell -ExecutionPolicy Bypass -File "%USERPROFILE%\SDCoreSolo\solo-ssh-firewall.ps1" -Remove
```

Exit **0** applied, **1** failed, **2** refused (for example, not elevated).
`-Show` changes nothing and needs no elevation. `-Open` allows any address,
`-Restrict` this computer only. The port is always 4251; the script takes no
port.

**This script owns its rule**, `SD-Solo-SSH-In-TCP` — it made it, and `-Remove`
takes it away. **It never touches Windows' own ssh rule for port 22**
(`OpenSSH-Server-In-TCP`): that is Microsoft's, and Solo's ssh server does not
use port 22. See [ssh access](08-ssh-access.html).

## See also

[Installing](01-installation.html) covers what the installer asks and what it
does with each answer. [The Scripts SD Runs For Itself](17a-scripts-sd-runs-itself.html)
and [The Scripts a Verb Calls](17b-scripts-a-verb-calls.html) cover the other
thirteen.
