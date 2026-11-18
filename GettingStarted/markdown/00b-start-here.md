Title: Start here
Subtitle: What SD Core Solo for Windows is, what this set covers, and what it deliberately leaves out.

You already know MultiValue. This set does not teach it.

**SD Core Solo for Windows is a personal SD for one Windows user.** Everything
it has — programs and data — lives in one folder in that user's profile,
`%USERPROFILE%\SDCoreSolo`. It has **one SD account, `sduser`**, and no way to
make another. Apart from that it has every feature of the multiuser SD Core
for Windows it was made from.

SD Core is a version of SD, with elements found in the main SD version and in
ScarletDME. ScarletDME was a fork of the original GPL release of OpenQM 2.6.6.

**That lineage matters when you go looking for documentation.** Not all the
features of the **commercial** OpenQM 2.6.6 were in the GPL release, and **no
documentation specific to the GPL version was ever released**. So even the
OpenQM documents are not authoritative here.

**The OpenQM 2.6.6 documents can be used as a reference**, but SD Core has
additions, changes and deletions — of features, of structure, of security and
of commands. **These pages cover those changes.**

If you have used OpenQM, or upstream `sdb64`, much of SD Core will still be
familiar: the same data model, the same query processor, the same BASIC. If you
are bringing programs over from another MultiValue database, start with
[Migrating to SD](26-migrating-to-sd.html).

## Two ways to use it, one install

**There is one installer and no mode to choose.** It asks for a **global
password** last, and you can leave it blank:

| | |
|---|---|
| **No global password** | a single-user database on this computer, in the way SQLite is. Nothing else manages it |
| **A global password** | a local database that an **SD Core for Linux** server also manages — one of several computers it looks after. The server signs in with the global password you gave the installer. Such a computer is called **managed** |

**A computer is managed if, and only if, it has a global password.** It is set
at installation and fixed until you reinstall: no command creates it, so a
computer installed without one needs a reinstall to be managed.

Only an SD Core for Linux server manages a Solo computer. What the server can
do today is on [Managed mode](15-managed-mode.html), and it will grow as
management features are added to SD Core for Linux.

## What "used to" means in these pages

These pages describe changes, so **used to**, **no longer** and **now** run all
through them. **The comparison is against the multiuser SD Core for Windows
this product was made from** — what it does differently in this release is in
[Differences from SD Core for Windows W2.0-0](01b-differences-from-sd-core-w2-0-0.html)
— and, behind it, upstream `sdb64`, ScarletDME and OpenQM 2.6.6.

## The pages

**The pages are numbered.** Read them in numerical order the first time; after
that they stand alone.

**A number with a letter after it continues the page before it**, split so
that no page runs longer than a reader will scroll: `01a` continues `01`, and
so on.

