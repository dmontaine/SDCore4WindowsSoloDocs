Title: Changes in WS2.0-0 (SD Core Solo for Windows)
Subtitle: What was added, changed and removed since WS1.1-3 (10 October 2026), by section and in alphabetical order.
Columns: 23 11 66

**This is the list of what changed between WS1.1-3 (10 October 2026) and WS2.0-0 (called WS1.2-0 until 10 November 2026).** It is kept up to date as the
release is built. It was started on 25 October 2026 from the list for SD Core for Windows (both carry the same
openQM items) and the release changelog (`sdsys/changelog`, section "WS2.0-0 - SD Core Solo for Windows, in progress"),
which has the full story of each change: what it does, why, and what it does not do. This document is the short form.
**Nothing in it has been run on a real Windows installation yet.**

## How to read it

- Every row is one thing that was **added**, **changed** or **removed**. A fix is a change.
- The rows are grouped into sections, and **within each section they are in alphabetical order**.
- A row is one or two lines. The User and Administrator manuals have the detail; so does the changelog.
- Names are in lower case as SD writes them. `$mode` words, options and BASIC keywords are written the same way.
- The shell and the operating system are **off as delivered**; the administrator turns each on with `set.os.access` and can turn it off again.
- A comment at the end of a row, `<!--s` then item numbers then `-->`, names the spreadsheet items the row covers. It is
  for whoever maintains this document and does not show on the page.
- SD Core Solo has one account, so the per-account gates, the shared account and the account verbs of SD Core for Windows are not here.

## Read this first: changes that alter what an existing system does

