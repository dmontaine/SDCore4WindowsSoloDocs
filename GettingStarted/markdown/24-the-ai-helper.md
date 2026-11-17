Title: The AI Helper
Subtitle: One provider and one model, a key that SD never shows or keeps, and one gate - and what a helper that runs as you can and cannot promise.

SD can ask an AI model a question for you (the `ai` verb, see *SD TCL - Asking the AI* in the User set). This page is for the administrator (`admin` first): it says what is set up, what leaves the computer, how to choose the provider and give it a model and a key, how to turn the gate on and off, and **what is weaker here than in SD Core**.

> **Nothing is on as delivered.** There is no provider, no key, and the gate is off (each named provider comes with a default model, so the model is the one thing you need not supply). The installer asks nothing about the AI. Until you have done the other three, the `ai` verb says so in words and nothing is sent anywhere. Turning one of them on is your act, and each is one command (or one key on the AI choice page) that can be undone.

## What leaves the computer

When you type `ai some text`, SD shows you the provider, the model and the size of all that will be sent, then your text, and asks whether to send it. Only if you answer `y` or `yes` does anything go out. What goes out is your standing text (below), that text, one fixed sentence that asks for a short plain answer, and a limit on the length of the answer. It is sent over **https**, with the certificate checked, to the address of the provider you chose, and to nowhere else. The key goes only to that address. No proxy is used: a computer that reaches the internet only through an HTTP proxy cannot use the AI.

SD adds nothing of the account, the session or the data. You can of course type or paste whatever you like into the question, which is why the preview is not optional.

**The standing text.** Every request, from the `ai` verb and from every AI line, begins with a few lines of yours: what SD is, how its commands are written, which gates there are, and where SD's help is. SD ships a default; you can replace it (see [The standing text](#the-standing-text) below). It is not in the preview and it does not change the question. The preview's size counts it.

**No passages of the documentation.** On SD Core Solo for Linux the AI line also adds sections of SD's own documentation, from a copy its installer puts beside SD. **SD Core Solo for Windows does not install that copy**, so the AI line here sends no passages, and the preview shows none.

## What the AI is never given, and what is not taken to it

**The AI in SD helps you use SD and write your own programs, dictionaries and files. It is never given SD's own code or any other system information, whichever provider you chose, and not even after `admin` can a question about changing SD be taken to it from inside SD.** The source is open, so if you want help changing SD you can ask an AI outside SD; inside SD that help is not offered. Two things enforce this, and they are not equally strong.

1. **Where the text comes from (enforced).** Every program that puts the text of a record or file into a request (the IDE, SED, the collection editor and the debugger) tells the AI routine where the text comes from. Before anything is shown or sent, the routine refuses text that is part of SD itself, and text whose origin it cannot tell. SD's own files are everything in the `sdsys` folder of your Solo folder: the programs, headers, messages, help text, dictionaries and files of the system account, the passwords, the audit trail, the error log and the AI helper's folder `sdsys\ai`; and any directory called `gplsrc`, `gplbld` or `gplobj`. Your account's own folder (`user_accounts\sduser`) is not among them. It is decided by where a file is, with `..` and links followed, never by what the text says. The headers in `syscom` are in `sdsys`, so they are not sent either; your own programs are, `$include` lines and all. If you want the AI to see one of those headers for a program of yours, copy it into the account: a copy there is the account's own file, and nothing is opened up.
2. **What is typed (a screen).** A request that names the system's code or build is refused in words before it is shown, and nothing is sent: `gpl.bp`, `gplsrc`, `gplbld`, `$internal`, a kernel file such as `op_sort.c`, a kernel key such as `k$audit`, the compiler's own programs, the makefile, SD's source or kernel code. **This is a screen, not a wall.** A question that names none of those is not caught, and a question that does reach an AI may be answered from whatever that AI learned of the open source. SD's gate is here, in SD.

Nothing the AI returns is run, and the output of a command is never given back to it. A refusal writes one line to the audit trail (`ai refused why=request ...` or `ai refused why=origin ...`) and never the text. What you type or paste yourself is yours, and is in the preview.

## What is weaker than in SD Core, said plainly

