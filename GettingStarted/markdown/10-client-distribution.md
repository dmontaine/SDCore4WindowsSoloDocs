Title: Client distribution
Subtitle: Which library an application needs, why there are four names, and the one file no installer can update.

An application reaches SD Core through a client DLL. **The installer puts the
clients on the computer**, in the folder below, and they are the same libraries
whichever SD Core they connect to.

## The four names, and why

Two builds, each producing its DLL under **two names from one source in one
build**:

| Build | Names | For |
|---|---|---|
| 64-bit | `sdclilib.dll` · `sdclient.dll` | new work, and existing SD applications |
| 32-bit | `qmclilib.dll` · `qmclient.dll` | QM applications |

**The `*clilib` names are what existing applications ask for and they never
move.** `qmclilib.dll` in particular is the original QMClient library name —
it is **how an unmodified QM application finds its client at all**, so it is a
name rather than a label and it is not going to be renamed.

The `*client` names are for new work.

> They are second **links**, not file copies. An import library records the DLL
> name its symbols come from, so a renamed copy would send the application back
> to the original.

## Which one does my application need?

| | |
|---|---|
| A 64-bit application | `sdclient.dll`, or `sdclilib.dll` if it already asks for that name |
| A 32-bit application | `qmclient.dll`, or `qmclilib.dll` if it already asks for that name |

**The architecture must match the application, not the computer.** A 32-bit
application on 64-bit Windows needs the 32-bit DLL. **The 32-bit build is a
shipping deliverable, not a testing convenience.**

## Where they are after an install

Under `%USERPROFILE%\SDCoreSolo`:

| | |
|---|---|
| `usr\clients\client64\` | `sdclilib.dll`, `sdclient.dll`, and the import libraries (`libsdclilib.dll.a`, `libsdclient.dll.a`) to link against |
| `usr\clients\client32\` | `qmclilib.dll`, `qmclient.dll`, and their import libraries |
| `usr\bin\` | **both** DLL pairs again, beside `sd.exe`. That folder is on your PATH, so it is where a 32-bit utility finds its client at run time |

**The C header is installed, but not beside the DLLs**: `sdclilib.h` is in
`%USERPROFILE%\SDCoreSolo\sdsys\syscom`, with the Gambas and PureBasic bindings,
`sdclient.bas` and `sdclient.pb`. The other language bindings are in the
source repository.

## The one file no installer can update

**An application that carries its own copy of the client** keeps it beside
its executable, and **Windows searches an executable's own directory before
`PATH`** — so no installer entry and no PATH change will ever update it. It
has to be replaced by hand.

**This is the most likely way to test an old client without realising it.**
If an application cannot log in after an upgrade, check the date of the client
DLL beside its executable before anything else.

## The library must match the release

**The cleartext API login is gone.** A client that still sends a password in
clear is refused with *"Cleartext login is no longer supported; this server
requires SCRAM authentication"*, and one that predates SCRAM is refused
outright. **Use a client library from this release or later** — and confirm
which build you are actually loading, because a stale 32-bit client is the
usual culprit.

## Connecting

| | |
|---|---|
| `SDConnect()` | over the network, to port **4243**, as `sduser` with the account password. It is the only way to connect |
| `SDConnectLocal()` | **disabled.** It answers *SDConnectLocal is not available in SD Core Solo for Windows - connect with SDConnect and the account password* |

**To reach this computer from a program on it, call `SDConnect()` with
`127.0.0.1`.** The API must be switched on; see [API access](09-api-access.html).

BASIC programs reaching another SD server use the `!sdclient` class, which
speaks the same login — `connect()` takes the same arguments, but **the
program has to be running under this release at both ends.**

What a connected session may open, and what it runs as, is on
[API access](09-api-access.html).

## Building from source

**One repository builds all four DLLs**:
<https://github.com/dmontaine/SDCore4WindowsSolo>. The client source is in it,
at `sdb_ai/sd64/gplsrc/sdclilib`, and `make sd` produces the 64-bit and 32-bit
pairs together with the server. **No built DLL is committed**, so a clone
builds — the same no-binaries rule the whole repository follows.

> **A note on how the 32-bit client is built, because it constrains changes to
> it:** the DLL must stay a **single self-contained file that can be copied
> next to an application** — hence static linking of the compiler runtime, and
> hence a preference for Windows' own `bcrypt.dll` and `crypt32.dll` over
> third-party crypto libraries. Any change to the client has to keep working in
> a 32-bit process.

**There is no client-only installer yet.** What exists is what the section
above describes: the SD Core Solo for Windows installer places all four DLLs.
If you need a client on a computer that has no SD Core, copy the pair you
need from `usr\clients` on one that does.
