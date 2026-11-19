# ⬡ Axon Terminal

**An AI-native terminal for macOS.** Claude lives inside your shell — ask questions, get explanations, run commands, and build persistent AI memory, all without leaving the prompt.

[![Download](https://img.shields.io/badge/Download-macOS-00ff88?style=flat-square&logo=apple&logoColor=white)](https://treillint.github.io/axon-terminal)
[![Release](https://img.shields.io/github/v/release/Treillint/axon-terminal?style=flat-square&color=00ff88)](https://github.com/Treillint/axon-terminal/releases/latest)
[![License](https://img.shields.io/badge/license-MIT-blue?style=flat-square)](LICENSE)
[![Platform](https://img.shields.io/badge/platform-macOS%2013%2B-lightgrey?style=flat-square)](https://treillint.github.io/axon-terminal)

---

## What is Axon Terminal?

Axon Terminal replaces the traditional shell with an AI-integrated experience. It runs a real `zsh` session via PTY — so every standard shell command works exactly as you'd expect. On top of that, it layers an AI mode powered by Claude (Anthropic), Ollama local models, or any OpenAI-compatible remote API (Groq, Together, OpenRouter).

The result: a terminal that understands what you're trying to do.

```
⬡ axon ~/ ❯ ? what does this docker-compose file do
⬡ axon ~/ ❯ deploy my app to staging
⬡ axon ~/ ❯ mem save deployment-notes
```

---

## Features

### AI Mode
- **`? <question>`** — ask AI anything, from any mode, at any time
- **`command_not_found` routing** — unknown commands are automatically sent to AI in AI mode
- **Streaming responses** — answers appear token by token, no waiting
- **Up-arrow recall** — AI queries are saved to zsh history for easy re-use

### Models
| Backend | How to activate | Notes |
|---|---|---|
| Claude (Anthropic) | `axon model claude` | Default; requires API key |
| Local (Ollama) | `axon model use <name>` | No API key needed; runs fully offline |
| Remote API | `axon remote on` | Groq, Together, OpenRouter, any OpenAI-compatible endpoint |

### Axon VM — Persistent AI Memory
All memory lives in `~/.axon/axonVM/`. Entries are scored by keyword relevance and injected as context into every AI query.

| Command | Description |
|---|---|
| `mem save <label>` | Save the last AI response |
| `mem import <label> <file>` | Import a file's contents into memory |
| `mem list` | List all saved entries |
| `mem show <id>` | Display an entry with headers |
| `mem retrieve <id>` | Raw content only (cat-like) |
| `mem run <id>` | Execute saved code |
| `mem rm <id>` | Delete an entry |
| `mem clear` | Clear all entries |

### Shell
- Full `zsh` PTY — every shell feature works (aliases, functions, pipes, redirects)
- Multi-tab support: `⌘T` new tab, `⌘W` close tab
- Font size: `⌘+` / `⌘-`
- Tab titles update to current directory via OSC-2

### UI
- Dark terminal theme with green accent (`#00ff88`)
- Narrow left sidebar: AI/Legacy mode toggle, auto-save, Axon VM manager, model picker
- Title bar shows current mode + active model
- `⬡ connected` badge — tap to set API keys

---

## Installation

### Download (recommended)
1. Go to [treillint.github.io/axon-terminal](https://treillint.github.io/axon-terminal)
2. Click **Download for macOS**
3. Open `AxonTerminal.dmg`
4. Drag **Axon Terminal** to **Applications**
5. Launch from Applications — macOS Gatekeeper will clear it (notarized + stapled)

### Requirements
- macOS 13 Ventura or later
- Apple Silicon or Intel (universal binary)
- An [Anthropic API key](https://console.anthropic.com) for Claude AI mode
- [Ollama](https://ollama.com) installed if you want local models

---

## Setup

### Claude API Key
Click **⬡ set API key** in the title bar, or run:
```zsh
axon key sk-ant-...
```

### Ollama (local models)
```zsh
# Pull a model
axon model pull llama3.2

# Switch to it
axon model use llama3.2

# Switch back to Claude
axon model claude
```

### Remote API (Groq, Together, OpenRouter)
```zsh
axon remote key gsk_...                          # your API key
axon remote url https://api.groq.com/openai/v1   # endpoint
axon remote model llama-3.3-70b-versatile        # model name
axon remote on                                    # activate
```

---

## Command Reference

### Mode
```zsh
axon ai          # activate AI mode
axon legacy      # return to standard shell mode
```

### Models
```zsh
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

### History
```zsh
axon history     # list past AI queries (numbered)
axhist           # full conversation log (user + assistant)
```

---

## Architecture

```
axon_terminal/
├── lib/
│   ├── ai/
│   │   ├── ai_service.dart      # Claude API streaming
│   │   ├── history_store.dart   # Persistent JSONL history (~/.axon/axonVM/history.jsonl)
│   │   └── providers.dart       # Riverpod providers (mode, settings, key)
│   ├── memory/
│   │   └── memory_store.dart    # JSON memory entries, keyword scoring, top-3 injection
│   ├── server/
│   │   └── axon_server.dart     # HttpServer on port 9099; zsh init script builder
│   ├── shell/
│   │   └── shell_session.dart   # PTY via flutter_pty, hardcoded /bin/zsh
│   ├── pages/
│   │   └── home_page.dart       # UI: title bar, tabs, sidebar, all dialogs
│   └── app/
│       ├── app.dart
│       └── theme.dart           # AxonTheme constants
└── macos/
    └── Runner/
        ├── Release.entitlements # No sandbox; network client+server; file access
        └── MainFlutterWindow.swift  # 80% screen, centered window
```

### How the shell hook works
Axon injects a zsh init script 900ms after PTY start. The script:
- Sets `command_not_found_handler` to POST unknown commands to `localhost:9099/query`
- Registers `?` as a prefix alias that POSTs to `/query` from any mode
- Sets `precmd_functions` to enforce the `⬡ axon ~/` prompt in AI mode and emit OSC-2 for tab title updates
- Saves AI queries to zsh history via `print -s`

The HTTP server (`axon_server.dart`) bridges the shell hook to the AI service and streams responses back to the terminal.

---

## Building from Source

```zsh
# Prerequisites: Flutter 3.x, Xcode 15+, CocoaPods

git clone https://github.com/Treillint/axon-terminal.git
cd axon-terminal
flutter pub get
cd macos && pod install && cd ..
flutter run -d macos
```

To build a distributable DMG:
```zsh
./build_dmg.sh
```
The script builds, signs (Developer ID), notarizes with Apple, and staples — producing a ready-to-distribute `AxonTerminal.dmg`.

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