| | | |
|---|---|---|
| **00** | Start here | this page — what the set is and what it leaves out |
| **00a** | [Copyright and license](00a-copyright-and-licence.html) | The copyright and the license for this set, in full and in one place |
| **01** | [Installing](01-installation.html) | What the installer asks, what it puts where, and the control file for installing many computers |
| **01a** | [Upgrading and uninstalling](01a-upgrading-and-uninstalling.html) | Installing a new release over an existing one, and taking SD off the machine |
| **01b** | [Differences from SD Core for Windows W2.0-0](01b-differences-from-sd-core-w2-0-0.html) | What Solo leaves out, adds and does differently |
| **01c** | [Installing many computers](01c-installing-many-computers.html) | The control file, what the installer checks when it finishes, when a step fails, and changing any of it afterwards |
| **02** | **[Your first thirty minutes](02-first-run.html)** | **Start here if you just want it working** |
| **03** | [Running SD](03-running-sd.html) | How SD starts, stopping it, and the command line |
| **04** | [Scheduled jobs](04-scheduled-jobs.html) | Running an SD command on a timer |
| **05** | [The account and its passwords](05-account-types.html) | `sduser`, the account password, the administrator and global passwords |
| **06** | [Administrator commands](06-administrator-commands.html) | The commands that need `admin` first |
| **06a** | [Sessions and locks](06a-sessions-and-locks.html) | Who is signed in, what is locked, and clearing it |
| **06a1** | [Task locks and the sessions screen](06a1-task-locks-and-forcing-a-lock-open.html) | Task locks, forcing a lock open, and the sessions and locks screen |
| **06b** | [Operating system access](06b-operating-system-access.html) | Reaching Windows from inside SD |
| **06b1** | [Showing a document with pdf](06b1-showing-a-document-with-pdf.html) | The one bypass of the `os` switch, naming a reader, and the context help |
| **06c** | [Backing up and restoring the account](06c-backup-and-restore.html) | Writing the account to a zip file, putting it back, and recording the system's settings |
| **07** | [Development and file commands](07-programmer-commands.html) | Compiling, editing, and the verbs that maintain files and indexes |
| **08** | [ssh access](08-ssh-access.html) | Reaching SD on this computer over ssh |
| **09** | [API access](09-api-access.html) | The client API, its port, and its login |
| **10** | [Client distribution](10-client-distribution.html) | Which client library an application needs |
| **11** | [Lower case](11-lower-case.html) | Case in commands, file names and record ids |
| **12** | [Security](12-security.html) | What protects the database, and what does not |
| **12a** | [Security and the operating system](12a-security-and-the-operating-system.html) | Reaching the computer from inside SD, and the audit trail |
| **13** | [Other hardening](13-hardening.html) | The global catalog, the logs, and the rest |
| **14** | [Not in SD Core](14-not-in-sd-core.html) | What has been removed, and what to use instead |
| **15** | [Managed mode](15-managed-mode.html) | What an SD Core for Linux server can do to a managed computer |
| **16** | [Configuration](16-configuration.html) | `sd.conf` and its parameters |
| **16a** | [System limits](16a-system-limits.html) | The sizes and counts SD works within |
| **16b** | [Changing a parameter from SD](16b-changing-a-parameter-from-sd.html) | `set.config` and the `E` key of the configuration screen |
| **16c** | [Configuration - the parameters](16c-the-parameters.html) | Every parameter SD accepts, by subject, and the ones that were removed |
| **17** | [The installed scripts](17-the-installed-scripts.html) | The PowerShell scripts in the installed folder |
| **17a** | [The scripts SD runs itself](17a-scripts-sd-runs-itself.html) | The ones the installer and the startup tasks run, which nobody types |
| **17b** | [The scripts a verb calls](17b-scripts-a-verb-calls.html) | The ones a verb runs, and the one SD runs as it starts |
| **18** | [Encryption and the SDEXT interface](18-encryption.html) | What encryption SD has, and the interface behind it |
| **18a** | [Encrypting the data SD keeps on disk](18a-data-at-rest.html) | Putting `%USERPROFILE%\SDCoreSolo` on a BitLocker drive, and what that does and does not protect |
| **19** | [Known limits](19-features-the-developers-could-not-test.html) | What depends on your own environment, and what to check before relying on it |
| **20** | [sdfix, the file check tool](20-sdfix.html) | Checking a hashed file's structure on disk, and recovering space |
| **20a** | [sdfix: what it checks and repairs](20a-sdfix-checks-and-repairs.html) | Every error a check reports, and what `-F` or `-Q` does about it |
| **21** | [sdidx, the index relocation tool](21-sdidx.html) | Where a file's indexes are kept, and moving or deleting them |
| **22** | [sdconv, the byte order convertor](22-sdconv.html) | What it converts (compiled programs only), and what it refuses |
| **23** | [Giving an AI agent an installation of its own](23-giving-an-ai-agent-its-own-installation.html) | Why the wall for an agent has to be a computer, and the recipe |
| **24** | [The AI helper](24-the-ai-helper.html) | Asking an AI model a question from SD: the provider, the key, the gate, and what a helper that runs as you cannot promise |
| **24a** | [Setting up the AI helper](24a-setting-up-the-ai-helper.html) | The AI choice page, the `set.ai` verb, the standing text, the key, a custom provider, the audit trail and troubleshooting |
| **25** | [The MCP door](25-the-mcp-door.html) | Letting an AI agent's MCP client use your account through read-only tools: the two switches (both off), the audit line, and what the door does not add |
| **26** | [Migrating to SD](26-migrating-to-sd.html) | Bringing programs and queries from another MultiValue database: where the compatibility words are set, and the words SD does not accept |
| **26a** | [Option words](26a-migrating-option-words.html) | The words of the `option` command, one line each |
| **26b** | [Mode words](26b-migrating-mode-words.html) | The names `$mode` accepts and the lines of a `$basic.options` record, one line each |

