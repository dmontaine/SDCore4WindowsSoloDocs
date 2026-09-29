Title: Development and file commands
Subtitle: Compiling, editing, and the verbs that maintain files, indexes and records in bulk.

**You have all of these, and none needs `ADMIN`.** This page is a reference for
what each does. Two things near them are gated, and are named where they come
up: **the global catalogue**, which nobody changes from your account, and
**editing the VOC directly**, which needs `ADMIN`.

## Compile, catalogue and run

| | |
|---|---|
| **`basic`** | compile SD BASIC source |
| **`catalog`** · **`catalogue`** | add to the catalogue |
| **`delete.catalog`** · **`delete.catalogue`** | remove from it |
| **`compile.dict`** | compile dictionary items |
| **`run`** | run a compiled program |
| **`map`** | show a program's map |
| **`generate`** | generate source |
| **`phantom`** | start a background process |

> **Catalogue locally or privately; the global catalogue is not yours.**
> `catalog` and `delete.catalog` work for local and private entries.
> `catalog ... global`, a name beginning `*`, `!`, `_` or `$`, and
> `delete.catalog` of a global entry are refused for every session, `ADMIN`
> or not: *The global catalogue holds the SD Core server's programs from
> GLOBAL.BP.OUT and is changed only by SYNC.GLOBAL.CATALOG*. On a managed
> computer the global catalogue holds the SD Core for Linux server's programs,
> which you can call; see [Managed mode](15-managed-mode.html).

## Edit and debug

| | |
|---|---|
| **`ed`** | the line editor |
| **`edit`** | a **full-screen** editor — opens the record in Microsoft Edit |
| **`micro`** | a **full-screen** editor — opens the record in micro |
| **`debug`** | the BASIC debugger |
| **`pstat`** · **`pdebug`** · **`pdump`** · **`dump`** | process introspection |

### Editors

**There are two, and they behave identically.** The verb chooses the editor
and nothing else changes:

| | |
|---|---|
| **`edit`** | **Microsoft Edit** |
| **`micro`** | **micro** |

**Both come with SD**, in `%USERPROFILE%\SDCoreSolo\usr\bin`, so nothing has to
be installed or downloaded for them.

```
edit  bp myprog
micro bp myprog
edit  dict customers name
```

Either verb writes the record to a working copy, opens the editor on it,
reads it back, and asks whether to save. For a `bp` record it then offers
the compile and the catalogue.

**Both are terminal editors**, so both work over ssh as well as at the
keyboard. **A session with no terminal is refused** — an API session or a
piped script has nowhere to draw a full screen, and is told so.

**Only `micro` highlights SD BASIC.** Microsoft Edit has no syntax
highlighting at all, which is the one real difference between the two
verbs:

| | |
|---|---|
| **`micro`** | statements, reserved words, intrinsic functions, `@variables`, `$directives`, labels, strings, numbers and comments |
| **`edit`** | plain text |

**It applies to a `bp` record and to nothing else.** SD names the working
copy so that micro can recognise the language — a record edited out of any
other file is treated as plain text, which is correct for a VOC entry or a
data record.

> **The word lists are generated from the compiler.** They come out of
> `BCOMP`'s own tables — **218 statements, 37 reserved words and 176
> intrinsic functions** — so the highlighting cannot drift from the
> language. **If a name you expect is not coloured, that is worth
> reporting.**

### What the editors are good for, and what they are not

**They are text editors**, so they suit a record whose content is lines of
text: BASIC source, VOC records, simple dictionary records, and data records
with multivalues or subvalues.

**A field is a line.** SD writes the working copy with one field per line, so
moving between fields is moving between lines.

**A value mark is not a line, and neither is a subvalue mark.** Both are
control characters an editor cannot show, so each has a token you can type:

| Type | To get |
|---|---|
| `~~` | a **value** mark |
| `` ~` `` | a **subvalue** mark |

SD converts marks to tokens on the way into the editor and tokens back to
marks on the way out.

```
SMITH~~JONES~~BROWN
```

is a three-value field, and

```
RED~`BLUE~~GREEN
```

is two values, the first of which has two subvalues.

**A record that cannot be written this way is refused, not mangled.** Some
records would come back different from how they went in — one that already
contains `~~` as data, for instance, or one with a `~` sitting immediately
before a mark. Before opening the editor, SD converts the record and converts
it back; **if the result is not what it started with, the verb refuses and
names `ed`**, which needs none of this. Text marks are not converted, and are
covered by the same refusal.

**A compiled dictionary record is truncated to its first 15 fields** while
you edit it, and recompiled with `cd` when you save.

**Editing the VOC needs `ADMIN`**, whichever editor does it — see
[Administrator commands](06-administrator-commands.html).

### What an editor can reach

**An editor can open any file your Windows user can**, inside the
`SDCoreSolo` folder or outside it. Neither can run a command, so neither is a
shell. On a Solo computer that is no more than you can do anyway; it matters
only for who you let use your account — see [Security](12-security.html).

### Over ssh

**A terminal editor is the point of an ssh session.** SD hands the editor the
session's terminal rather than reading it through a pipe. **If an editor
misbehaves over ssh and not at the keyboard, that is worth reporting** with
the terminal you connected from.

The removed full-screen editors `sed`, `update.record` and `modify` are gone
and are not coming back. See [Not in SD Core](14-not-in-sd-core.html).

## Files

| | |
|---|---|
| **`create.file`** · **`delete.file`** · **`clear.file`** | the life of a file |
| **`configure.file`** | change a file's configuration |
| **`analyse.file`** · **`analyze.file`** | report on a file's internals |
| **`fstat`** | file statistics |
| **`hsm`** | hashed-file statistics monitoring |
| **`set.trigger`** | attach a trigger |
| **`cd`** | change directory |

`create.file` writes the file's entry into the VOC for you; that side effect
needs no `ADMIN`.

## Indexes

**`create.index`** · **`delete.index`** · **`build.index`** · **`make.index`** · **`list.index`**

## Bulk record editing

| | |
|---|---|
| **`copy`** · **`copyp`** | copy records |
| **`delete`** | delete records |
| **`rename`** | rename records |
| **`reformat`** · **`sreformat`** | reformat |
| **`sort.item`** | sort |
| **`cname`** | change a record's name |
| **`delete.common`** | clear a common block |

**Copying into the VOC, or deleting from it, needs `ADMIN`**, like any other
direct VOC edit.

## On a managed computer

**Any of these can be on the list of commands you may not run** without
`ADMIN`; the SD Core for Linux server keeps that list. See
[Managed mode](15-managed-mode.html).

## Two things to know when you compile

**`basic` no longer creates an object file it can never open again.**
Compiling into a reused file name previously produced an object SD could
not subsequently open.

**SD's own compiled programs are replaced on upgrade; yours are not.** A new
release overwrites SD's system programs, messages, include records and VOC
templates. Your `bp`, `bp.out` and everything else in your account are left
alone.
