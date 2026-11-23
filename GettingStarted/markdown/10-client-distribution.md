Title: Client distribution
Subtitle: Which library an application needs, why there are two names, and the one file no installer can update.

An application reaches SD Core Solo for Windows through a client DLL. **The installer puts the
clients on the computer**, in the folder below, and they are the same libraries
whichever SD Core they connect to; there is no client-only package.

## Two names, one library

One 64-bit library, under **two names from one build**:

| Names | For |
|---|---|
| `sdclilib.dll` · `sdclient.dll` | new work, and existing SD applications |

**`sdclilib.dll` is the name existing applications ask for and it never
moves.** `sdclient.dll` is for new work.

> They are second **links**, not file copies. An import library records the DLL
> name its symbols come from, so a renamed copy would send the application back
> to the original.

## No 32-bit client

**SD Core Solo for Windows ships no 32-bit client library.** The architecture
must match the application, not the computer, so a 32-bit application cannot
load the 64-bit DLL; it has to be rebuilt as a 64-bit program.

**A program written for QM does not work with SD out of the box**, whatever its
architecture: the login, the network session and the server's rules have
changed (below, and [API access](09-api-access.html)), so such a program needs
changing to work with SD.

## Where they are after an install

Under `%USERPROFILE%\SDCoreSolo`:

| | |
|---|---|
| `usr\clients\client64\` | `sdclilib.dll`, `sdclient.dll`, and the import libraries (`libsdclilib.dll.a`, `libsdclient.dll.a`) to link against |
| `usr\bin\` | the DLL pair again, beside `sd-solo.exe`. That folder is on your PATH |

**The C header is installed, but not beside the DLLs**: `sdclilib.h` is in
`%USERPROFILE%\SDCoreSolo\sdsys\syscom`, with the language bindings that ship:
`sdclient.py` (Python), `sdclient.cs` (C#), `sdclient.php` (PHP),
`sdclient.module` (Gambas), `sdclilibbind.pas` (Lazarus and Free Pascal),
`sdclient.bas` (Visual Basic) and `sdclient.pb` (PureBasic).

If you need a client on a computer that has no SD Core, copy the pair from
`usr\clients\client64` on one that does.

## The one file no installer can update

**An application that carries its own copy of the client** keeps it beside
its executable, and **Windows searches an executable's own directory before
`PATH`** — so no installer entry and no `PATH` change will ever update it. It
has to be replaced by hand.

**This is the most likely way to test an old client without realizing it.**
If an application cannot log in after an upgrade, check the date of the client
DLL beside its executable before anything else.

## The library must speak SCRAM

**There is no cleartext API login.** A client that sends a password in
clear is refused with *"Cleartext login is no longer supported; this server
requires SCRAM authentication"*. Use the client library that comes with SD.

## Connecting

| | |
|---|---|
| `SDConnect()` | over the network, to port **4249** - name it; the library, SD Core's too, takes -1 as SD Core's 4247 - as `sduser` with the account password. It is the only way to connect |
| `SDConnectLocal()` | **does not work.** It returns false at once, with `SDError()` saying why: it starts the server program beside the DLL, which Solo does not have under that name (Solo's is `sd-solo.exe`) |

**To reach this computer from a program on it, call `SDConnect()` with
`127.0.0.1`.** The API must be switched on; see [API access](09-api-access.html).

BASIC programs reaching another SD server use the `!sdclient` class, which
speaks the same login — `connect()` takes the same arguments, but **the
program has to be running under this release at both ends.**

What a connected session may open, and what it runs as, is on
[API access](09-api-access.html).

## Where the library comes from

The client library is a **clean-room** library, written without the original
GPL client's source, so a commercial program may link to it. It is the same
library for every SD Core. **Its source is not published and it is not built
from this repository**: the DLLs are in the release package, in
`clients\Windows64` beside the installer, each listed with its SHA-256 in that
folder's `COMMIT.txt`, and the installer copies them from there. An installer
run with no `clients` folder beside it installs no client library.
