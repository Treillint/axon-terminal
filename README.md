# ⬡ Axon Terminal

**An AI-native terminal for macOS and Linux.** Claude lives inside your shell — ask questions, get explanations, run commands, and build persistent AI memory, all without leaving the prompt.

[![Download](https://img.shields.io/badge/Download-macOS%20%7C%20Linux-00ff88?style=flat-square&logo=apple&logoColor=white)](https://treillint.github.io/axon-terminal)
[![Release](https://img.shields.io/github/v/release/Treillint/axon-terminal?style=flat-square&color=00ff88)](https://github.com/Treillint/axon-terminal/releases/latest)
[![License](https://img.shields.io/badge/license-MIT-blue?style=flat-square)](LICENSE)
[![Platform](https://img.shields.io/badge/platform-macOS%2013%2B%20%7C%20Linux%20ARM64-lightgrey?style=flat-square)](https://treillint.github.io/axon-terminal)

---

## What is Axon Terminal?

Axon Terminal replaces the traditional shell with an AI-integrated experience. It runs a real shell session via PTY — so every standard shell command works exactly as you'd expect. On top of that, it layers an AI mode powered by Claude (Anthropic), Ollama local models, or any OpenAI-compatible remote API (Groq, Together, OpenRouter).

The result: a terminal that understands what you're trying to do.

---

## Installation

### macOS

#### Homebrew (recommended)
```bash
brew tap treillint/tap
brew install --cask axon-terminal
```

#### Direct download
1. Go to [treillint.github.io/axon-terminal](https://treillint.github.io/axon-terminal)
2. Click **Download for macOS**
3. Open `AxonTerminal.dmg`
4. Drag **Axon Terminal** to **Applications**
5. Launch from Applications — macOS Gatekeeper will clear it (notarized + stapled)

**Requirements:** macOS 13 Ventura or later · Apple Silicon or Intel

### Linux (ARM64)
1. Go to [treillint.github.io/axon-terminal](https://treillint.github.io/axon-terminal)
2. Click **Download for Linux**
3. Make executable and run:

```bash
chmod +x AxonTerminal.AppImage
./AxonTerminal.AppImage
```

**Requirements:** Ubuntu 22.04+ or Debian 12+ · ARM64 (aarch64)

---

## Setup

### 1. Set your API key

Click **⬡ set API key** in the title bar, or type directly in the terminal:

```
axon key sk-ant-...
```

Your key is stored locally and never leaves your machine except to reach Anthropic's API.

### 2. Activate AI mode

```
axon ai
```

The prompt changes to `⬡ axon ~/` — you're now in AI mode.

---

## Tutorial

### Asking questions

Prefix any line with `?` to ask the AI without leaving the terminal:

```
⬡ axon ~/ ❯ ? what does the grep -P flag do
```
```
The -P flag tells grep to use Perl-compatible regular expressions (PCRE).
This enables lookaheads, lookbehinds, and other advanced regex features
not available in basic or extended regex modes.

Example:
  grep -P '(?<=foo)bar' file.txt   # matches 'bar' only when preceded by 'foo'
```

The `?` prefix works from any mode — you don't need to switch to AI mode first.

---

### Explaining shell output

Run a command, then ask what it means:

```
⬡ axon ~/ ❯ df -h
Filesystem      Size   Used  Avail Capacity  Mounted on
/dev/disk3s1s1  460G   8.5G   217G     4%   /

⬡ axon ~/ ❯ ? explain the output above
```
```
Your disk is 460GB total. Only 8.5GB is used (4%), leaving 217GB free.
The filesystem is an APFS volume on disk3 — typical on macOS with Apple Silicon.
You have plenty of space.
```

---

### Debugging errors

Paste an error and ask for help — the terminal context is automatically included:

```
⬡ axon ~/ ❯ pip install numpy
error: externally-managed-environment

⬡ axon ~/ ❯ ? how do I fix this
```
```
This error means your Python is managed by the system package manager
(Homebrew or the OS), which prevents pip from installing globally.

Fix: use a virtual environment:
  python3 -m venv .venv
  source .venv/bin/activate
  pip install numpy

Or, if you just want to install it quickly:
  pip install numpy --break-system-packages
```

---

### Writing and running scripts

Ask the AI to write a script, save it to memory, then execute it:

```
⬡ axon ~/ ❯ ? write a bash script that renames all .jpeg files to .jpg in the current folder
```
```bash
#!/bin/bash
for f in *.jpeg; do
  [ -f "$f" ] && mv "$f" "${f%.jpeg}.jpg"
done
echo "Done."
```

```
⬡ axon ~/ ❯ mem save rename-jpeg
⬡ axon ~/ ❯ mem run rename-jpeg
Done.
```

---

### Using AI mode for unknown commands

In AI mode, any command that isn't found is automatically sent to the AI:

```
⬡ axon ~/ ❯ axon ai

⬡ axon ~/ ❯ deploy my app to staging
```
```
To deploy to staging I'll need a bit more context. What's your stack?
If you're using a Makefile, try: make deploy-staging
If you're on Heroku: heroku git:remote -a your-app-staging && git push heroku main
...
```

---

### Persistent memory

Save useful AI responses and retrieve them in future sessions:

```
⬡ axon ~/ ❯ ? what are the most useful git commands for undoing mistakes
```
*(AI responds with a detailed list)*

```
⬡ axon ~/ ❯ mem save git-undo-cheatsheet
Saved → git-undo-cheatsheet

⬡ axon ~/ ❯ mem list
  1  git-undo-cheatsheet
  2  rename-jpeg

⬡ axon ~/ ❯ mem show git-undo-cheatsheet
```

Memory persists across sessions in `~/.axon/axonVM/` and is automatically injected as context into relevant AI queries.

---

### Switching models

```
# Switch to a local Ollama model (no API key needed)
⬡ axon ~/ ❯ axon model pull llama3.2
⬡ axon ~/ ❯ axon model use llama3.2

# Switch to a remote API (Groq example)
⬡ axon ~/ ❯ axon remote key gsk_...
⬡ axon ~/ ❯ axon remote url https://api.groq.com/openai/v1
⬡ axon ~/ ❯ axon remote model llama-3.3-70b-versatile
⬡ axon ~/ ❯ axon remote on

# Back to Claude
⬡ axon ~/ ❯ axon model claude
```

---

## Command Reference

### Mode
```
axon ai          # activate AI mode
axon legacy      # return to standard shell mode
```

### Models
```
axon model claude              # switch to Claude
axon model pull <name>         # download Ollama model
axon model use <name>          # switch to Ollama model
axon model list                # list installed models
axon model rm <name>           # remove a model
axon remote key <api-key>      # set remote API key
axon remote url <url>          # set remote endpoint
axon remote model <name>       # set remote model name
axon remote on                 # activate remote model
```

### Memory
```
mem save <label>               # save last AI response
mem import <label> <file>      # import a file into memory
mem list                       # list all saved entries
mem show <id>                  # display an entry
mem retrieve <id>              # raw content only
mem run <id>                   # execute saved code
mem rm <id>                    # delete an entry
mem clear                      # clear all entries
```

### History
```
axon history     # list past AI queries (numbered)
axhist           # full conversation log (user + assistant)
```

---

## Features

| Feature | Description |
|---|---|
| `? <question>` | Ask AI anything, from any mode |
| AI mode | Unknown commands auto-routed to AI |
| Streaming | Responses appear token by token |
| Multi-tab | `⌘T` new tab · `⌘W` close tab |
| Font size | `⌘+` / `⌘-` |
| Shell history | AI queries saved to history (up-arrow recall) |
| Axon VM | Persistent RAG memory in `~/.axon/axonVM/` |
| Local models | Ollama — fully offline, no API key |
| Remote APIs | Groq, Together, OpenRouter, any OpenAI-compatible endpoint |

---

## Privacy

Axon Terminal sends your queries to the AI backend you configure:
- **Claude**: Anthropic's API — governed by [Anthropic's privacy policy](https://www.anthropic.com/privacy)
- **Ollama**: fully local, nothing leaves your machine
- **Remote APIs**: governed by the respective provider's policy

No telemetry, no analytics, no data collection by Treillint.

---

## License

MIT © 2026 [Treillint LLC](https://treillint.com)

---

<p align="center">
  Built by <a href="https://treillint.com">Treillint LLC</a> · 
  <a href="https://treillint.github.io/axon-terminal">Download</a> · 
  <a href="https://github.com/Treillint/axon-terminal/releases">Releases</a>
</p>