| Name | Change | What |
|---|---|---|
| `@who` | Changed | was the account name in upper case; now lower case, as on SD Core Solo for Linux. A program that compares it with an upper-case name must compare without regard to case |
| A `select` or `list` with `by` | Changed | records with equal `by` values came out in the order they sat in the file; now in record-id order (also under `by.dsnd`), as `sort` and the other S verbs always had it |
| AccuTerm and SDTerm | Removed | were supported; use terminal type `xterm` or `vt100` |
| CSV and DELIMITER reports | Changed | a multivalued field was one cell with value marks inside it; now each value is a line of its own |
| Dictionary divide by zero | Changed | stopped the report; now gives zero |
| `fileinfo(f, 1021)` | Changed | gave text such as `12 (don), 14 (ann)`; now the user numbers of the other sessions that have the file open, and the text form is key **1100** |
| `intprec` and `sortmem` | Changed | were 13 and 1024; now **11** and **4096** (set the old values in `sd.conf` if you need them) |
| Removed features | Removed | words and modes that existed to imitate other products (55 `option` words, 35 `$mode` words, and the second spellings and forms of statements, clauses, functions, conversions, TCL verbs and query keywords, each listed in [REMOVED](#removed)), and other things, `call *name` among them: see [REMOVED](#removed) |
| `select file field` | Changed | a field not otherwise used was ignored; now it is `saving multivalued field`, so the list holds the values |
| Shell and operating system access | Changed | were always on for the one account; now the SH switch (the `sh` verb) and the OS switch (`os.execute`, the Python helper, the editors, and files outside the account directory by path) are **off** as delivered; the administrator turns them on with `set.os.access` |
| The spelling of what SD prints | Changed | are in US spelling (catalog, color, canceled, recognized, license, behavior); a program that matches a message by its old wording must be changed; names of verbs, keywords and files are as they were |
| The terminal's size | Changed | every session started at the terminal type's 120 by 36; now SD asks the terminal how big it is at the start and when it is resized (`term w,h` still sets it by hand) |
| The VOC and the server's files | Changed | only some of the commands that change a file you name asked for `admin` (or the global password for `gcat`, `global.bp.out` and `solo.policy`); now they all do |
| `when field` with no operator | Changed | kept the null values; now keeps the values that are **not** null |
| Where a session starts | Changed | a session on a terminal started at the command prompt; now it opens the home screen (`admin`, then `set.start.default terminal`, gives the prompt back) |

## Sections

**Built from the openQM list** (each row is an item on the feature spreadsheet): [ADMIN](#admin) · [API](#api) ·
[BASIC](#basic) · [DATA](#data) · [FILES](#files) · [QUERY](#query) · [SYSTEM](#system) · [TCL](#tcl) · [TOOLS](#tools).

**[REMOVED](#removed)**: what SD no longer has, and what to write instead.

**[New Features Unique to SD Core](#new-features-unique-to-sd-core)**, not part of openQM and not on the spreadsheet:
[Administration](#administration) · [AI](#ai) · [API and client library](#api-and-client-library) · [Encryption](#encryption) ·
[File transfer](#file-transfer) · [Files](#files-1) · [Installer](#installer) · [Screen library and terminals](#screen-library-and-terminals) · [SDNet](#sdnet) ·
[Security](#security) · [System](#system-1).

**[NO CHANGE](#no-change)**: openQM items that needed nothing.

## ADMIN

Administrator commands from the openQM list.

| Name | Change | What |
|---|---|---|
| `inhibit.login` | Added | administrator only: stops new sessions while maintenance is done; a killed backup no longer locks everybody out <!--s 387 g46--> |
| `lock.tracer` | Added | administrator only (`admin`): logs every lock taken and released, with program and line, to the file `locktrace` in the system folder; see Getting Started 06a |
| `logout` | Changed | `logout n group` also ends the phantoms n started; `logout` of a session whose process has gone clears the entry and its locks <!--s 386--> |
| `newvoc.mods` | Changed | applied to SDSYS's own VOC when SD is installed, as it already was to new accounts and by `update.accounts` <!--s 501--> |
| `set.recovery` | Added | administrator only: the IDE keeps a recovery copy of text you have not saved (`set.recovery on`), offered back when the record is opened again; off as delivered; Getting Started page 06, Administrator commands |

## API

The server side of the API, the client library and the language headers.

| Name | Change | What |
|---|---|---|
| client library headers | Added | all six language bindings are installed in `sdsys\syscom`: C, Gambas, PureBasic, Free Pascal, Python, and now C# (`sdclient.cs`) and PHP (`sdclient.php`); neither of the last two has been run against a runtime |
| client library: dictionary | Added | `sdevaluate`, `sdevalconv`, `sdiconv`, `sdoconv`, `sdiconvs`, `sdoconvs` |
| client library: files | Added | `sdopenseq`, `sdreadseq`, `sdreadblk`, `sdwriteseq`, `sdwriteblk`, `sdseek`, `sdweofseq`; `sdwriteseqkey` and `sdwriteuseqkey`, which only a session signed in with the global password may send to a Solo server |
| client library: more functions | Added | `sdrevision`, `sdgetvar`, `sdsystem`, `sdindices`, `sdtrans`, `sdrtrans`, `sdtxn` (mode 4 commits a nested transaction non-durably) |
| client library: threads | Added | each thread has its own session, transfer buffer and arguments; several threads may use the library |
| `master.login` | Changed | the paragraph runs for API connections as for interactive and phantom sessions <!--s 170--> |
| The ssh key request | Changed | the API's ssh key request is 72 (it was 49), and request 49 is the server's revision, as in SD Core; a script written for WS1.1-3 that sends 49 to install a key must send 72 |

## BASIC

Statements, functions, `$mode` words and compiler directives.

| Name | Change | What |
|---|---|---|
| `~=` | Added | comparison that ignores case <!--s 430--> |
| `account.path()`, `vocpath()` | Added | the pathname of an account and of a file named in the VOC, told only where the account may open it <!--s g107 g193 86--> |
| `arg.present()`, `reusing()` | Added | whether the caller passed an argument, and whether with `reuse()` <!--s g108 g169--> |
| `$basic.options` | Changed | a `mode` line takes a list and a minus turns a mode off; an `include` line names records or files that every program includes; a `search` line names the files an `$include` with no file name looks in <!--s 406 407 290--> |
| `bool()` and friends | Added | `bool()`, `is.null()`, `numeric()`: Boolean, SQL-null and numeric conversions <!--s g109 g144 g161 451--> |
| `@build.time`, `@line` | Added | the time the program was compiled and the source line it stands on <!--s 506 211--> |
| `casing on` and `off` | Added | case sensitivity of string comparisons for the rest of the program <!--s g111 g274--> |
| `ceil()` and `floor()` | Added | `ceil()`, `floor()`, `roundup()`, `rounddown()`: whole numbers above and below, and rounding in steps <!--s g112 g129 g174 g175--> |
| `child()`, `signal()` | Added | whether a user number is a running phantom of this process; raises a signal in another process <!--s g113 g182 301--> |
| `class ... inherits as` | Added | the inherited class's object variable can be named; an empty `()` is accepted after a method name <!--s 374 237--> |
| `class` initial values | Added | a `public` or `private` variable may start with a value; `inherits` passes constants to the inherited class <!--s 356 565--> |
| `class` lookup | Changed | a class with 32 names or more is searched through an index, many times faster <!--s 13--> |
| `class`: `restricted` | Added | a name for the class and the classes that inherit it, and no one else <!--s g168--> |
| `class`: `run` | Added | `run` of a class module or its catalogue name makes the object, runs its `main` and lets it go <!--s 484--> |
| `clearcommon name` | Added | sets the variables of the named common block to zero <!--s 52--> |
| `compare()` | Changed | an `i` after `l` or `r` in the justification compares ignoring case; `compares()` does it over two arrays <!--s 307 g114--> |
| `convert()` | Changed | `mcn-`, `mc/n-`, `mcs`, `mcp` with a quoted replacement character, `mt` with `m`, `md` options in any order <!--s 23 183 198 298 488--> |
| default argument values | Added | `subroutine name(a, b = 7)`: a value for an argument the caller leaves out (a number, a string, `@true` or `@false`); the program needs `var.args` |
| `digest()` | Added | the md5, sha1, sha256, sha384 or sha512 digest of a string or a file <!--s 63 g118--> |
| `dir()` | Changed | includes the size of each file; sets `status()` (0, or why the directory could not be read) and lists only files and directories <!--s 366--> |
| `fix()` | Added | a number with a fixed number of decimals <!--s 472--> |
| exceptions | Added | `try`, `catch`, `throw`, `caught()` and `@exception`; many run time errors are caught under openQM's `sys.` names; `dumping` and `saving.stack` <!--s 353 354 355 295 309--> |
| `execute ... silent`, `phantom ... silent` and `os.execute ... silent` | Added | `execute`, `phantom` and `os.execute` with `silent` run the command and throw its output away; not with `capturing` |
| `execute ... stacklist` | Added | keeps the default select list and puts it back when the command returns <!--s 61--> |
| `execute.list` | Added | runs commands for each id of list 0 in turn <!--s g38--> |
| `fcontrol()` | Added | the openQM function: next sequential key, valid id, minimum modulus, line terminator modes, forced writes <!--s 452 102--> |
| `field()` and `vslice()` | Changed | take an asterisk for the number of elements and a subvalue slice <!--s 212 132--> |
| `flush.dh.cache` | Added | makes every process close the dynamic files it keeps cached <!--s 83 123--> |
| `for` with a list | Added | a loop over a list of values, and over the elements of a delimited string, with a delimiter <!--s 84 300--> |
| `get var from device` | Added | reads from a FIFO, port, character device or `openseq` file a character at a time, with a timeout <!--s 583--> |
| `getlocks()`, `testlock()` | Added | information about file and record locks; the owner of a task lock <!--s 508 42--> |
| `getpu()` | Added | the `pu$pathname` key (`pu$form.queue` was added and removed again, see REMOVED) <!--s 18 214--> |
| `groupstore` and `revremove` | Added | `groupstore`, `revremove`, `setrem`, `rmvd()`: statements and functions over delimited strings <!--s g136 g170 g172--> |
| `$ifdef`, `$ifndef` | Changed | accept `and` and `or` between token names <!--s 219--> |
| `inmat()` | Changed | after `matches` or `matchfield()` it is the number of the pattern that matched; after `matbuild` and `matwrite` it is the element count; a collection array is accepted <!--s 315 454 591--> |
| `input ... fill` | Added | shows a fill character in every empty cell of the field <!--s 389--> |
| `input ... no.echo` | Added | shows nothing as the user types <!--s 168 363--> |
| `input ... raw` | Added | reads exactly the number of characters given, as they arrive <!--s 548--> |
| `inputif` | Added | reads a line only if one is already there, else takes `else` <!--s 584--> |
| `inputnull`, `inputtrap` | Added | a single character typed alone makes the answer null, or sends the program to a label <!--s 408--> |
| `last()` and more | Added | `last()`, `lasts()`, `nv()`, `ns()`: the final element of a delimited string; value and subvalue position numbers (`left()` and `right()` were added and removed again, see REMOVED) <!--s g150 g151 g171 g162--> |
| `locate ... by` | Added | `c` forces a case-sensitive search and `n` a case-insensitive one; `lan` is ascending; fixed: with `ar` a number is placed before a list's single text item <!--s 419--> |
| `matchess` and `trimws` | Added | `matchess`, `matchfields`, `trimws`, `trimw()`: `matches`, `matchfield` and `trimw` over a dynamic array; `trimw` trims white space <!--s 302 g152 g153 g189--> |
| matrix arguments | Changed | a matrix passed as an argument needs no `dim` in the receiving subroutine; `if mat a = mat b` compares two matrices; the compiler no longer reports an undimensioned argument <!--s 44 111 519--> |
| `matstr()` | Added | converts a two dimensional matrix to a dynamic array <!--s 509--> |
| `matwritecsv` | Added | one CSV line of every element of a matrix <!--s g154--> |
| `mergelist()` | Added | merges two lists by a mode; a mode for lists not in order; fixed for an empty list <!--s 420 539 566 g155--> |
| `$mode debug.equates` | Added | keeps `equate` and `$define` tokens in the object so the debugger can show, set and watch them <!--s g218 246 239--> |
| `$mode explicit` | Added | a variable must be named in a `dim` statement before it is used <!--s g220 234--> |
| `$mode` lists | Changed | `$mode` takes several names separated by commas, each with an optional minus <!--s 105--> |
| `$mode non.durable.txn` | Added | a committed nested transaction writes nothing until the outer one commits <!--s g221 442--> |
| `$mode partial.expressions` | Added | `a and b` and `a or b` stop evaluating as soon as the value is known <!--s 417--> |
| `$mode strict.equate` | Added | warns when a name given to `equate` or `$define` is also a statement or function name <!--s 582--> |
| `$mode trap.unused` | Changed | warns only of a value an ordinary assignment gave and nothing read; a statement's own target (`readu`, `locate ... setting`, `capturing`) is not reported |
| `$mode trap.unused.main` | Added | `trap.unused` for the program's own variables <!--s g227--> |
| `no.query` on cataloging | Added | the `basic` command, the `$catalog` directive and the `catalog` line of `$basic.options` take `no.query`, which stops the compiler asking before it removes an older entry <!--s 27 191--> |
| `open ... non.transactional` | Added | a write through that file variable is not undone by a rollback <!--s 64--> |
| `open ... sync` | Added | the file's data is written to disk after every write and delete; `fileinfo(f, fl$sync)` says whether it was opened so |
| `openseq` | Added | `creating` makes the file at once, only under the lock on its name, refuses a link to nothing and gives `status()` 6032 if the file appeared meanwhile; `shared` lets several processes append; a block device can be opened <!--s 485 397 112--> |
| `opentemp` | Added | makes a temporary dynamic file, at a path the program names or under a generated name, deleted when the last file variable on it is closed; it asks the `os` switch, as `openpath` does |
| `option()` | Added | reads and sets the state of an option <!--s 224 473--> |
| `oscopy` | Added | copy an operating system file; `overwrite`, `append`; `!oscopy` and `!ioscopy` (a whole tree) for programs <!--s 517 369--> |
| `ospath()` | Added | keys `os$fullpathx` and `os$info`; and any program, not only an internal one, may now call `ospath()` with openQM's key numbers 0 to 18, behind the account's OS switch (`os$uid`, `os$gid` and `os$modes` answer 0 on Windows; key 18 is the owner's Windows account name) <!--s 293 432 233--> |
| `osread`, `oswrite`, `osdelete` | Added | a whole operating system file read, written or deleted by name; outside the account's directory they need the `os` switch; `osdelete` removes a file, never a directory |
| `parse()` and `ttyget()` | Added | `parse()`, `sort.compare()`, `ttyget()`, `ttyset`: split by a pattern; compare by a sort rule; the terminal modes as an array (`terminfo.ex()` was added and removed again, see REMOVED) <!--s g166 g183 g187 g190 g191--> |
| `pdump` | Added | writes a process dump of the running program and goes on |
| `phantom` statement | Added | starts a phantom from a program; the log name may be a variable <!--s 390 351--> |
| `precision inherit` | Added | takes the precision of the calling program <!--s 201--> |
| `@pstat` | Added | text a program leaves for `pstat` to show <!--s 492--> |
| `quotes()` | Added | encloses every element of a dynamic array in double quotes <!--s 11--> |
| `read` etc. as a condition | Added | `read`, `readnext`, `open` and the like may be the condition of `while` or `until` <!--s 206--> |
| `recordlocku` | Changed | fixed: it did not always trap an illegal record id <!--s 528--> |
| `redim` | Added | resizes a matrix passed to a subroutine as an argument <!--s 215--> |
| `regex()` | Added | regular expression matching <!--s 510--> |
| `seek` | Changed | fixed: a negative offset from the end of a sequential file was refused <!--s 259--> |
| select list variables | Added | `formlistv`, `getlistv`, `writelist`, `movelist`, `movelistv`: select list variables and numbered lists <!--s 291 g134 g135 g195 g156--> |
| `select.socket()` | Added | waits on the sockets held in a matrix <!--s 161 g176--> |
| `selectindexv` family | Added | `selectindexv`, `selectleftv`, `selectrightv`: index scans into a select list variable <!--s g177--> |
| `send` | Added | `writeseq`, or with a colon `writeblk`, to a file opened with `openseq` <!--s g178--> |
| `$setptr.defaults`, `@seqno` | Added | a VOC record of the options `setptr` takes for each mode as if written after it; `@seqno` and `getpu(pu$seqno, unit)` give the number of the last `as next` hold file |
| `socket.info()` | Changed | fixed: `skt$info.open` on a number, a string or an unassigned variable stopped with a fatal error <!--s 249--> |
| `!sort` | Changed | sorts a select list variable; the internal sort is open to programs <!--s 274 371--> |
| `sselectv`, `sselectn` | Added | sorted selects into a select list variable; `right.aligned` and `descending` <!--s 209 243 367--> |
| `status()` after a transaction | Added | 0 if it committed, 1021 if it rolled back <!--s 19--> |
| string arithmetic | Added | `sadd`, `ssub`, `smul`, `sdiv`, `scmp`: exact string arithmetic on numbers of any length and any decimals <!--s 391 468--> |
| substring assignment | Added | `s[3] = "xy"` overwrites the last three characters <!--s 41 325--> |
| `trans()` family | Changed | `trans()`, `rtrans()`: accept an `l` link; a file that will not open takes the error action of the call <!--s 43 182 7--> |
| variable names | Changed | a name with two or more underscores together (`a__b`) is valid <!--s 336--> |
| `watch` | Changed | the debugger's `watch variable operator value` stops only when the condition is met <!--s 45--> |
| `write ... creating.seqkey` | Added | the hashed file's counter gives the record id and puts it in a variable; an id already in use is skipped; `fileinfo(f, 1031)` is the last one given |

## DATA

Data collections, JSON, XML, CSV and the verbs that import them.

| Name | Change | What |
|---|---|---|
| collection arrays | Added | `mat()`, `c{list, n}`, `ins ... as`, `del`, `inmat()` on an array |
| collection file verbs | Added | `ed`, `copy`, `compare.files`: work on a collection file; the post-write trigger no longer crashes the process |
| collection files | Added | `create.file name collection`; records are collections stored as JSON; `read` and `write` them whole; `ct` shows JSON <!--s 304--> |
| `collection()` and `c{name}` | Added | a collection is a set of named values read and written by name; `enumerate()`, `element.exists()` <!--s 311 518 529 339--> |
| `csv.mode` | Added | how `printcsv`, `writecsv` and `formcsv()` quote, with a delimiter, quote and escape character <!--s 173 200--> |
| `csvparse()` and the CSV clauses | Added | `readcsv`, `writecsv` and the rest take `delimiter`, `quote` and `escape`; a quoted field with a line break is one row; `csvparse()` and `csvbuild()` read and write a whole CSV text as a collection; `csvopen()`; `csvdq()` takes a quote and an escape character |
| `element` keyword in queries | Added | names an element of a collection file by its path, in a display list, `with`, `by`, `sum` and the like |
| `expand()` and links | Added | a link `"file:id"` is followed; `trans()` of an `e` item |
| file variable in a collection | Changed | a collection holding a file variable is written to a collection file as a null |
| `hu` conversion | Added | `oconv` and `iconv` escape and unescape |
| `import.csv`, `import.json`, `import.xml`, `convert.data` | Added | read a CSV, JSON or XML file into the records of an SD file, and write a CSV, JSON, JSON Lines or XML file as another of them; the pathnames ask the `os` switch, and an import into a VOC needs `admin` |
| `indices` on `e` items | Added | a data collection element can be indexed; a query with `with` on it uses the index |
| `jboolean()` and `jtype()` | Added | `jboolean()`, `jnull()`, `jtype()`: true, false and null in collections, and the type of a value; a Boolean type <!--s 312--> |
| `jmerge()`, `jpatch()`, `jpath()`, `jvalidate()` | Added | merge (RFC 7396), patch (RFC 6902), JSONPath queries and JSON Schema checks of a collection; `readjson` and `writejson` read and write JSON Lines; `json` and `json.lines` report keywords |
| `jparse()`, `jbuild()` | Added | JSON text into a collection and back; `jbuild()` takes an indent for the pretty-print mode |
| `oconv()` date `i` | Changed | fixed: the ISO week number is worked out without `yi` beside it (it gave whatever number was left in memory) |
| operators on an element | Added | `+=`, `-=`, `*=`, `/=`, `:=` change an element in place |
| the debugger and collections | Added | `/c`, `/c{path}` and `dump c{sub} pathname` show and write a collection; diving in with `/*` no longer crashes <!--s 530--> |
| XML functions | Added | an XML document into a collection and back, and XPath 1.0 queries over it; no DTD is read |
| `xml` keyword for queries | Added | `list ... xml` writes the report as XML, with an optional DTD or schema <!--s g211 g202--> |

## FILES

File handling: opening, indexes, triggers, print files, sorting, sequential files.

| Name | Change | What |
|---|---|---|
| `analyse.index` | Added | every index with its number of values and spread <!--s g9--> |
| `analyze.file` | Changed | `statistics` shows a histogram of group utilization <!--s 360--> |
| `clear.file` | Changed | fixed: a `no.resize` file keeps its modulus <!--s 590--> |
| `configure.file ... next` | Added | `next n` sets and `next query` shows the next id `write ... creating.seqkey` gives; a rebuild keeps it |
| `configure.file ... reporting` | Added | shows how many records have been converted <!--s 172--> |
| `create.file` | Added | `comment` and a relative `pathname` kept relative in the VOC <!--s 383 522 278--> |
| `create.index` | Added | `no.case`: keys that differ only in case are one key; `deleting` replaces an index; a default location in `$index.path` <!--s 54 424 184--> |
| `dir_dict` | Added | three items for a directory file's dictionary: `days`, `dtm` and `owner` <!--s 181--> |
| directory file line ends | Added | line terminator modes (`l`, `c`, `b`, `x`) and triggers <!--s 177--> |
| `directory files` ids | Changed | the record ids may hold bytes 128 to 247 <!--s 34--> |
| `disable.index` | Added | stops an index being kept up to date for a large load; `build.index` switches it on <!--s g28--> |
| exclusive access | Changed | a command that needs a file to itself names the user holding it when it cannot get it <!--s 107--> |
| `file.stat` | Added | one line per file: type, modulus, records, bytes and flags; limited to named files; with dictionaries <!--s 109 152 g40--> |
| `find.file`, `find.program` | Added | what a file is; the catalogue entries of a program <!--s g42 g43--> |
| `fixdir` | Added | takes the carriage returns out of directory file items that came from a Windows system <!--s 458--> |
| hashed files | Changed | a split that finds the disk nearly full stops before the file's size is changed; `sdfix -z` clears dead space <!--s 329 33--> |
| `hold.file.logger` | Added | a catalogued subroutine called when a hold file print unit opens and closes <!--s 56--> |
| index, deleting many records | Changed | fixed: removing the last entry under a branch of an index of three levels cut the rest of that branch out of every lookup by key; rebuild an index that has had deletions with `build.index` |
| `l-type` dictionary links | Added | usable in an `i`-type; links may follow links, up to ten, and reach `e` items <!--s 7 204 544--> |
| `list.common detail` | Added | shows where each common block was created <!--s 362--> |
| `list.index` | Changed | the report may go to a printer <!--s 73--> |
| `list.triggers` | Added | every trigger subroutine used by the files of an account, dictionaries and directory files included <!--s 74 187 321--> |
| print unit mode 3 | Changed | takes an update lock on the `$hold` item while it is written <!--s 53--> |
| `rebuild.all.indices` | Added | rebuilds the indices of every file in the account <!--s 58--> |
| `select` of a directory file | Changed | fixed: at the open-file limit it closes the oldest cached file and goes on <!--s 279--> |
| `set.file` | Changed | the target may be a `q`-pointer; a `q`-pointer with no file name points at the VOC <!--s 32 435--> |
| `set.trigger` | Changed | fixed: refused every function name; takes `all`; no longer needs exclusive access <!--s 388 157--> |
| sort of long records | Changed | the disk sort handles records and keys over 32 KB <!--s 455--> |
| sort that uses work files | Changed | fixed: a sort that needs more work files than `sortmrg` put records with equal keys out in an order that depended on `sortmem` and `sortmrg`, and a `unique` sort could keep a later record; now they come out in the order they arrived |
| trigger on a read | Changed | fixed: a locally catalogued read trigger could end the process <!--s 317--> |
| `vocname` letters | Added | `r` opens a file read-only, `n` non-transactionally, `s` with forced writes, from field 6 of its VOC entry <!--s 197 380 102--> |

## QUERY

The query processor and the dictionary.

| Name | Change | What |
|---|---|---|
| `absent.ignore` | Added | do not print the `not found` list for ids that are not in the file <!--s 25--> |
| `break.on "'n'"` | Added | starts a new page and numbers it 1 again <!--s 16--> |
| `by` with equal keys | Changed | records with equal sort keys come out in record-id order, also under `by.dsnd`; `sort` and the other S verbs already did |
| `by.exp` | Changed | more than one `by.exp` in the same association is sorted <!--s 572--> |
| `csv 3` | Added | every item quoted, numbers too <!--s 26--> |
| `csv ... as file id` | Added | `as file id` after `csv` or `delimiter` writes the report as a record of a directory file (`no.query`, `appending`); a multivalued field gives one line for each value |
| dictionary expressions | Changed | divide by zero is zero (an `if` with a part missing, `fn` for a field, a format after a value and hyphens in names were added and removed again, see REMOVED) <!--s 36 48 270 271 423 547 564--> |
| `for`, `from none` | Added | `search file for "text"`; `from none` ignores an active list 0 <!--s 190 378--> |
| `in`, `not.in` | Added | test a value against a saved list or a file record <!--s 94--> |
| `median`, `mode`, `range` | Added | figures at the end of a report and each breakpoint <!--s 231 g209--> |
| `$mode` in an `i`-type | Added | an `i`-type may begin with `$mode nocase.strings`, so its string comparisons ignore case <!--s 489--> |
| `no.page` | Changed | accepted on any query command <!--s 379--> |
| `$query.defaults` | Changed | field 3 holds default words for `select`, `sselect` and `search`; field 4 the format of the id column <!--s 122 405--> |
| `reformat ... @record` | Added | appends the whole source record <!--s 546--> |
| `reporting` | Added | `n record(s) examined` for every thousand records <!--s 416--> |
| `rpt` | Added | repeats a single valued field on each line of a report on a multivalued one <!--s g210--> |
| `scan`, `search ... display` | Added | show the lines of a record that hold the strings; `search all` searches the id too; strings may come from a file <!--s 437 447 221 141--> |
| `select file field` | Changed | a field no clause uses is `saving multivalued field` <!--s 39--> |
| `show` | Changed | `b` goes to the bottom; `sort.sup`; a missing record is listed and the command ends; `show ... eval` writes nothing into your dictionary <!--s 477 495 524--> |
| `sorted.sample n` | Added | counts the records after the sort <!--s 438--> |
| `vertically` | Changed | `fmt` for the name and data format; `$vlist` sets them for the file; multivalued data and a wrapped line are told apart <!--s 160 579 580--> |
| `when field` | Changed | with no operator keeps the values that are not null; `between` compares every value of a multivalued field, not only the first; `unlike no.case` ignores the case of the pattern <!--s 543 343 248--> |
| `x` breakpoint control | Added | captures the previous value of a break field in `@bpv` <!--s 232--> |

## SYSTEM

The kernel, `sd.conf`, and the numbered keys.

| Name | Change | What |
|---|---|---|
| `-quiet` | Changed | `sd -quiet` suppresses the VOC update question <!--s 533--> |
| `-status` | Added | `sd -status` shows the version, start time and process counts <!--s 511--> |
| `cleanup=n` | Added | seconds between the checks that clear sessions that ended without logging out <!--s 98 g283--> |
| `config` | Changed | lists the new settings; `config()` reads them |
| `deadlock=2` | Added | saves the lock tables when a deadlock is found <!--s 500--> |
| `fileinfo()` | Added | the keys 1022 to 1040 as the openQM manual numbers them; `fl$seqtype`, `fl$locks`; 1021 now answers the user numbers <!--s 62 113 498 337--> |
| `intprec` | Changed | default 11 <!--s 6--> |
| lock table | Changed | the time each lock was taken and each wait began is noted; more than 65,535 locks in a file work; a program that filled the table can be logged out <!--s 487 344 532--> |
| `numlocks` | Changed | range checked, 1 to 32767 |
| `phantoms=u,n` | Added | keeps user numbers for phantoms started with `user` <!--s 31 g296--> |
| process dump | Changed | shows the length of each string, the variables of objects and the environment; the dump file is readable by its owner only <!--s 22 125 569 439--> |
| `pterm` keys | Added | `pterm erase`, `ctrl.filter`, `mark`, `query`: which key erases; control characters filtered; control keys that type the marks; the bytes a key sends <!--s 155 253 254--> |
| `sd.conf` parameters | Added | `dhcache`, `lgnwait`, `maxrlock`, `pwdelay`, `userpool`: `sd.conf` parameters, each optional with the old fixed number as the default <!--s 90 46 313 106 551--> |
| `sortmem` | Changed | default 4096 <!--s 535--> |
| `status` | Changed | file times are in the user's time zone; field 27 is the file pathname <!--s 96 585--> |
| `system()` keys | Added | 14, 19, 20, 21, 33, 35, 45, 1016, 1036, 1037, 1040, 1045, 1047, 1048, 1049, 1051, 1062, 1065, 1067, 1071, 1073, 1083, 1086, 1088 <!--s 65 66 114 192 225 244 206 294 307 308 326 421 433 586 587 g252 g253 g254 g255 g256 g260 g261 g265 g266 g267 g270 g271 g274 g276--> |
| terminfo | Changed | `eblink`, `eprot`, `edim`, `erev`, `ebold` capabilities <!--s 47 545--> |
| the call name | Changed | can be as long as 128 characters (the lower of 128 and `maxidlen`) <!--s 241--> |
| time zone and `epoch()` | Added | `epoch()`, `mvdate()`, `mvtime()`, `mvdate.time()`, `mvepoch()`; `set.timezone` and `config timezone` <!--s 69 124 g126 g157 g158 g159 g160 g181 g310--> |
| `usplit=1` | Added | interactive users take the lowest user number, phantoms and API sessions the highest <!--s g312--> |

## TCL

Commands and VOC verbs that every account has, and the command language.

| Name | Change | What |
|---|---|---|
| `basic ... no.catalog` | Added | compiles and does not run the program's catalog directive; the compile says so |
| `block.print`, `block.term` | Added | print text, or show it on the terminal, in letters seven rows high <!--s g11 g12--> |
| `catalog ... global c` | Added | puts the prefix `!`, `_` or `$` in front of every call name that has none of its own, so a whole select list can be cataloged globally |
| `catalog ... local relative` | Added | stores the object file's path relative to the account <!--s 131--> |
| catalog name as a command | Changed | a program run by its catalogue name takes `lptr`, `no.page` and the like <!--s 146--> |
| `catalog` overwrite | Changed | pointing a local catalogue entry at a different object file asks first <!--s 490--> |
| `catalog` prefix | Added | a prefix separate from the call name; `$private.catalog` or `$private.catalogue` <!--s 210 93--> |
| command stack | Changed | `.c-` undoes a `.0`; the stack is kept in the account the user entered <!--s 296 396--> |
| `command.logging` | Added | a log of every command this session runs, in the account directory, readable by its owner only <!--s 444 g16--> |
| `como` | Added | `append`, `in file`; `$como` ids pass the same name transform <!--s 8 361 9--> |
| `compare.files` | Added | counts the records only in one, only in the other, in both but different <!--s 534--> |
| `ct` | Changed | `id.sup`, `num.sup`; lower case file names; the object code of a compiled dictionary item is not shown <!--s 242 347 373--> |
| `creating.seqkey` in `ed` and `sed` | Added | in the place of a record name: a new record under the file's next sequential id |
| `decatalog` | Added | deletes a local catalogue entry and its object code <!--s 576--> |
| `delete` | Changed | a record another user holds locked is reported and skipped <!--s 50--> |
| `delete.account` | Changed | not stopped by a file it cannot open when it looks for references <!--s 348--> |
| `delete.common all except` | Added | deletes every named common block but those listed <!--s 425--> |
| `ed` | Changed | `l` with no text as the first command works; marks typed in input; `ev` on a compound `i`-type; line numbers show positions in edit values mode; `spool` closes the printer; external commands; automatic ids <!--s 264 268 320 331 375 446 523 305--> |
| `ed` and `pick` with piped input | Changed | fixed: they leave their record or list when the pipe has ended, instead of asking again for ever |
| `fstat` | Changed | wider columns <!--s 55--> |
| `fstat.user` | Added | file activity of each session <!--s g44--> |
| `git` option | Added | a directory file keeps `.gitignore` and `.gitattributes` under their own names <!--s g238--> |
| `hot spot monitor` | Changed | counts opens, reads, writes and deletes of each program <!--s 491--> |
| `if` in a paragraph | Added | `exists`, `not.exists`, `in` and `not.in`, with `no.case` <!--s 28 230 426 476--> |
| inline prompts | Changed | largely case insensitive; may use quoted prompt strings <!--s 171 377--> |
| `k` entry in the VOC | Changed | fixed: a `k` entry over a locally catalogued program stopped with "Unable to load" <!--s 263--> |
| list merges | Changed | `merge.list` finds a key without scanning, so large lists merge in seconds <!--s 414--> |
| `list.item` | Changed | numbers lines beyond 999 with as many digits as needed <!--s 349--> |
| `listu`, `who`, `pstat` | Changed | `!listu` too: show the account and the login name; `listu detail`; the blocking user and when a wait began <!--s 12 24 29 38 77 121 289 427 577--> |
| `logto reset all` | Added | discards all programs, menus and paragraphs at every command level <!--s 30--> |
| `map local`, `map no.page` | Added | list the locally catalogued programs; heading once <!--s 153 287--> |
| `move.list` | Added | the command form of `movelist` <!--s g62--> |
| `multisort` | Added | `!multisort` sorts several related lists by one of them <!--s 568--> |
| `option` words | Added | `dir.dtm`, `git` and `spool.command` (the others added in this release were removed again, see REMOVED) <!--s 288 79 314 428 461 445 37 57 154 165 404 429 49 g230 g231 g232 g233 g237 g239 g240 g241 g242 g245 g247 g248--> |
| `phantom` | Added | `no.msg`, `data`, `log`, `group` and `user` options; `!phlog` gives a phantom's log record; a phantom started while SD is suspended gives back its place <!--s 87 137 364 483 31 385 280--> |
| phantom and terminal lists | Added | `list.phantoms`, `list.term.types`: the phantoms and no one else; the terminal types <!--s 381 g49--> |
| `private vocabulary` | Added | `$private.voc`: a file of commands searched after all other command sources <!--s 1--> |
| `profiler` | Added | records what the programs of a session do and where <!--s g66 g203--> |
| `redo` | Added | runs a command again and again, `times` and `every` <!--s 188--> |
| saved list verbs | Changed | `save.list`, `get.list`, `delete.list`: `keep`, `deleting`, `count.sup`, a default name, a name left out, `in file` <!--s 2 384 387 401 494--> |
| SDSYS's `bp` examples | Added | SDSYS's `bp` file holds `py_json`, `py_term`, `py_test` and `py_test2`; an upgrade replaces those four and keeps the rest of `bp` |
| `selectindex` | Added | the command form of the statement <!--s 119--> |
| `setptr` | Changed | `form.feed`, `inform`, `eject`, `as ... in file`; the digits of `next` <!--s 51 59 70 139 448--> |
| `show.list` | Added | a page at a time, starring and clearing items <!--s 449--> |
| `sort.list`, `select.dir` | Added | sort a saved list; select the files of a directory <!--s 158 465--> |
| `sp.view` | Changed | `by field`; `x` deletes the item shown; shows where output goes for every print unit mode <!--s 450 560--> |
| `term` | Changed | takes two numbers for the default printer <!--s 466--> |
| `update.accounts` in the SDSYS account | Changed | refreshes SDSYS's own VOC from `voc_template`, which is installed again |

## TOOLS

The debugger, from the openQM list.

| Name | Change | What |
|---|---|---|
| `debugger` | Changed | full-screen display on a terminal with an alternate screen; `/*` for every variable; `^`; `window n`; `set v = token`; a stop before each `case`; a blank line repeats the last command; `brk` lists the breakpoints <!--s 21 67 97 142 193 203 327 335 513 550--> |
| `debugger` missing source | Changed | says when it cannot find the program's source and lets it run on; `no.source.warning` in a `$debug.options` VOC record leaves the message out |
| `debugger` phantom | Changed | an `@variable` is read from the process being debugged <!--s 251--> |

## REMOVED

These were in WS1.1-3 - or were added and taken out again during WS2.0-0 - and **SD no longer has them**. This is the one place where a removal is listed,
whatever the area it belonged to. Most were removed because they existed only to make SD behave like another MultiValue
product, or gave a second way to do something SD already does one way. **A program, a login paragraph or a `$basic.options` record that uses one has to
be edited**; the last column says what to write instead. A program that was compiled with a removed form keeps what it was
compiled with until it is compiled again; compiling it again stops with an error that names the form.

| Name | Change | What |
|---|---|---|
| `!` (the shell verb) | Removed | write `sh command` |
| `=>`, `=<` and `><` | Removed | the mirrored spellings of `>=`, `<=` and `<>`: a new account has no record for them, and the next `update.accounts` removes them from an existing account |
| A, S and C dictionary item types | Removed | with them the `a;nocase` correlative and an `if` in field 8 of an `a` or `s` item: write the item as a `d` item (a field of the record) or an `i` item (a value worked out from the record) |
| AccuTerm terminal types | Removed | the eleven `-at` terminal types and AccuTerm's escape sequences |
| `adds()`, `subs()`, `muls()`, `divs()` | Removed | added and taken out again in this release |
| `@am`, `@svm`, `@user`, `@user.no`, `@filename` | Removed | second names of variables that stay: write `@fm`, `@sm`, `@logname`, `@userno` and `@file.name` |
| `analyse.file` | Removed | the British spelling: write `analyze.file` |
| `bscan`, `root`, `key()` | Removed | added and taken out again in this release: scan an index with `selectindex`, `selectleft` / `selectright` and `setleft` / `setright` |
| `call "file name"` | Removed | the Pick style call was added and taken out again in this release: catalog the program and call it by name |
| `caption`, `nopage`, `col.hdr.space` | Removed | `caption` and `nopage` were second names of `grand.total` and `no.page`; `col.hdr.space` was added and taken out again |
| `catalogue`, `delete.catalogue`, `$catalogue` | Removed | the British spellings: write `catalog` (the verb and the `$basic.options` word), `delete.catalog` and `$catalog` |
| `cd` | Removed | the short name of `compile.dict`: write `compile.dict` |
| `cleardata`, `clearinput`, `clearprompts`, `clearselect` | Removed | the run-together names of the verbs `clear.data`, `clear.input`, `clear.prompts` and `clear.select` (the BASIC statements of the same names are not affected) |
| `closeseq` and `equ` | Removed | write `close` and `equate` |
| `colour`, `without`, `no.grand.total` | Removed | write `color`, `with no` and `grand.total "'L'"` |
| `$contrib` and `$licence` | Removed | the VOC records of the contributors and the licence notice; read the files `contrib` and `licence` in the system folder, as on SD Core Solo for Linux |
| `copyp`, `copy.listp` | Removed | the Pick-style copy and its saved-list form (`copy.listp` was added and taken out again in this release): write `copy` and `copy.list` |
| `create.file ... no.case`, `case` and `version` | Removed | write `create.file` without them; a file is made in the current format |
| `cs` | Removed | second name of `clr`: write `clr` |
| dictionary field numbers 9998 and 9999 | Removed | an item whose location was 9998 gave the field count and 9999 the record length in bytes; they are ordinary field numbers now: write an `i`-type |
| `dquote()`, `dquotes()`, `swap()`, `xlate()` | Removed | second names of `quote()`, `quotes()`, `change()` and `trans()`: write those |
| `extract()`, `replace()`, `insert()`, `delete()` and `subr()` | Removed | write `a<f,v>`, `a<f,v> = x`, `ins`, `del` and `call` or `deffun` |
| `FILERULE` | Removed | the `sd.conf` parameter and the `path:pathname` form of a file name (write an F-pointer); the `account:file` form is always on, checked as a Q-pointer is |
| fourteen dead query keywords | Removed | `verify`, `verify.sup`, `exclusive`, `creating`, `inquiring`, `portrait`, `odd`, `even`, `mark`, `space`, `no.label`, `no.object`, `static` and `encrypt` were read by nothing |
| `go`, `go to`, and `#` before a compiler directive | Removed | write `goto` (also in `on ... goto`) and `$ifdef`, `$define` |
| `inputclear` | Removed | write `clearinput`; the file type constant `fl$type.sh` (a static hashed file SD never had) is gone too |
| `is.alnum()`, `is.alpha()`, `is.digit()`, `is.graph()`, `is.mark()`, `is.space()`, `left()`, `right()`, `md5()` | Removed | added and taken out again in this release: write `s[1, n]` and `s[n]` for the ends of a string, and `digest()` for md5 |
| `list.union`, `list.inter`, `list.diff` | Removed | the saved-list twins of `merge.list`: write `merge.list` |
| `listdict`, `edit.list` | Removed | write `list dict` *file*, and `ed $savedlists` *name* |
| `listf`, `listfl`, `listfr`, `listk`, `listpa`, `listph`, `listq`, `listr`, `lists`, `listv` | Removed | write `list voc with type = "F"` (and `"K"`, `"PA"`, `"PH"`, `"Q"`, `"R"`, `"S"`, `"V"` for the others); the file `sd.voclib` that held them and its VOC pointer are gone |
| loose forms of a dictionary expression | Removed | an `if` with no `then` or no `else` is refused again, a hyphenated name such as `cust-no` is a subtraction, and a format string straight after a value needs its operator |
| `make.index` | Removed | write `create.index` and then `build.index` |
| `mcd`, `mcx` and `mcxd` conversions | Removed | second spellings of `mx`: an invalid conversion code now (`mcdx` stays, as `mx`) |
| `md` | Removed | the Pick name for the VOC: write `voc` |
| `$mode` words added in this release | Removed | `change.no.overlap`, `conditional.statements`, `count.overlap`, `default.unass.args` (a default argument value needs `var.args`), `exception.save.stack` (write `saving.stack` on the `catch`), `execute.clearlist`, `implied.stop` (end a program with `stop`), `index.overlap`, `open.fail.zero.fvar`, `pick.convert`, `pick.readnext`, `pick.select`, `selectv`, `stdfil` (name the file variable), `stdfil.shared`, `time.ms` and `write.delete.then.else`: each is refused, and the program does what it did with the mode off |
| `$mode` words of other products | Removed | `autolock`, `case.sensitive`, `compatible.append`, `composite.readnext`, `for.store.before.test`, `heading.no.eject` (write `heading no.eject`), `no.case.invert`, `optional.final.end`, `optional.then.else`, `pick.enter` (write `call`), `pick.jump.range`, `pick.matrix`, `pick.read`, `pick.substr`, `pick.substr.assign`, `prclose.default.0`, `string.locate` and `unassigned.common`: each is refused, and the program does what it did with the mode off; their bits stay reserved |
| multifiles | Removed | `create.file name,part`, `create.file ... multifile`, `delete.file name,part`, the `file,part` form of a name in `open` and the TCL verbs, field 4 of a VOC F-pointer and the parser key `parser$mfile` (write `parser$get.token`): make each part an ordinary file |
| `nano` | Removed | the second external editor: write `edit` or `micro` |
| old terminal definitions | Removed | 46 entries: the Data General, MDC Prism, Prime and Televideo 910 definitions, and `addsviewpoint`, `viewpoint`, `adds4000`, `viewpoint60`, `vp60`, `vp`, `vpa2`, `vt52`, `hpgeneric`, `hp`, `ibm-pc`, `ibm5051`, `5051`, `glasstty` and `kermit`: use `ansi`, `vt100` or `xterm` (`wyse50` and `wyse60` stay) |
| `only` and `id.only` before the file name | Removed | `list only customers` is refused: write `list customers id.only` |
| `option` words added in this release | Removed | `assoc.all`, `backslash.not.quote` (a backslash in a command line is always part of a path, never a quote), `catalogue.local`, `chained.select`, `clear.select` (with the `keep.select` statement), `command.level.locks`, `correlative.nocase`, `correlative.reuse`, `default.mv`, `dir.sel.omit.hidden`, `force.reload`, `inherit.mfile.path`, `keep.old.object`, `no.phantom.notify`, `numeric.exponent`, `paginate.on.heading`, `pick.ml.conv.mask`, `query.footer.pause`, `query.merge.print`, `query.sort.no.case`, `query.str.comp.all`, `query.string.comp` and `stacked.account`: each does what it did when off, and `option` shows its place as reserved |
| `option` words of other products | Removed | `option pick` and `option sdbasic.warnings` (the shortcuts), `ampm.upcase`, `assoc.unassoc.mv`, `chain.keep.common`, `crdb.upcase`, `create.file.upcase`, `debug.rebind.keys`, `div.zero.warning`, `ed.no.query.fd`, `inherit`, `lock.beep`, `no.date.wrapping`, `no.sel.list.query`, `non.numeric.warning`, `pick.breakpoint`, `pick.breakpoint.u`, `pick.explode`, `pick.grand.total`, `pick.implied.eq`, `pick.null`, `pick.wildcard`, `proc.a`, `qualified.display`, `query.no.case`, `query.priority.and`, `run.no.page`, `select.keep.case`, `space.mct`, `suppress.abort.msg`, `unass.warning` and `with.implies.or`: each does what it did when off (a phantom starts with every option off; `chain` always discards the unnamed common; a run is paginated) |
| `procread`, `procwrite`, `@pib`, `@sib`, `@pob`, `@sob` | Removed | leftovers of PROC, which went earlier |
| `rename` | Removed | second name of `cname`: write `cname` |
| screen save and restore | Removed | `save.screen()` and `restore.screen`: still exist and do nothing the terminal could see |
| `sd -<n>` and eleven `sd.conf` settings | Removed | `apilogin`, `codepage`, `creatusr`, `exclrem`, `fixusers`, `gdi`, `jnldir`, `jnlmode`, `portmap`, `ringwait` and `txchar`: SD refuses to start on a setting it does not know, so the upgrade takes the line out of a kept `sd.conf`; `sd -<n>` (start with a given user number) is gone |
| `sd -l` | Removed | "apply new license": SD has no license; `sd -l` is an unrecognized option now |
| `sdterm` terminal type | Removed | with SDTerm and SDConsole, and the `term display` line about them |
| second forms of statement clauses | Removed | `execute ... setting` (write `returning`), `execute ... passlist` with no variable (name the variable, or leave the clause out), `input ... for` and `input ... timeout` (write `waiting`) and the bracketed Pick form of `locate` (write `locate expr in array<f, v> setting position`) |
| second names for functions and variables | Removed | `ereplace()` (write `change()`), `sentence()` (read `@sentence`), `swapmarks()` (it returned its argument), `terminfo.ex()` (write `terminfo()`) and `@system.set` (write `@system.return.code`) |
| second names for statements | Removed | `$insert` and `include` without the `$` (write `$include`), `com` (write `common`), `dimension` and `redimension` (write `dim` and `redim`), `enter` (write `call`), `perform` (write `execute`), `printerr` (write `inputerr`), `remark` (write `rem`), `rqm` (write `sleep`), `sub` (write `subroutine`) and `tclread` (read `@sentence`): the compiler refuses each |
| second spellings of query keywords | Removed | `col.spcs` (write `col.spaces`), `display.name` (`col.hdg`), `evaluate` (`eval`), `calculate` (`calc`), `intersection` and `difference` (`intersect` and `diff`), `margin` (`left.margin`), `stats` (`statistics`), `supp` (`hdr.sup`) and `only` (`id.only`): the next `update.accounts` removes their VOC records, through the deletion list (saved copy first; a record of your own with the same name is kept; `restore.voc` puts one back) |
| second spellings of query words | Removed | `avg`, `enum`, `vert`, `percent`, `pct`, `%`, `header`, `footer`, `first`, `#` and `&`: write `average`, `enumerate`, `vertically`, `percentage`, `heading`, `footing`, `sample`, `ne` and `and`; a `break.on` control `l` is no longer ignored when `u` is given too; and `f` and a number names a field only if the dictionary has an item of that name |
| `set.queue`, `sp.assign`, `getpu(pu$form.queue)`, `system(1066)` | Removed | Pick style form queues were added and taken out again in this release: use `setptr` with the settings you want |
| `setptr` print mode 5, and its `gdi` and `raw` words | Removed | write `setptr` without them |
| `sp.open` and `sp.close` | Removed | write `printer keep.open` and `printer close` |
| `system:account:file` file names | Removed | a Q-pointer's fourth field named a file on another system and opened nothing; a Q-pointer with a fourth field is refused |
| `term 120,36` at login | Removed | new accounts no longer carry it in their `login` paragraph |
| terminal case inversion | Removed | `pterm case invert` / `noinvert` (and the line in `pterm display`), `input ... nocaseinvert` and the `invcase` setting of `sd.conf` (the upgrade takes it out of a kept `sd.conf`); `keyinc()` is now the same as `keyin()`. Type in the case you want |
| the global catalog prefixes `*` and `-` | Removed | `!`, `$` and `_` stay; on a managed computer `sync.global.catalog` catalogs the server's programs under their own names, so a program calls one with `call name` and must be compiled again |
| the Pick style matrix in the run-time system | Removed | nothing could make one since `$mode pick.matrix` went; its descriptor type and the branches for it in the kernel, the debugger and the process dump are gone. No program changes |
| the word forms of query operators | Removed | `equal` (write `eq`), `greater` and `after` (`gt`), `less` and `before` (`lt`), `matches` and `matching` (`like`), `not.matching` (`unlike`) and `spoken` (`said`) |
| VOC record type M (menu) | Removed | a VOC record of type M is no longer run as a menu; the home screen is the menu system |

## New Features Unique to SD Core

Everything in this section is SD Core's own. None of it is on the openQM list, so none of it has a row in the
feature spreadsheet: the shell and operating system switches, the protection of the VOC and the server's files, the
screen library, the installer's additions and the fixes to SD's own code. The sections above are the openQM items that
were built, with the changes they made.

### Administration

| Name | Change | What |
|---|---|---|
| `docs.install` | Added | the administrator copies a directory of PDF files in, for the `pdf` verb to show |
| `load.help` | Added | F1, `?` or a click on a row of an administrator's page shows a short help box, kept in the file `context.help`, which `load.help` fills |
| `mcp.call` | Added | the work of every call of the MCP launcher `solo-mcp.py` - the gate, the checks of every word, the work and the audit line; not typed by a person |
| `pdf` | Added | shows a document in the PDF reader the administrator has allowed, even for an account that may not run programs; no reader is allowed as delivered, and it does nothing over ssh |
| `remote.api`, `remote.ssh` | Added | administrator commands: report each door, or set it off, local or open, from any session, through a doors helper the installer sets up; the administrator password is asked again; `remote.ssh local` or `open` sets ssh up if it was not chosen at install |
| `restore.account` | Changed | fixed: no longer overwrites a different global program of the same name |
| `set.config` | Added | administrator command: changes one of 14 tuning parameters of `sd.conf` in the Solo folder, within the range SD enforces, keeps the old file as `sd.conf.bak` and writes one audit line; the change takes effect at the next `sd-solo -start` |
| `set.mcp` | Added | administrator only: opens the MCP door for an AI agent's MCP client (`set.mcp on`) and gives it the write tool (`set.mcp write on`); both off as delivered; Getting Started page 20, The MCP door |
| `set.os.access` | Added | administrator command: `sh on`, `sh off`, `os on`, `os off`; with no argument it shows both; it writes the account's register record and the audit trail, and a change is seen at the next sign-on |
| `set.pins` | Added | administrator: `set.pins on` lets the account add programs of its own to its home screen (`my.menu`) |
| `set.start.default` | Added | administrator: `menu` or `terminal`, whether a session starts at the home screen or the command prompt |
| `set.tile` | Added | administrator: `set.tile sduser <tile> off` or `on` takes a tile off the home screen or puts it back (`editor`, `ide`, `ced`, `terminal`, `keys`; sign out cannot be turned off) |
| `set.viewer` | Added | allows one PDF reader for the `pdf` verb, by its full path or by the name this computer's App Paths knows it by (`msedge`); it must be an .exe under Program Files or the Windows folder that only the system and the Administrators can change; none is allowed as delivered |
| `sh` and `os` switches | Added | two per-account switches, both off as delivered: `sh` for the `sh` verb, `os` for `os.execute`, the Python helper, the editors `edit` and `micro`, and files and directories outside the account by path; after `admin` the session passes both, except that the `sh` verb still refuses a command with a shell metacharacter unless the `sh` switch is on; the API server reads them when it enters the account |
| `tool.audit` | Added | administrator page: the audit trail from its newest line, with filters, a go-to-date, a follow mode and the lines shown saved to a record |
| `tool.backups` | Added | administrator page: the backup directory, the archives in it with what each holds, and `backup.account`, `restore.account` and `set.backup.directory` behind a box |
| `tool.config` | Added | administrator page: every parameter of `sd.conf` with the value SD runs with, the file's value and the default; `E` changes one of the 14 through `set.config` |
| `tool.gates` | Added | full-screen page of the account's four gates (`sh`, `os`, `net`, `xfer`): on or off for this session, what the register says, how each is delivered and when a change reaches you; after `admin`, Enter turns a gate on or off through its own verb |
| `tool.menus` | Added | administrator page: the home screen's settings, changed through `set.start.default`, `set.tile` and `set.pins` behind a box answered Run |
| `tool.remote` | Added | administrator page: the two doors, the SDNet servers, the sessions now and this computer's TLS identity; `A` and `S` change a door through `remote.api` and `remote.ssh` |
| `tool.sessions` | Added | administrator page: every session and every lock, read again every three seconds; `L` logs a session out, `U` releases a lock, `I` stops or allows new sessions, each through its verb behind a box answered Run and written to the audit trail |
| `tool.viewers` | Added | the administrator's Bypass programs page and home tile: the readers `pdf` may start, and the known readers App Paths names, each allowed or stopped through `set.viewer` after a box answered Run |

### AI

| Name | Change | What |
|---|---|---|
| `agent-pack` | Added | a guide to SD BASIC for AI agents (its examples compiled and run on SD Core Solo for Linux), a generated reference of everything the product has, a Claude Code skill and a recipe for giving an agent a way in; in the source only, not installed; User page 41 and Getting Started page 23 |
| AI helper (`sd-ai.exe`) | Added | the one program that talks to the provider, in `usr\bin`; SD starts it for each request as your own Windows user (no service); its settings and keys are in `sdsys\ai`, each key encrypted with DPAPI for your Windows user, kept by an upgrade and by the uninstaller's Keep; any program running as you can use it; Getting Started page 24 |
| `ai` line | Added | an `AI>` row on the home screen, the toolkit's pages, the IDE, SED, the dictionary form, CED (F2) and the debugger (`AI`), when the AI is on: a preview first, Cancel the default, a proposal Reject by default, Follow up; SD's own files are never sent; User pages 29a, 29b, 29d, 24 |
| `ai` verb | Added | asks the AI the administrator has chosen one question: you see the provider, the model, the size and your text, and nothing is sent without `y`; the answer is printed, never run; off until a provider, its key and the AI gate are set; User page 29b |
| `set.ai` | Added | administrator only: the AI gate (`set.ai gate on`, off as delivered), the provider, model, key, Custom connection, certificate file, limits and standing text; a key typed at its prompt never passes through SD; Getting Started page 24 |
| `tool.ai` | Added | administrator only: the AI choice page and home tile (*Data, network and AI*): the provider with a check box, the key and model in one setup dialog, the gate, the limits and the standing text, every change a `set.ai` sentence behind a box answered Run; Getting Started page 24 |

### API and client library

| Name | Change | What |
|---|---|---|
| api `dict` file names | Changed | fixed: a client that opened `dict <file>` in lower case was told the file did not exist; any case opens the dictionary |
| API `filelock`, `fileunlock` | Changed | fixed: they locked the last file opened, not the file named; an `on error` reply no longer carries the text of an earlier refusal |
| api request 70, `delete.file` | Changed | fixed: one the verb refused (the file open in another session) left the client's file number bad and reported 0; the file number stays good and the verb's status is reported |
| api server loop | Changed | fixed: a client that hung up at the wrong moment, or sent a false packet length, left its server process tied up |
| client library: `sdchecksum` | Added | a checksum computed on the client |

### Encryption

Three simple layers, with no vault and nothing on by default.

| Name | Change | What |
|---|---|---|
| `encrypt.key` | Added | asks for a key on a hidden prompt and puts it in the key ring; `encrypt.key set` refuses a passphrase of fewer than 12 characters |
| key ring subroutines | Added | `!keyset`, `!keyclear`, `!keylist`: fill, empty and list the session's key ring, kept in the memory of its own process |
| `sdkey()` and friends | Added | `sdsalt()`, `sdkey()`, `sdrandkey()`: a salt, a key from a passphrase, and a random key for `sdencrypt()` and `sddecrypt()`, open to every program; every `sdencrypt()` uses a fresh random nonce <!--s 504 g116 g123--> |
| `xe` conversion | Added | a dictionary item with `xe` (or `xe;name`) shows a field decrypted for a session that holds the key and the cipher text for one that does not; `iconv` encrypts |

### File transfer

Plain-text files in and out of the account with an sftp program over Solo's ssh. Off as delivered, twice.

| Name | Change | What |
|---|---|---|
| home screen Receive files | Added | a tile that runs `receive` and waits for Enter after it, shown only where a transfer can be opened (`XFER` on and the account's switch on) |
| `receive` | Added | opens a transfer for plain-text files coming in to the account's file `xfer.in` from an sftp client signed in to Solo's ssh (port 4251) with your Windows name, waits, and says what came with each file's SHA-256; `receive overwrite` lets a file replace one of the same name |
| `send` | Added | `send <file> <id> ...` copies records that are plain text to the account's file `xfer.out` and opens a transfer for an sftp client to fetch them; the same switches and the same confinement as `receive` |
| `set.xfer.access` | Added | the account's file transfer switch: `set.xfer.access on` and `off`; off as delivered; with `XFER=1` in `sd.conf` it lets `receive` and `send` move plain-text files over Solo's ssh |
| `XFER` and the transfer limits | Added | `sd.conf`: `XFER` (0 as delivered; 1 lets the account transfer files once `set.xfer.access` is on), `XFERFILE` (10 MB a file), `XFERSLOT` (100 MB a transfer), `XFERWAIT`, `XFERIDLE`, `XFERMAX` (60 minutes) and `XFERKIND` (the endings taken; blank is any plain text); read when Solo starts |

### Files

| Name | Change | What |
|---|---|---|
| group size of a new file | Changed | `grpsize` is 4 (KB) in the shipped `sd.conf` and the built-in value when it is absent; it was 2 and 1. An installed Solo keeps its `sd.conf`, and a file keeps the size it was made with |
| index on a right justified item | Changed | a right justified index made now keeps its keys in one order: the empty key, numbers by value (decimals too), then text, so 1, 01 and +1 are one key and `build.index` loses no id. An index made before keeps the old order; an index made on SD Core for Linux or its Solo since 25 Oct 2026 now opens here, and an index made here now cannot be opened by an older release |
| `sdfix`, `sdidx`, `sdconv` | Changed | fixed several faults (see the changelog); `sdidx -m` says why it cannot make the directory; `sdidx` refuses a link put in place of a file's primary subfile instead of following it |
| sort of equal or ordered keys | Changed | fixed: a long run of equal or already ordered keys took minutes; now a fraction of one |

### Installer

| Name | Change | What |
|---|---|---|
| Shell and OS access page | Added | the installer asks, on a new installation and on an upgrade, with two boxes, both unticked; the control file's `os-access=` (`none`, `sh`, `os` or `both`) answers it on a new installation; a silent installation, or an unticked page, turns nothing on |
| source | Added | the complete source (C, `gpl.bp`, build and installer scripts) is `SDCore4WindowsSolo-source-<version>.zip` beside the installer; none of it is installed |
| Start Menu entry | Added | the install puts SD Core Solo in your own Start Menu, opening a classic console window; the uninstaller removes it |
| start question | Added | on a new installation, one unticked box: start new sessions at the command prompt instead of the home screen (`set.start.default terminal`); the control file's `start=` (`menu` or `terminal`) answers it; an upgrade keeps the start the installation has |
| PDF reader page | Added | where a known PDF reader is installed and none is allowed, one unticked box: allow it for the `pdf` command (`set.viewer add pdf <name>`); the control file's `viewer=` answers it; a silent installation allows nothing |
| API and ssh boxes | Changed | the "other computers" boxes use SD Core for Windows' words: *Let other computers on your network reach it* (the API) and *... connect to this one over ssh (port 4251)*; as delivered the API is still off and ssh answers this computer only |
| uninstall Keep | Changed | keeps what SD Core keeps with its accounts: the passwords, the audit trail, the TLS key, your ssh key file and the ssh host key, the SDNet logins, `newvoc.mods`, the saved VOC records and where sessions start; the question is asked before anything is removed |
| reload of kept data | Changed | puts all of it back (the audit trail joined, kept lines first) and asks no password and no start question when the kept passwords are complete; passwords in the control file replace the kept ones |

### Screen library and terminals

The screen statement and its library, and what changed in the terminal handling around it. The library is Linux's code carried here; it has not yet been run on a Windows console.

| Name | Change | What |
|---|---|---|
| alternate screen in SD's window | Added | in SD's own window a page opened with `screen open clear` (the home screen, the toolkit's pages, the IDE, CED) is drawn in the console's alternate screen, which Windows does not rewrap, so dragging the window wider or narrower no longer turns the page to garbage; leaving the page brings back the lines from before it; not over ssh |
| `ced`, `ide`, `sed` | Added | `ide` is SD Core's editor: tabs, record locks, an OS-file tree, breakpoints handed to the debugger, a single-record mode, undo and redo; `sed` opens it (the old SED program is gone); `ced` edits a data collection; they save into the VOC only after `admin` |
| `debugger` full-screen display | Added | on a terminal that reports its size and has an alternate screen the debugger takes the whole screen; F1 help, F4 the program's screen; a terminal without one (the `windows` type today) keeps the line-by-line debugger |
| development pages | Added | `tool.files`, `tool.voc`, `tool.dict`, `tool.query`, `tool.bulk`, `tool.catalog` and `tool.test`, each a tile of the home screen |
| full-screen pages | Changed | every page has the same frame: a menu bar (Esc or F10), F6 and Shift-F6 between the boxes, Ctrl-T the theme, Ctrl-K the keys, F1 the help of the box with the keys of that box above it; a larger window is used whole |
| home screen | Added | a tile whose action is `menu:name` opens a page of its own with a Back tile (Esc, Backspace go back); **View > Recent group** keeps the tiles started last, off as delivered; `set.tile sduser recent off` forbids it |
| home screen | Changed | a menu bar, boxed tiles in the paper and night themes, and the development pages' tiles; a home screen that cannot open (a window too small) signs the session out when the account starts there |
| home screen version | Added | the header's date line ends with the product and its version (`SD Core Solo for Windows WS2.0-0`) |
| immediate messages | Changed | `message ... immediate` is shown in a box over the alternate screen, or as lines and `Press return to continue` |
| `screen ask` history | Added | `history list` gives earlier answers: Up and Down step through them and Ctrl-R brings back the newest; the SD pages open a question empty and pass the value in force this way |
| `screen` controls and keys | Added | `slider`, `selector`, `container`, `viewport`; `color` and `accel` on the controls; the letter marked with `&` in a button, check box, slider, selector or label is its Alt-letter; `keycode()` can return an Alt-letter (`bindkey(, -7)`, off by default) |
| `screen form fill`, `stretch` | Added | a form that takes the whole window and follows it: a text area or table with `stretch` gets the spare room and what lies beyond it moves, with no code in the program; below the form's own size one warning holds the keyboard |
| `screen` statement | Added | menus, dialogues and forms from BASIC; tabs, text areas, tables and trees; scroll bars and progress bars; the mouse |
| `sp.view` | Changed | on a window of 140 by 43 or more, the pages' look: a menu bar, the theme, Ctrl-K the keys; F5 or Ctrl-L redraws |
| themes | Added | `paper` (the delivered one: a new install's `sd.conf` says `SCREENTHEME=paper`), `night`, `mono`, `classic` and `blue`; `screentheme=` in `sd.conf` chooses the default (no line means `mono`); a colour theme is refused where the terminal cannot show colour |
| `!tkconfirm` typed word | Changed | the word a box asks for (`open`, `restore`) is taken in either case, as any command is, and the pages show it in lower case |
| `!tkform` fields | Changed | every entry opens empty, its starting value shown beside it (`now: ...`) and put in by Ctrl-R, and a field may keep that value when left empty; a field can be a choice of fixed words, which the pages use wherever a fixed set was typed |
| `tool.keys` | Added | a diagnostic for anyone: the terminal type, window, color and mouse, the name every key has under this terminal type, and the raw bytes of a key the type does not name (Ctrl-R) |
| toolkit window size | Changed | a page of the full-screen toolkit refuses a window of more than 500 columns by 300 lines before it takes the screen (opening one cost time as the square of the height); "Press Enter to return" no longer names the home screen |
| window size | Changed | SD reads the terminal's window size at the start and when it is resized; `kernel(k$winsize)` answers it; `term w,h` holds until the next resize |

### SDNet

Files on another SD server. All of it is off as delivered. The client is Linux Solo's, carried here; it is compiled and its
free checks pass, and it has not been run against another server.

| Name | Change | What |
|---|---|---|
| `list.servers`, `delete.server` | Added | the administrator's list of defined servers, and removal of one <!--s g52 g26--> |
| `net` gate | Added | `set.net.access on`: one of the four things that must all be true before a remote file opens (the gate, a defined server, your login on it, a VOC pointer) |
| `netwait`, `netreply` | Added | `sd.conf` settings, 1 to 3600 seconds, for a connection and a reply; a silent server is given up on <!--s 235 g291--> |
| `open.socket()` | Changed | the fourth argument is the number of seconds to wait for the connection, zero for no limit of SD's own; it was the unused security context <!--s 174--> |
| remote files, a pointer | Added | a VOC entry whose first field is `n` points to a file on another server |
| remote file administration | Added | on a remote file: `analyze.file`, `configure.file`, the index verbs, `fcontrol()`, `write ... creating.seqkey`; `create.file name ... server srv {as remote}` makes the file there and the pointer here, and `delete.file` of a pointer deletes both; the other machine decides (a Solo server serves them only to a session signed in with the global password) |
| remote files, cache | Added | a short record cache; `logto` closes the connections |
| remote files, index | Added | `indices()`, `selectindex`, `setleft`, `setright`, `selectleft`, `selectright` ask the other machine, so a query on a remote file uses its indices |
| remote files, query | Added | `list`, `count`, `sort`, `select`, `list.item`, `list dict` on a remote file; `fileinfo` type 7 |
| remote files, refusal | Changed | the server's own words are shown, not `status 0`; a lost connection is an error <!--s 256--> |
| remote files, reply checked | Changed | a reply that cannot be true is a lost connection at once |
| remote files, write | Added | for a server defined `write`: `write`, `delete`, `clearfile`, the locking reads and release |
| remote lock functions | Added | `recordlocked()`, `getlocks()`, `filelock`, `fileunlock`: work on a remote file |
| remote `select` | Changed | fixed: a server error stopped the program with an abort |
| SDNet logins | Added | `set.netlogin`, `delete.netlogin`: a person's login on a server, kept as derived values and never the password |
| `set.server` | Added | defines a server; `connect` shows its key hash for you to confirm <!--s g84 147--> |
| `settings.report` | Changed | the `[api]` section shows the hash of the API's public key, the figure another administrator pins |
| tls reads | Changed | fixed: a half record or a late ticket made a read wait without limit |

### Security

Fixes that close a way round a gate, and what the gates now ask.

| Name | Change | What |
|---|---|---|
| api `clearfile` | Changed | fixed: it cleared whichever file a stale counter pointed at instead of the file the client named |
| api request 29 | Changed | fixed: it opened a file for any login once `netfiles` bit 2 was set; it now needs the account's `net` gate (`set.net.access on`), and a VOC pointer that leads out of the account needs the `os` switch |
| api sign-in: audit | Changed | writes `api login user= account=` to the audit trail |
| audit trail, control characters | Changed | fixed: a value written to the audit trail that held a newline started a false line; a control character, DEL or a mark is now written as `?` |
| commands that change a file you name | Changed | `get.list ... deleting`, `save.list`, `delete.list`, `sort.list`, `copy.list`, `map file`, `format`, `fixdir`, `generate`, `compile.dict data`, `set.trigger`, `reformat ... to`, the `pageseq` clause of a report, `sp.view` (deleting the item shown), `como ... in`, `basic`'s object file, `create.file` on an existing file, `import.csv`, `import.json` and `import.xml` into a VOC, and over the API `write`, `delete` and `clearfile`, each now ask before they change anything and say why when they refuse |
| deleting a directory tree | Changed | fixed: a link inside a tree being deleted (`delete.file` and the rest) is removed as a link; a link to a directory was walked and everything under its target deleted; a pathname of more than 255 characters fails instead of overrunning its buffer |
| global password | Changed | one set after the account password now shares the account's salt, so an SD Core server can sign in over the API under the account's name with it (set it again to give an older one the shared salt); it is no longer accepted under the name SDSYS |
| hang-up and fault | Changed | fixed: a session whose console closed while it was writing the error log or the audit trail waited for itself for ever, and every other session behind it; the "Received termination signal" line is now written outside the signal, and a session that faults gives back its locks before it logs |
| `kernel(k$internal, n)` | Changed | only a program compiled in internal mode may set internal mode; reading it (`-1`) is unchanged |
| `list.readu`, `pstat`, `getlocks()`, `system(1083)` | Changed | record ids, paths, commands and program names another session wrote are shown by `list.readu`, `pstat`, `profiler` and `hsm` with their control characters as `?`, so a record id cannot send codes to your terminal |
| named common with a `$` or `_` name | Changed | the system's own common blocks: the kernel refuses one to a program that is not a system program |
| `openseq` that fails | Changed | fixed: a record lock it had taken stayed until SD was restarted; it is now given back |
| operating system paths | Changed | `openseq`, `openpath`, `deleteseq`, `dir()`, `digest()` of a file, the file-system keys of `ospath()`, a print file named with `setptr ... as pathname` or `setpu`, a CSV report's `to` pathname and a VOC pointer to a file outside the account each ask the account's `os` switch; inside the account's own directory nothing changes, except that the VOC's own files are read only there without `admin` or the `os` switch (`openseq` cannot create one, `deleteseq` and a print file are refused) |
| paths with `..` | Changed | fixed: a path with a folder that does not exist followed by `..` kept the `..`; it now resolves to the right place |
| phantom debugger messages | Changed | `pdebug` and the phantom it debugs leave each other their messages in `sd\pdebug` in the `AppData\Local` folder of the user's profile (a private folder, made with access for that user, SYSTEM and Administrators only), not in `$ipc`; message 12953 when the store cannot be set up |
| phantom start-up records | Changed | `$ipc` is a file every SD user can write, so the records a new phantom runs, stacks and sets (command, `DATA` lines, log file, options) are sealed with a secret given only to the new process in its environment; a phantom whose records do not check runs nothing, says so and writes `phantom refused` to the audit trail |

### System

| Name | Change | What |
|---|---|---|
| `autologout` | Changed | counts the time a full-screen page spends waiting, so such a page no longer keeps a session alive for ever |
| classic console window | Changed | a full-screen page took 3 to 5 seconds to draw in the window Windows PowerShell or cmd opens; SD now writes to the console directly and it draws as quickly as in Windows Terminal |
| F1 at the prompt | Changed | fixed: names the changelog in the SD Core Solo folder, not in its `sdsys` folder |
| failed start | Changed | fixed: an `sd.exe` that stopped early ("SD has not been started.") kept an internal lock, and every session started after it hung; the lock is given back |
| kernel faults | Changed | fixed: `getlocks()` with a user number nobody holds read outside the user table; a time conversion with negative parts or long hours overran its buffer, and a prefix strip in a conversion copied over itself |
| phantom of an ended session | Changed | a phantom whose session ended before it started ends at once |
| `sd -status` | Changed | fixed an overrun on a server that recorded no start time |
| `sd-solo -start` | Changed | a script that captures its output, or runs a session whose phantom outlives it, gets its end of file: the background process no longer holds the pipe |
| `sd-solo -stop` and `sd-solo -restart` | Changed | an SD process still running ten seconds after the termination signal is ended with SIGKILL, and the stop says how many; before, it was left running with SD gone under it |
| `sd-solo` window | Changed | typed at a PowerShell or cmd prompt it opens a classic console window of its own, 140 by 43 to start, and the prompt comes back at once; one that started in a smaller classic console window makes it 140 by 43 (Windows Terminal is left alone); after an error, such as a refused sign-in, the window waits for Enter, and a refused sign-in exits with status 1 (it was 0) |
| session with no terminal | Changed | its display is 140 by 42 (it was 120 by 36); `term default` gives the same |
| small fixes | Changed | `execute ... capturing` of a command whose whole output is one newline no longer risks a crash; a failure to extend an index file is reported; an empty TMP is treated as unset; `run` of a program whose path is too long says what the limit is |
| terminal size | Changed | SD asks the terminal for its size at the start and when it is resized, instead of using the terminal type's 120 by 36 |

## NO CHANGE

Items on the openQM list that were looked at for this release and needed no change to SD: the defect is not in SD, the
feature was already there, or it does not apply. They are here so that nobody looks for them above.

| Name | Change | What |
|---|---|---|
| `*` comment line in a paragraph | None | already works <!--s g2--> |
| 64-bit versions | None | no change in this release <!--s 196--> |
| `$echo` | None | already there in W1.1-3 <!--s g1--> |
| `formlist` bad data | None | no change recorded <!--s 265--> |
| hangup at the break key prompt | None | already ends the process by itself <!--s 266--> |
| inherited public variable | None | beside a function of the same name: already found <!--s 342--> |
| `insert` appending a field | None | `insert`, `ins` and `<-1>`: no change recorded <!--s 475--> |
| `itype()` | None | no change recorded <!--s 328--> |
| `list.users` | None | removed in an earlier release with the other user verbs; the administrator uses `listu` <!--s g53--> |
| `locate` insertion position | None | also a very long numeric item: no change recorded <!--s 527 556--> |
| `loop` ... `repeat` | None | already there in W1.1-3 <!--s g59--> |
| `matbuild` trailing elements | None | unassigned elements at the end: no change recorded <!--s 557--> |
| `matchfield()` `n-mx` pattern | None | at the start of the template: no change recorded <!--s 562--> |
| memory alignment in `redim` | None | no change recorded <!--s 382--> |
| negative default argument | None | for an optional argument: the defect cannot occur, because the feature it belongs to was not in SD <!--s 324--> |
| Pick wildcards in `ne` | None | no change recorded <!--s 322--> |
| `qmbasic.warnings` | None | not built: SD's own shortcut of the kind, `option sdbasic.warnings`, was removed in this release (see REMOVED) <!--s g251--> |
| `$qmcall` | None | already there as the `$sdcall` flag, which lets a restricted client session call a subroutine <!--s g212--> |
| `readv` fatal error | None | no change recorded <!--s 559--> |
| record cache and case | None | no change recorded <!--s 330--> |
| `set.date` | None | SD still has the command; nothing was withdrawn <!--s 78--> |
| the startup process | None | not done: SD has no startup command, so there is no startup process to name <!--s 91--> |
| the `u` format in a report | None | already wraps a long value as `t` and `l` do <!--s 531--> |
| the user limit reached | None | login prints message 1000 and exits cleanly <!--s 275--> |
| `update.account` | None | already there in W1.1-3 <!--s g102--> |
