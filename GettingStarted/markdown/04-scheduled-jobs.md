Title: Scheduled jobs
Subtitle: Running an SD command on a schedule: SD's own jobs (set.job), or a Task Scheduler task of your own.

There are two ways: **SD's own scheduled jobs**, which SD lists, starts and
stops for you and which run whether or not you are signed in, and **a task you
make yourself in Windows Task Scheduler**. Both run `sd-solo.exe` as your own
Windows user.

## SD's own scheduled jobs

**Off as delivered.** The administrator turns them on once:

```
:admin
:set.jobs.access on
```

(or on the gates page, `tool.gates`, the row *Scheduled jobs*). While they are
off, no job can be made or started, and no job runs; `set.jobs.access off`
stops them all. The jobs stay defined until you delete them.

**Make a job** with its name, its schedule and the command, as you would type
the command at the prompt:

```
:set.job add nightly daily 02:30 my.report
Password:
Job nightly is scheduled: daily 02:30
```

**It asks for your account password, every time** you make or change a job,
even in a session that has signed in; it cannot be done from a script or a
one-shot command. So a job runs only a command that you, knowing the password,
gave it.

| Command | Does |
|---|---|
| `set.job` | lists your jobs, each with its schedule, its command, when it runs next and when it last ran |
| `set.job add` *name* *schedule* *command* | makes a job (refused if one of that name exists) |
| `set.job change` *name* *schedule* *command* | changes one |
| `set.job delete` *name* | removes one |
| `set.job run` *name* | starts one now, through the job clock |
| `tool.jobs` | the same on a full-screen page: N, C, D and R; its form has a masked field for the password |

*name* is 1 to 32 characters: a lower-case letter first, then lower-case
letters, digits and dashes. *schedule* is one of `every 5 minutes` (or 10, 15,
20, 30), `hourly`, `daily hh:mm`, `weekly mon hh:mm` (mon to sun), `monthly 1
hh:mm` (day 1 to 28), in the computer's time, with two digits for the hour
(`09:30`). A run missed while the computer was off, or more than five minutes
late, is not made up.

**What a job is.** Its definition in SD
(`%USERPROFILE%\SDCoreSolo\sdsys\$jobs`): the schedule, the command, and when
it last ran. **The job clock** runs them: one Windows Task Scheduler task, in
the folder `\SD Core Solo Jobs`, named `sd-solo-jobs-clock`, which the
installer registers. Every minute it runs
`%USERPROFILE%\SDCoreSolo\usr\bin\sd-solo.exe run.jobs.due` as your Windows
user, **whether you are signed in or not, without a stored password** (Task
Scheduler's "Do not store password"). `run.jobs.due` starts each job whose
time has come through `run.job`, which reads the command from the job's
definition and runs it; SD keeps the command, never the task. The audit trail
(`%USERPROFILE%\SDCoreSolo\sdsys\audit`) has a `run.job start` and a
`run.job end` line for every run, the end with the command's status. A job's
output goes nowhere unless its command sends it somewhere. Jobs need SD
running, which its startup task does at boot.

**Why one clock and not a task for each job.** Only you, elevated, can register
a task that runs as you while you are signed out: Windows refuses everyone
else, even SYSTEM. SD's sessions are not elevated, so they cannot register a
task when you make a job. The installer can, once, when you approve its
prompt, so it registers the clock, and SD itself decides which job is due.
**Approve the installer's prompt as yourself.** If another administrator
approves it, the clock cannot be registered for you, the installer's last
page says *SCHEDULED JOBS : NOT running*, and no job runs until you install
again and approve the prompt yourself. The folder `\SD Core Solo Jobs` can be
changed only by SYSTEM and Administrators; the clock lets you read and start
it, not change it.

**While jobs are off** the clock still ticks every minute, and stops before it
signs in: nothing is run and nothing is written to the audit trail. While they
are on, every tick signs in, and the audit trail has a sign-in line for it each
minute.

**How the clock signs in while you are signed out.** `run.jobs.due` and
`run.job` normally sign in with the copy of your account password Windows
keeps for you (as delivered, only `run.job`, `run.jobs.due` and the MCP door's
`mcp.call` may use it). While you are signed out, Windows will not open that
copy for a task that has no stored password. So `run.jobs.due`, and `run.job`
given exactly one job name, and nothing else, sign in instead by being your
own Windows user: SD checks that the process runs as the Windows user who owns
SD's password folder (`sdsys\$cred`), and that nobody else may use that
folder. `run.job` then runs only a job that is defined, and only while jobs are
on.

**What that means for security, in plain words.** It adds no way in. While you
are signed in, any program of yours can already ask Windows to open the kept
copy of the password, and sign in with it. While you are signed
out, a program can run as you only from a task registered with administrator
rights. What a job runs was fixed by someone who typed the account
password when the job was made. The limit is the same as for the doors: the
definitions (`%USERPROFILE%\SDCoreSolo\sdsys\$jobs`) and the password records
are files in your own folder, so a program that already runs as you could
change them; this is not a wall against you, and it is not meant to be.

**Uninstalling** removes the job clock. When you keep your data, the job
definitions are kept with it, and a new install that reloads the data runs
them again. The register's switch is not kept: turn jobs on again with
`set.jobs.access on`.

## A task of your own

**A task of your own is `sd-solo <command>`, run by Windows Task Scheduler as you.**
There is no permit list to fill in and no administrator rights to give it. **As
delivered it needs the account password on its input** (below): the kept copy of
the password signs in only SD's own jobs and the MCP door's calls. To run such a
command on a timer, make it one of SD's jobs with `set.job` (above), pipe the
password in, or have the administrator say `set.stored.access on`, after which a
command on the `sd-solo` command line signs in with the kept copy again, as in
earlier releases.

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
| **Run whether user is logged on or not**, with **Do not store password** ticked | **does not sign in while you are signed out**: Windows signs such a task in without your password, and the kept copy, which your password protects, does not open. Use one of the two above, supply the password on the input (below), or use SD's own jobs, which are made this way and sign in by your Windows identity |

## Supplying the password on the input

**If the kept copy may not be used (as delivered), does not open, or no longer
matches** — you changed the password without `set.password` — a command whose
input is piped takes the first line of that input as the account password, once:

```
Get-Content C:\jobs\sd-password.txt | sd-solo my.report
```

**That file holds the password in clear text.** Keep it where only your Windows
user can read it, or prefer the kept copy.

## When it does not run

| | |
|---|---|
| *A command given on the sd command line needs the account password on its input* | the kept copy did not work and the command was run at a terminal, with nothing piped in |
| *The kept password signs in only SD's scheduled jobs and MCP calls* | the task ran `sd-solo <command>` itself: make it a job with `set.job`, pipe the password in as above, or have the administrator say `set.stored.access on` |
| *Wrong password* | the piped password was wrong |
| the task reports success and nothing happened | look in `%USERPROFILE%\SDCoreSolo\sdsys\audit`: a sign-in with the kept copy is recorded as `login password account=sduser via=stored` |

**A job's output goes nowhere** unless the paragraph sends it somewhere — a
file, or a printer. Task Scheduler shows only the exit code.
