Title: Scheduled jobs
Subtitle: Running an SD command on a timer with Windows Task Scheduler.

**A scheduled job is `sd-solo <command>`, run by Windows Task Scheduler as you.**
There is no permit list to fill in and no administrator rights to give it: a
command on the `sd-solo` command line signs in with the copy of the account
password Windows keeps for you, and runs.

## Setting one up

**1. Write the work as a paragraph** — a `PA` record in the VOC, with the
commands on the lines after the type. Editing the VOC needs `admin` first:

```
:admin
:ed voc my.report
```

**2. Check it runs when you type it**, at the `:` prompt, before putting it on
a timer. Then check it from a PowerShell window, the way the task will run it:

```
sd-solo my.report
```

**3. Create the task** in Windows Task Scheduler. Two fields carry the whole
of it:

| | |
|---|---|
| Program/script | `%USERPROFILE%\SDCoreSolo\usr\bin\sd-solo.exe` |
| Add arguments | `my.report` |

**Run it as your own Windows user**, the one SD is installed for. Another
Windows user has no copy of the password and no `SDCoreSolo` of its own.

**Do not tick "Run with highest privileges".** It is not needed, and SD drops
the rights anyway.

## Signed in or not

| | |
|---|---|
| **Run only when user is logged on** | runs in your signed-in session. The kept password works as it does in a PowerShell window |
| **Run whether user is logged on or not**, with your Windows password stored by Task Scheduler | runs when you are signed out. Expected to work the same way, because Windows signs the task in with your password |
| **Run whether user is logged on or not**, with **Do not store password** ticked | **not measured.** Windows signs such a task in without your password, and the kept copy is protected by it, so it may not open. Use one of the two above, or supply the password on the input (below) |

## Supplying the password on the input

**If the kept copy does not open, or no longer matches** — you changed the
password without `set.password` — a command whose input is piped takes the
first line of that input as the account password, once:

```
Get-Content C:\jobs\sd-password.txt | sd-solo my.report
```

**That file holds the password in clear text.** Keep it where only your Windows
user can read it, or prefer the kept copy.

## When it does not run

| | |
|---|---|
| *A command given on the sd command line needs the account password on its input* | the kept copy did not work and the command was run at a terminal, with nothing piped in |
| *Wrong password* | the piped password was wrong |
| the task reports success and nothing happened | look in `%USERPROFILE%\SDCoreSolo\sdsys\audit`: a sign-in with the kept copy is recorded as `login password account=SDUSER via=stored` |

**A job's output goes nowhere** unless the paragraph sends it somewhere — a
file, or a printer. Task Scheduler shows only the exit code.
