Title: System limits
Subtitle: The sizes, counts and depths compiled into SD Core Solo for Windows, and which of them a site can change.

Two kinds of number appear on this page and they behave differently.

A **compiled limit** is fixed in the build. Changing it means rebuilding SD, and
several would change on-disk file formats, so in practice they are ceilings.

A **configured limit** is a value in `sd.conf` with a default. A site can raise or
lower it, within a range the compiled limits set. Those are covered in full on
[Configuration](16-configuration.html); this page gives the number in force and
says which parameter controls it.

> Every figure here was read from the sources that define it, or counted directly
> from this tree's own shipped directories, not from documentation of another
> product or port.

## Data

| | | |
|---|---|---|
| Maximum string or record size | **1,073,741,822** bytes | compiled |
| Maximum record id length | **63** by default, up to **255** | `MAXIDLEN` |
| Fields per record | limited only by record size | — |
| Values per field, subvalues per value | limited only by record size | — |

The record size limit is just under 1 GB rather than 2 GB. It is the maximum
size of any single string SD will build, so it applies to a record, to a
concatenation, and to anything a program assembles in memory.

**A record id may be longer than 63 characters only if `MAXIDLEN` is raised**,
and the range starts at 63 rather than 0 — a lower value is refused at start-up
rather than accepted. The ceiling of 255 is compiled, and the comment in the
source notes that raising it would require major file-format changes.

## Files

| | | |
|---|---|---|
| File types | **two** — directory and dynamic | compiled |
| Group size for a dynamic file | **4 KB** by default (2 KB until 12 Nov 2026), maximum **8** | `GRPSIZE` |
| Open files across all sessions | **80** | `NUMFILES` |
| Maximum pathname length | **255** | compiled |
| Sort keys in one query | **32** | compiled |

Group size is expressed in 1 KB units, so the compiled maximum of 8 is 8 KB.
**The 255-character pathname limit includes the path of the `SDCoreSolo` folder**:
a long path to it leaves less room for the paths SD builds inside it.

**A sort that outgrows `SORTMEM` is written to work files, and an item or a key may be of any
length SD can hold.** The lengths in a work file were 16 bits, so an item or a key of 32 KB or
more wrapped when a sort spilled to disk and the result was wrong; they are 32 bits now. The
count of a sort that returned no data was also one short when the last record ended the file.

**A split of a dynamic file stops before it changes the file when the disk cannot take it.** A
dynamic file grows by splitting its groups. The split writes the new, empty group first and only
then advances the file's modulus; it does nothing if that write cannot be made, or if the file
system holds less than 1 MB, and writes a line to the log. Before, a full disk could leave the
modulus advanced with no group behind it, and the records that hashed there could not be found.
**A split that fails after the new group has begun to be used still cannot be undone**: it logs
the failure and writes the file's header, and the file needs a file check (`sdfix`).

## Sessions and locks

| | | |
|---|---|---|
| Concurrent sessions | **20** | `NUMUSERS` |
| Record locks across all sessions | **100** | `NUMLOCKS` |
| File locks | one per file | compiled |
| SD account name | **32** characters | compiled (`MAX_ACCOUNT_NAME_LEN`) |

`NUMUSERS` sizes the user table in shared memory, so it is read once when SD
starts and cannot be changed in a running system. **On a Solo computer every
session, phantom and API connection is one of the 20.**

## Programs

| | | |
|---|---|---|
| Subroutine call depth | **10,000** | `MAXCALL`, range 10 to 1,000,000 |
| Cataloged name length | **63** characters | compiled |
| Matrix dimensions | **two** | compiled |
| Compiled programs held in memory | no limit by default | `OBJECTS` |
| Memory those programs may occupy | no limit by default | `OBJMEM`, in KB |
| Path of a program named to `RUN` | **255** characters | compiled |

## Select lists

| | | |
|---|---|---|
| Numbered select lists | **13**, numbered 0 to 12 | compiled |
| Items in a select list | limited by disk | — |

Thirteen is the number a session has available at once, not a limit on how many
lists can be saved. `save.list` writes to a file and is limited only by space.

## Terminals

| | | |
|---|---|---|
| Terminal definitions shipped | **26** source definitions | — |
| Terminal names they compile to | **37** | — |
| Default page size | **140 × 43** (it was 120 × 36 before 18 October 2026) | — |
| Minimum page size | 10 lines × 20 columns | compiled |
| Default printer page | **80 × 66** | `LPTRWIDE`, `LPTRHIGH` |

The extra terminal names are aliases and variants that share a definition with
their base name.

## What the account gets

**There is one account and one VOC.** The account gets the whole of `newvoc`;
SDSYS's own VOC is built from the whole of `voc_template`.

| | VOC records |
|---|---|
| Your account (`newvoc`) | **440** |
| SDSYS (`voc_template`) | **449** |

**These are record counts, not verb counts** — a VOC record may be a verb, a
keyword, a file pointer, a sentence or a paragraph, and only some are verbs.
Counted directly from the shipped directories. The account and grant commands
are not among them; see
[Differences from multiuser SD Core for Windows W1.1-1](01b-differences-from-multiuser-w1-1-1.html).

## Configuration

| | |
|---|---|
| Parameters accepted in `sd.conf` | **59** |
| Parameters the `config` verb displays | **56** |
| Parameters readable with `config()` | **57** |
| Parameters settable for the current session | **28** |

The verb displays `DEBUG`, `FDS` and `PHANTOMS` only when they are set, and
`TIMEZONE`, which is not a file parameter. The parameters removed in 2.0-0, and
what happens to a kept `sd.conf` that still names one, are on
[Configuration](16-configuration.html). `NETFILES` is live: its bit 2 opens API
request 29, behind the NET gate (see the same page).