SD Core for Windows runs the helper as a Windows service under an account of its own (`NT SERVICE\sd-ai`), with its key encrypted for that account in a folder only it and SYSTEM can open, and asks Windows who is calling. **Solo makes no Windows account and no service of its own** (that is what makes it a Solo), so none of that is possible. The helper runs **as your own Windows user**: SD starts it for each request, as a child of your session, and it ends when the request is answered. Its settings and keys are in your Solo folder (`sdsys\ai`), and each key is encrypted with Windows DPAPI for your Windows user.

What follows from that, and what does not:

- **SD itself keeps its side.** `set.ai` and `tool.ai` are the administrator's. Nothing in SD ever shows a key, and none is ever written to the audit trail; `set.ai` reads back only whether it is *set*. A key typed at the helper's own prompt (`set.ai key PROVIDER`) does not pass through SD at all; one typed in the page's setup dialog does pass through your SD session and a hand-over file on its way (see [The key](#the-key), below).
- **Another program of yours is not stopped.** Any program that runs as your Windows user - a PowerShell window, a script, a program of the account that has been given `os.execute` - can run the helper's program with the words of `set.ai`, change its settings, turn the gate on, or use the key: DPAPI opens the key for whoever runs as you. The gate is the one switch SD keeps; it is not a wall against a program that is not SD. **A key is as safe as your other private files, and no safer.** If that matters, do not turn on the account's `sh` or `os` switches (see [Operating system access](06b-operating-system-access.html)) and do not run untrusted programs as this Windows user.
- **Another Windows user is stopped.** Each key file's access list allows only you and SYSTEM, and its DPAPI encryption is for your Windows user, so another ordinary user of the computer cannot read it, and a copy of the file, a backup or a copied disk does not carry a usable key. **A Windows administrator of this computer is not stopped**: treat everyone who is an administrator here as someone who can obtain the key.
- **A session that did not sign in with your Windows password may not be able to open the key.** DPAPI opens a key with what Windows unlocked when you signed in with your password. A session that reached SD in another way - over ssh with a key rather than your Windows password, for example - may find a key that is set but cannot be opened: SD then says the provider's key is not set. This has not been tried; sign in over ssh with your password if it happens.

If you need the key out of reach of your own programs, use SD Core for Windows, where the helper is a different account.

## The pieces

All in your Solo folder (`%USERPROFILE%\SDCoreSolo` as installed):

| Piece | What it is |
|---|---|
| `usr\bin\sd-ai.exe` | The helper, a native program. SD starts it for each request, as you, with no window; it is the only part of SD that connects outward and the only part that reads a key. Run with the words of `set.ai`, it is also the helper's client. |
| `usr\bin\ai-providers` | The connection details that come with SD for Anthropic, OpenAI, Google and DeepSeek, and each one's default model. |
| `usr\bin\ai-context` | The standing text that comes with SD. An install replaces it. |
| `sdsys\ai\` | The helper's own folder: the settings (`ai.conf`, which holds the gate line), your standing text (`context`), the counters of the limits (`state\`) and the helper's log (`sd-ai.log`: your security identifier, the provider, the result and the sizes of each request, never the text or the key). The helper makes it on the first request. |
| `sdsys\ai\keys\` | One key file per provider, encrypted with DPAPI for your Windows user, open only to you and SYSTEM. A key file whose access list names anyone else is refused, not repaired. |
| `sdsys\$aikey\` | The setup dialog's hand-over folder, open only to you and SYSTEM. A key typed in the dialog is in a file here only until `set.ai key PROVIDER handover` has given it to the helper; the file is then removed. |

There is no service to set up and nothing to start: the helper does nothing until the AI is set up and a request is made.

**An upgrade keeps `sdsys\ai`.** So does the uninstaller's **Keep**: the settings, your standing text and the keys stay with your data, and an install that reloads the kept data finds them again (see [Upgrading and uninstalling](01a-upgrading-and-uninstalling.html)). **Delete** removes them with the rest of the folder.

## The four things that turn it on

1. **A provider.** One: Anthropic, OpenAI, Google, DeepSeek or Custom.
2. **A model** for that provider. Each of the four named providers comes with a default model, shown on the page and used until you choose another (in the setup dialog, or `set.ai model PROVIDER NAME`; a dash goes back to the default). Model names change, so check the one you will pay for; the setup dialog offers the provider's own list once a key is set, and `set.ai models PROVIDER` prints it at the prompt. Model names are case-sensitive. Custom has no default.
3. **A key** for that provider, from the provider's own account. It is typed or pasted in the setup dialog (shown as `*`) or at the helper's own prompt, and kept for the helper only.
4. **The gate**: `set.ai gate on`. It is a setting of the helper (the line `gate=yes` in `sdsys\ai\ai.conf`); anything else, or no line, is off.

## The AI choice page: `tool.ai`

```
tool.ai
```

An administrator-only full-screen page (140 columns by 43 lines at least), and a tile on the home screen (*Data, network and AI*, **AI**). It has two boxes.

**AI provider** lists Anthropic, OpenAI, Google and DeepSeek, then Custom, one to a row. A check box marks the provider in use; none marked means the AI is off. The four named providers come with their address and a default model filled in (the Model column says `(default)` beside one you have not changed); you supply the key, and S (setup) sets the key and the model together. **Custom** is the fifth choice, where you fill in everything yourself: a name, an `https://` address, the request layout (`anthropic` or `openai`), the authentication (`bearer` or `x-api-key`), a version header (or none) and the name of the field that limits the length of an answer. Below the list, the row under the cursor is shown in full, with the file of certificate authorities and the limits.

