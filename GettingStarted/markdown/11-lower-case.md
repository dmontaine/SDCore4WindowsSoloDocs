Title: Lower case
Subtitle: Commands, file names, record ids and the account name are lower case — and nothing you type has to change.

**Everything that can be lower case is lower case.** SD used to be inconsistent
about it: BASIC source is free-form and usually written lower case, while file
names, field names and account names were forced up.

**What you type does not change.** `LIST`, `list` and `List` all run the same
verb, and so does every keyword — `with`, `by`, `no.page` and the rest.

## The lookup rule

SD tries a name **as you typed it, then in lower case, then in upper case**.

```
as typed  →  lower  →  upper
```

A name that matches exactly still wins, so nothing that works today changes. A
name that exists in **no** case is still reported as not found.

This order is used everywhere: the parser, the query processor, `RUN`,
multifile resolution, the `LOGIN` paragraph lookup, and `SET.FILE`'s default
`qfile` pointer.

## What is spelled in lower case

| | |
|---|---|
| Commands in the VOC | **`list`**, `count`, **`select`**, **`create.file`**, **`setptr`** … and that is how they appear in `list voc`, `listv` and `ct voc` |
| System files on disk | `accounts`, `bp`, `bp.out`, `messages`, `newvoc`, `pcode.out`, `syscom`, `voc` and the rest under `sdsys` |
| The hold file | `$hold` |
| The saved select list file | `$savedlists` |
| The command stack record | `$command.stack` |
| The account | `sduser`, on disk in `user_accounts\sduser` and in `sdsys\accounts` |
| Files in the account | created with lower-case names on disk |

**Renaming these is cosmetic for resolution** — NTFS matches without being
asked — but the stored path text is user-visible through `listf` and the
current-directory reporting, which is the point.

## Record ids in directory files are not case sensitive

Two changes that go together:

- **Record ids in directory files** are matched case insensitively.
- **Queries against a directory file** match ids the same way.

**`create.file`** also takes a `no.case` option, which creates a file whose record
ids are treated as case insensitive: SD writes records preserving the casing
given by whatever performs the write, and reads locate records regardless of
casing.

## One correction worth reading

**`LIST` and `CT` used to disagree about the same name.** `list voc $HOLD`
answered *"'$HOLD' not found"* on the very record `ct voc $HOLD` had just shown
you. `LIST`, `SORT`, `SELECT` and the rest of the query language now use the
same as-typed → lower → upper order as everything else.

## The account name

**The account is `sduser`, and the name is never case sensitive**: `SDUSER`,
`sduser` and `SdUser` all mean it. It is `sduser` on every computer, however
your Windows user name is spelled or cased.

## After an upgrade

**An upgrade adds to your VOC and never removes.** If your account's VOC holds
a record under an older upper-case spelling, `update.accounts` adds the
lower-case one beside it, so you may hold both. That is harmless — they
dispatch to the same programs.

## Windows user names and other languages

**Case folding that depends on the computer's language is a trap**, and SD
avoids it: on a Turkish or Azeri system Windows turns `I` into a dotless `ı`,
so a name folded by the computer's rules would not match itself. The one
place Solo writes your Windows user name in lower case is the ssh
configuration — the `Match User` line, see [ssh access](08-ssh-access.html) —
and it folds it the same way on every computer. The account name never
depends on it, because it is always `sduser`.

> **If you are testing on a Turkish or Azeri computer**, check that your ssh
> sign-in lands in SD when your Windows user name contains an `I`. The fold
> is fixed in the source, and has been measured on a Turkish culture setting
> but not yet on a Turkish computer.

## A related refusal that no longer depends on case

**`delete.file`**'s refusal to delete `voc` and `$acc` no longer depends on the
case you type. It could not be got round before, because those names were upper
case — **it could have been, once they are not.**
