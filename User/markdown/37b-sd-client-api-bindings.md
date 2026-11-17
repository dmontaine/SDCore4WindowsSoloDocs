Title: SD Client API: Language Bindings
Subtitle: The headers for eight languages, and what the API cannot do.

This page continues [SD Client API](37-sd-client-api.html).

## Language bindings

The API headers define bindings for eight languages. The function
signatures are identical in meaning; the syntax differs. **The installation
carries all of them**, in `sdsys\syscom`: the C header `sdclilib.h`, Visual
Basic (`sdclient.bas`), Gambas (`sdclient.module`), PureBasic
(`sdclient.pb`), Free Pascal (`sdclilibbind.pas`), Python (`sdclient.py`), C#
(`sdclient.cs`) and PHP (`sdclient.php`).

### Gambas3

```
Library "./sdclilib"
Extern SDConnect(Host As String, Port As Integer, UserName As String,
  Password As String, Account As String) As Boolean
Extern SDRead(FileNo As Integer, Id As String,
  ByRef Errno As Integer) As String
```

### PureBasic

```
PrototypeC.l P_SDConnect(host.p-utf8, port.l, user.p-utf8,
  pass.p-utf8, account.p-utf8)
PrototypeC.i P_SDRead(fno.l, id.p-utf8, *err)
```

Per-arity prototypes are provided for `SDCall` and `SDCallx`:
`P_SDCall0` through `P_SDCall20`, `P_SDCallx0` through `P_SDCallx20`.

### Free Pascal / Lazarus

```pascal
type
  TSDConnectFn = function(Host: PAnsiChar; Port: LongInt;
    User, Pass, Account: PAnsiChar): LongInt; cdecl;
  TSDReadFn = function(FileNo: LongInt; Id: PAnsiChar;
    Err: PLongInt): PAnsiChar; cdecl;
```

A `LoadSdCliLib` function loads the library and resolves all
function pointers; `UnloadSdCliLib` releases it.

### Python (ctypes)

```python
import ctypes
_lib = ctypes.CDLL('./sdclilib.dll')

SDConnect = _lib.SDConnect
SDConnect.argtypes = [ctypes.c_char_p, ctypes.c_int,
    ctypes.c_char_p, ctypes.c_char_p, ctypes.c_char_p]
SDConnect.restype = ctypes.c_int

SDRead = _lib.SDRead
SDRead.argtypes = [ctypes.c_int, ctypes.c_char_p,
    ctypes.POINTER(ctypes.c_int)]
SDRead.restype = ctypes.c_char_p
```

Per-arity `CFUNCTYPE` definitions are provided for `SDCall` and
`SDCallx` wrappers.

### C# and .NET (P/Invoke)

`syscom\sdclient.cs`:

```csharp
using static SDClient;

if (SDConnect(Z("server"), 4249, Z("user"), Z("password"), Z("account")) != 0)
{
    int fno = SDOpen(Z("CUSTOMERS"));
    string rec = Str(SDRead(fno, Z("1001"), out int err));
    SDDisconnect();
}
```

**Strings are bytes.** SD strings carry the marks `0xFB` to `0xFF`, and a UTF-8 conversion by .NET would damage them, so every string parameter is a `byte[]` and every string result an `IntPtr`; `Z(s)` makes the NUL-terminated bytes of a string (one byte for each character; characters above 255 are not allowed) and `Str(p)` / `StrN(p, n)` read a result the same way and give the library's memory back with `SDFree`. `FM`, `VM` and `SVM` are the marks as characters. An `int *` parameter is an `out int`. Variadic C functions cannot be declared in C#, so `SDCall` and `SDCallx` are `SDCall0` to `SDCall20` and `SDCallx0` to `SDCallx20`, all the same entry point; **`SDCall` writes the subroutine's results back into the buffers you give it with no check of their size, so use `SDCallx` and read the arguments with `SDGetArg`.**

The library is loaded by the name `sdclilib`, so .NET looks for `sdclilib.dll` where it looks for any native library: the program's directory or `PATH` (the installer puts `%USERPROFILE%\SDCoreSolo\usr\bin` on your `PATH`), or where a resolver you set with `NativeLibrary.SetDllImportResolver` points. The calling convention is `cdecl`. A 64-bit program needs the 64-bit library.

### PHP (FFI)

`syscom\sdclient.php`, for PHP 7.4 and later with the FFI extension enabled:

```php
require 'sdclient.php';
$sd = SDClient::load(getenv('USERPROFILE') . '\\SDCoreSolo\\usr\\bin\\sdclilib.dll');
if ($sd->SDConnect('server', 4249, 'user', 'password', 'account')) {
    $fno = $sd->SDOpen('CUSTOMERS');
    $err = SDClient::intRef();
    $rec = SDClient::str($sd->SDRead($fno, '1001', FFI::addr($err)));
    $sd->SDDisconnect();
}
```

A PHP string is already a string of bytes, so the marks pass through unchanged. `SDClient::str($p)` copies a result into a PHP string and frees the library's copy; `strn($p, $n)` does the same for `$n` bytes that may hold zeros (`SDReadBlk`). Use `SDCallx` and `SDGetArg` rather than `SDCall`, which writes into the buffers it is given and must not be given a PHP string.

**Neither of these two files has been run.** They are held to the library's header by a check (`gplbld\test-clientheaders-units.py`: every function, every parameter and result type, the calling convention, the status constants, and the PHP declarations read by a C compiler beside the header itself), which is not the same as a program having called the library.

## What the API cannot do

| | |
|---|---|
| `sh` and `OS.EXECUTE` | **refused over the API while the `sh` and `os` switches are off, which is how they are delivered** (the API server reads the switches when it enters the account, and answers *is not permitted to use the operating system shell* to an `execute 'sh ...'`); once the administrator has turned a switch on (`set.os.access`) the client's next login can use it, on your ordinary Windows token like any other session. See *API access* and *Operating system access* in the GettingStarted set |
| Open files outside the account | refused (status 3035 — *not permitted*) |
| Reach the credential file | never, and cannot be added |
| Enumerate accounts | refused; all the failure cases give the same message |
| Write `global.bp.out` | refused, unless the session signed in with the global password |

## Continued in

[SD BASIC - Python Integration](37a-sd-basic-python-integration.html) — the
other direction: a BASIC program reaching out to Python, rather than a
program outside SD reaching in.