**May this account use the AI?** is the one row for your account, with the state of the gate.

| Key | In the Provider box | In the Account box |
|---|---|---|
| Enter or space | check the provider under the cursor (you type its name to confirm), or uncheck the one in use | turn the gate on (you type `ai`) or off |
| S | the setup dialog for the provider under the cursor: the key and the model, the model chosen from the provider's own list (below) | |
| X | remove the key | |
| C | fill in Custom | |
| D | remove the Custom connection (the helper refuses while it is in use) | |
| A | the file of certificate authorities (you type `ca` to confirm) | |
| L | change a limit | |
| T | the standing text, in a box with an editor | the same |
| F6, Shift-F6, Left, Right | change box (F6 goes on to the AI line when the AI is on) | change box |
| F1, `?` or a click on `[?]` | explain the box | explain the box |
| F5 | read the machine again | read the machine again |
| Ctrl-Q | leave | leave |

**The page keeps nothing itself and never shows a key.** Every change is a `set.ai` sentence, shown with its effect in a box that you answer Run or Cancel (Enter chooses Cancel). The page then runs exactly that sentence. The exception is the setup dialog, whose Save is the confirmation.

**The setup dialog (S).** It looks the same every time, whatever is already set:

- **Key**, a password field. What you type or paste is shown as `*`. When a key is already set the field shows a row of twelve stars, which keeps it; replace them to give a new key.
- **Model**, a list of the provider's own model names, with the one in use chosen. With no key yet the list says the key comes first: enter the key and press Tab, and the list is filled from the provider with that key. A model in use that the list does not hold is pointed out, and when it differs from a listed one only in capitals the listed one is chosen for you.
- **Other name**, only for a model the list does not hold, such as on a Custom server that gives no list. A name typed here is still checked against the list when there is one.
- **Save** sets what you chose; **Cancel** or Esc changes nothing more (a key already handed over by Tab stays set, and the page says so).