## The five things most likely to surprise you

**1. Every session asks for the account password.** At the keyboard, over ssh
and through the API. Being signed in to Windows is not enough. A command given
on the command line — `sd-solo list voc` — uses a copy of the password kept for you
by Windows, so a script or a scheduled job does not have to type it. See
[The account and its passwords](05-account-types.html).

**2. Administrator commands need `admin` first.** Type `admin` and the
administrator password (or, on a managed computer, the global password) and
they work for the rest of that session. See
[Administrator commands](06-administrator-commands.html).

**3. There is one account and you cannot make another.** It is called
`sduser` on every computer, whatever your Windows name is. There is no
`sdsys` sign-in and no groups to join. The account and grant commands are gone.
See [Differences from SD Core for Windows W2.0-0](01b-differences-from-sd-core-w2-0-0.html).

**4. Commands and names are lower case, completely.** Typing in upper case
still works — SD converts it — but nothing can exist in two casings. See
[Lower case](11-lower-case.html).

**5. An ssh session lands inside SD, and the API login is SCRAM.** ssh to this
computer, on port 4251, as your Windows user with your Windows password, and you
are at SD's prompt, asked for the account password. Clients built against the old cleartext
API login will not connect.
See [ssh access](08-ssh-access.html) and [API access](09-api-access.html).

## What this release is

**WS2.0-0.** Windows only and English only. **It is a hobby project with no
release schedule.**

This set covers installing and running SD Core Solo for Windows, administering
it, managed mode, and what differs from the multiuser SD Core for Windows. The
reference for the language and the command processor is the separate User set.

## Where the source is

**Both repositories are public, and everything in them is open source.** SD is
GPL software — `config gpl` at an `sd-solo` prompt displays the license, and the
installed folder carries it as the file `sdsys\licence`.

| | |
|---|---|
| The server, the client libraries and the installer | <https://github.com/dmontaine/SDCore4WindowsSolo> |
| These pages | <https://github.com/dmontaine/SDCore4WindowsSoloDocs> |

**Neither repository contains a built binary, deliberately** — no `.exe`, no
`.dll`, no object files. A clone builds. That is why installing means building,
and why there is no download of a compiled artifact in the repository itself.

The documentation repository holds the **Markdown only**; the HTML and PDF you
are reading are generated from it and are not stored there.

## Reporting what you find

**Open an issue on the server repository** —
<https://github.com/dmontaine/SDCore4WindowsSolo/issues> — for anything about
SD itself, and on the documentation repository for an error in these pages. If
you are not sure which, the server one is the right guess.

**Issues rather than pull requests.** Both repositories are readable by anyone
and **writable only by the author**, so a change cannot be merged from outside;
a clear issue is worth more than a patch nobody can apply. A patch attached to
an issue is welcome, it just travels that way.

The two things worth reporting in most detail are **anything that behaves
differently from OpenQM and is not described here**, and **anything in these
pages that turns out not to be true of the build you are running**. The second
is as valuable as the first.

**Quote the version as `WS2.0-0`** — the string in the header bar of every page
here, in the installer's file name, in the sign-on banner, and in what
`sd-solo --version` reports. The `WS` says it is Solo for Windows, and that is
the part worth keeping in a report.
