Title: Encryption and the SDEXT interface
Subtitle: What libsodium provides, what an application can reach, and why field encryption has no key route.

SD Core Solo for Windows links libsodium and ships it as `cygsodium-26.dll` in
`usr\bin`, beside the server. Two things are built on it: the credential
exchange that authenticates an API login, and a pair of BASIC functions that
encrypt and decrypt a string. A third thing, the kept copy of your account
password, uses Windows' own protection (DPAPI) rather than libsodium, and is
reached through the same interface.

SD folds case, so a command may be typed in either case. Commands are shown
here in lower case.

## The short version

This page usually answers one of two questions.

**For credentials**, nothing needs configuring. Passwords are never stored. SD
keeps a SCRAM-SHA-256 verifier for each of the three passwords in the
credential store, the API login proves knowledge of the password without
sending it, and the primitives behind that exchange are the ones listed
further down. The one place a password is kept in a recoverable form is the
copy Windows protects for you — see
[The account and its passwords](05-account-types.html).

**For encrypting application data**, SD does not provide a usable route. The
functions exist and work, but the only way to produce a key they accept is an
internal-only call. This is set out in full under *Why field encryption is not
available* below, because a site planning to encrypt fields needs to know
before it writes the application, not after.

## What was removed

| | |
|---|---|
| The `encrypt.field` verb | Removed. It pointed at a program, `$CRYPTO`, which never existed in the GPL release, so the verb could not have worked in any build derived from it |
| `encrypt()` and `decrypt()` | Removed upstream in July 2024 and replaced by `sdencrypt()` and `sddecrypt()`. The old names do not compile |
| `SD_EUID_SET` and `SD_EUID_RESTORE` | Removed. They were the Linux setuid model, which has no meaning on Windows. Keys 102 and 103 are left unused, so an old program that calls one gets the answer any unknown key gets |

**The Python half of SDEXT is not on this list.** It was removed along with
the embedded interpreter, then rebuilt as a separate helper process,
`sdpy.exe`, rather than restored as a library inside `sd.exe` — the two cannot
safely share a process (`long` is a different width on each side of the MSYS2
boundary). `SDPYFUNC.H` and its twenty-one `PY_*` functions are documented in
full in the User set's *SD BASIC - Python Integration* chapter. A session may
start the helper on the same terms as `SH` and `OS.EXECUTE`, which on Solo you
always may — see [Operating system access](06b-operating-system-access.html).

## sdencrypt() and sddecrypt()

These are ordinary BASIC functions. Any program may compile a call to them.

```
sdencrypt(data, key, encoding)
sddecrypt(data, key, encoding)
```

Three arguments, not two. The third selects how the key is encoded and how the
result is returned:

| Encoding | Meaning | Key length required |
|---|---|---|
| `201` | hexadecimal | 64 characters |
| `202` | base64 | 44 characters |

The cipher is libsodium's authenticated `secretbox`, so the key is exactly
256 bits. The two lengths above are that key after encoding, and they are
checked exactly: a key of any other length is refused.

### Why field encryption is not available

A passphrase is not a key, and this is where a reader will otherwise lose a
day. On the multiuser W1.0-0, where this was measured:

```
sdencrypt('The quick brown fox', 'secretkey', 202)
```

returned nothing and set `status()` to **10204**, a key length error. `secretkey`
is nine characters and the function wanted 44. The code is the same here.

The function that turns a password into a key of the right length is
`sdext()`'s `SD_KEYFROMPW`, and `sdext()` is internal-only — it needs a program
compiled with `$internal`, which is reserved to SD's own setup steps. **So an
ordinary program cannot obtain a key these functions will accept, and there is
no supported way in.** On Solo the door that setup steps use, `sd -internal`,
admits a session only on a one-shot marker file the installer writes — see
[Security](12-security.html#the-installers-own-door).

An application that must encrypt data should do it outside SD, in the client,
and store the result as an ordinary string.

## The SDEXT interface

`sdext()` is the internal entry point to the cryptographic primitives.

```
rtnval = sdext(arg, isargmv, key)
```

`key` selects the operation, `arg` carries the arguments and `isargmv` says
whether `arg` holds several values rather than one.

**It cannot be called from an application.** `sdext` is in the compiler's
internal function table, so an ordinary program does not merely get refused —
it gets a misleading error. An unknown function is read as a matrix reference,
so the complaint is about a `dim` statement the program does not contain,
reported at the last line rather than at the call. That behaviour is covered by
*SD Basic - Restricted Commands* in the User set.

`$internal` needs both halves: the compiler tests for internal mode **and** for
the administrator flag. Internal mode alone was enough until 13 August 2026 and
was not safe, because internal programs are the only ones that may set the
administrator flag.

### The keys

Twelve are implemented. Every binary value is base64, because the interface
carries NUL-terminated strings and a raw 32-byte digest would contain a mark
character about one time in nine.

| Key | Value | Arguments | Result |
|---|---|---|---|
| `SDEXT_TestIt` | 1 | any | Prints each argument and returns a count. A diagnostic |
| `SD_SALT` | 100 | none | A fresh salt, base64 |
| `SD_KEYFROMPW` | 101 | password, salt | A 256-bit key derived from the password |
| `SD_SHA256` | 104 | one | SHA-256 of the argument |
| `SD_HMACSHA256` | 105 | base64 key, text message | HMAC-SHA256 |
| `SD_PBKDF2` | 106 | password, salt, iterations, length | Derived key |
| `SD_RANDBYTES` | 107 | count | Random bytes |
| `SD_XORBYTES` | 108 | two equal-length values | Their exclusive-or |
| `SD_CTEQUAL` | 109 | two values | `1` or `0`, compared in constant time |
| `SD_TLS_CBIND` | 110 | none | The API session's channel-binding value, for the SCRAM login. Empty when the session is not TLS |
| `SD_DPAPI_PROTECT` | 111 | one | Encrypts the argument with Windows' DPAPI, for this Windows user; the result is base64. **`$internal` callers only** |
| `SD_DPAPI_UNPROTECT` | 112 | one | The reverse. **`$internal` callers only** |

`SD_CTEQUAL` reports a malformed argument as an error rather than as `0`,
because by the time the login path compares these values both sides are the
server's own — a decode failure there is a defect, not a wrong password. A
caller deciding whether to admit a login must still treat the error as a
refusal.

**The two DPAPI keys are the kept copy of your account password.** DPAPI
encrypts for a Windows user, so what the first produces only that user can turn
back. It is Solo's, not the multiuser SD's, and the plaintext each one held is
wiped after use.

### What uses it

Eight shipped programs call `sdext()` for these keys, and they are the whole of
its use apart from the Python programs: `APISRVR`, `CRED_SET`, `CRED_VERIFY`,
`LOGIN`, `SD_GET_SALT`, `SD_KEY_FROM_PW`, `SDCLIENT` and `SOLO_STORE_PW`.
Between them they set a credential, verify one, run the API's SCRAM exchange,
and keep and read back the account password for `sd <command>`. The six
`PY_*` programs that start, stop and run the Python helper call it too, with
their own keys, described in the Python chapter.

**None of the other seven has a VOC entry**, so they are not an indirect route
to the interface. `LOGIN` is the program SD runs at every sign-in.

## Two constants that are not SDEXT keys

`SD_ENCODEHX` (201) and `SD_ENCODE64` (202) are defined alongside the SDEXT
keys and are easily mistaken for them. They are not keys and `sdext()` does not
implement them — passing either as a key returns a key error. They are the
third argument to `sdencrypt()` and `sddecrypt()`, and they are documented
above.