**What the dialog does with the key.** When you leave the Key field with a new key (Tab, or Enter, which then shows the list rather than saving), the page writes it to the file `key` in `sdsys\$aikey` (a folder whose access list it sets to you and SYSTEM only), runs `set.ai key PROVIDER handover`, which gives the path of that file to the helper's client, and the file is removed whether or not that worked: the client removes it as soon as it has read it, and `set.ai` removes it again. The key is never on a command line and never in the audit trail. It does pass through your SD session's memory while the dialog is open, and through that file for a moment (Windows does not overwrite a deleted file's space). If you would rather it never touched SD, use `set.ai key PROVIDER` at the prompt, which hands the console to the helper's own prompt.

Opening the page changes nothing and writes one audit line, `tool start app=tool.ai`.

## The verb: `set.ai`

```
set.ai
set.ai gate on | off
set.ai provider anthropic | openai | google | deepseek | custom | none
set.ai model PROVIDER NAME | -
set.ai models PROVIDER
set.ai key PROVIDER
set.ai key PROVIDER handover
set.ai key PROVIDER clear
set.ai custom NAME URL FORMAT AUTH VERSION MAXPARAM
set.ai custom clear
set.ai ca PATH | -
set.ai limit NAME NUMBER
set.ai context HEX | -
```

For the administrator only. Alone it says what is set: the provider in use, the gate, each provider's address, model and whether its key is set, the certificate file and the limits. It never shows a key. `provider none` turns the AI off. `gate off` stops every request while leaving the provider, the model and the key as they are. `models` asks the provider, with its key, for the names of its models and prints them, one to a line. `model` keeps the name you give, and says so when the provider does not list it (`deepseek does not list a model named Deepseek-V4-Pro, so requests may be refused`, then `It lists deepseek-v4-pro`); a dash goes back to the provider's default. `key` hands the console to the helper's prompt; nothing typed there reaches SD. `key PROVIDER handover` is what the page runs: it hands the page's file to the helper's client and removes it. When a provider refuses a request, its own explanation is shown after the HTTP status (`the provider refused the request (HTTP 400): Model Not Exist`), cut to one line, with any copy of the key taken out; a refused key (401 or 403) shows only that the key was not accepted. A word may hold letters, digits and `. _ : / @ + , -` and nothing else, so **a path is written with forward slashes**: `set.ai ca C:/Users/you/ca.pem`. The helper checks every result whole before it writes it: one provider selected means its details are complete, a Custom address is `https://` with no query string, a limit is a whole number in range.

`set.ai` runs the helper's client, `usr\bin\sd-ai.exe` in your Solo folder, as you; it needs no password, because the helper's only possible caller is your own Windows user.

The limits, and what they are as delivered:

| Limit | Delivered | Meaning |
|---|---|---|
| `requests-hour` | 20 | requests you may make in an hour |
| `tokens-day` | 100000 | tokens (in plus out) you may use in a day |
| `timeout` | 30 | seconds to wait for the provider (at most 300) |
| `bytes-request` | 65536 | the most that may be sent in one request |
| `bytes-reply` | 65536 | the most that may come back |
| `max-tokens` | 1024 | the longest answer asked for, at the prompt (`ai`) and on every page: raise it for longer answers, lower it for shorter ones (a provider may refuse a figure above its model's own limit) |

## The standing text

Press **T** on `tool.ai` (or choose Provider, Standing text). A box shows the text that is sent first with every AI request, and says whether it is SD's default or yours. Edit it there: Enter starts a new line and Tab moves to the buttons. **Save** shows the command and its effect in a Run / Cancel box (Enter chooses Cancel), and Run keeps your text. **Shipped default** removes yours, so SD's default is sent again. **Cancel** or Esc changes nothing.

The default tells the AI that this is SD Core Solo for Windows, that it should answer for SD and not for another multivalue product, that command words are lower case, which dictionary types there are, that there is one user and one account and that `admin` makes the same person the administrator, that access to the operating system and the shell is a gate that is off as delivered, and where SD's help is: F1 and the `[?]` buttons, the `pdf` command, and the documentation on the web. Put your own facts in it, such as which files matter and how your programs are named.

What is kept:

- plain English: printable ASCII characters and new lines, at most 4,000 characters;
- nothing that names SD's own code or build (`gpl.bp`, `gplsrc` and the other names of the screen above). Such a text is refused when you save it, with the line `set.ai context refused name=...` in the audit trail. A text that names them all the same (written into the helper's file by hand) stops every request, with `ai refused why=context`, until it is put right.

The text is sent to the provider with every request, before SD's own instruction, and counts towards `bytes-request`. It is not in the preview (the size shown there counts it) and the question is sent as typed. It is kept by the helper in `sdsys\ai\context`; the shipped default is `usr\bin\ai-context`. An upgrade keeps yours.

At the prompt the verb takes the text as one word of hexadecimal (`set.ai context 48656c6c6f`), which is how the page passes a text of several lines; `set.ai context -` goes back to the default.

## The key

The key is typed at the helper's own prompt (`set.ai key anthropic`) or in the setup dialog on the page. At the prompt it goes from your console straight to the helper's client and SD never sees it; the prompt shows one `*` for each character you type or paste (Backspace takes one back, Ctrl-U clears the line), so a long key shows that it arrived, and the key itself is never shown (the number of stars is its length, which anyone looking at the screen can count). From the dialog it passes through your SD session's memory and a hand-over file only you and SYSTEM can reach, which is removed at once (see above). Either way the helper encrypts it with DPAPI for your Windows user and writes it to `sdsys\ai\keys\anthropic`, open only to you and SYSTEM; SD does not keep it, print it or put it in the audit trail, and the page and the verb read back only whether it is **set**. `set.ai key anthropic clear` removes it. A key is never an argument of a command, so it is in no process list and no history.

If a key is lost or shown to someone, clear it and ask the provider for a new one: SD cannot tell you who has seen it.

## Custom: say it plainly

Custom lets you point the helper at any `https://` address. The text you send, and the key, then go to that host. That is your deliberate act and it is the widest thing on this page. What stays fixed: https only (a local model server on `http://` is not allowed), TLS 1.2 or later, the certificate and the host name are always checked (there is no switch to turn that off), the helper never follows a redirect to another host, and the key is sent only to the address you gave. With no certificate file set, Windows itself checks the provider's certificate, as it would for a browser on this computer. A private certificate authority is the file you name with `set.ai ca` (a drive path written with forward slashes); **it replaces the Windows store for the helper**, so whoever holds one of its keys can read what you send and the key. Use a file you trust, and `set.ai ca -` to go back to the Windows store.

If you want only the four named providers, leave Custom unfilled.

## The audit trail

All in lower case, none with a key or the text:

```
ai request account=sduser program=ai bytes.out=154
set.ai gate=on
set.ai provider=anthropic
set.ai model provider=anthropic model=claude-...
set.ai key set provider=anthropic
set.ai key set provider=anthropic from=page
set.ai key cleared provider=anthropic
set.ai custom name=MyLLM host=api.example.com
set.ai ca file=C:/Users/you/ca.pem
set.ai limit name=tokens-day value=50000
set.ai context bytes=1640
set.ai context=default
set.ai context refused name=gpl.bp
ai refused why=context app=home name=gpl.bp
tool start app=tool.ai
```

The `ai request` line is written by the SD kernel **before** the helper is started, with the account and the program that called, so a crash or a refusal still leaves the record. The Custom line names the host and never the whole address. The helper itself writes nothing to the audit trail; its own one line per request is in `sdsys\ai\sd-ai.log`.

## Troubleshooting

| What you see | What it means |
|---|---|
| `The AI helper could not be reached: it is not running, or it is not installed` | SD could not start `usr\bin\sd-ai.exe` in your Solo folder: check that it is there; installing again puts it back |
| `The AI is off: the administrator has not chosen a provider (tool.ai or set.ai)` | nothing is checked on the page (`set.ai provider ...`) |
| `The AI is not turned on: the administrator can turn it on with set.ai gate on` | the gate is off |
| `The AI is not set up for anthropic: the administrator has not set its key (tool.ai or set.ai)` (or, for Custom, `model`) | the provider is chosen but its key (or Custom's model) is missing; or a key is set and this session cannot open it (see [What is weaker](#what-is-weaker-than-in-sd-core-said-plainly)) |
| `The AI did not answer: refused (the provider refused the request (HTTP 400): ...)` | the provider's own words follow the status: often a model it does not have; choose one from its list with S |
| `The AI did not answer: limit (...)` | one of the limits above is used up; wait, or raise it |
| `The AI did not answer: net (...)` or `timeout` | the provider could not be reached; check the address, the certificate, the clock, and whether the machine needs a proxy to reach the internet (the helper uses none) |

## What has not been shown

**Not run on an installed SD Core Solo for Windows yet.** The helper is SD Core for Windows' program with SD Core Solo for Linux's same-user rules; its own tests pass against a stand-in provider, and the kernel's side has started it and read its answers outside an installed SD. Not yet run: a request from an installed Solo, the page and the verb there, the DPAPI key files under a real sign-in, a session that signed in over ssh with a key, and any real provider.

## See also

[Administrator commands](06-administrator-commands.html) ·
[Operating system access](06b-operating-system-access.html) ·
[Upgrading and uninstalling](01a-upgrading-and-uninstalling.html) ·
[Giving an AI agent an installation of its own](23-giving-an-ai-agent-its-own-installation.html).
