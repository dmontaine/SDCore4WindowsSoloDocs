Title: Setting Up the AI Helper
Subtitle: The AI choice page, the set.ai verb, the standing text, the key, a custom provider, the audit trail and troubleshooting.

This page continues [The AI Helper](24-the-ai-helper.html).

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
| `The AI is not set up for anthropic: the administrator has not set its key (tool.ai or set.ai)` (or, for Custom, `model`) | the provider is chosen but its key (or Custom's model) is missing; or a key is set and this session cannot open it (see [What is weaker](24-the-ai-helper.html#what-is-weaker-than-in-sd-core-said-plainly)) |
| `The AI did not answer: refused (the provider refused the request (HTTP 400): ...)` | the provider's own words follow the status: often a model it does not have; choose one from its list with S |
| `The AI did not answer: limit (...)` | one of the limits above is used up; wait, or raise it |
| `The AI did not answer: net (...)` or `timeout` | the provider could not be reached; check the address, the certificate, the clock, and whether the machine needs a proxy to reach the internet (the helper uses none) |

## See also

[Administrator commands](06-administrator-commands.html) ·
[Operating system access](06b-operating-system-access.html) ·
[Upgrading and uninstalling](01a-upgrading-and-uninstalling.html) ·
[Giving an AI agent an installation of its own](23-giving-an-ai-agent-its-own-installation.html).
