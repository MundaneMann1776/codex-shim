# shim

One-click model switcher for Codex Desktop on macOS.

A menubar app that flips Codex between a ChatGPT subscription and any BYOK
provider (DeepSeek, OpenRouter, Google, Anthropic, OpenAI, anything
OpenAI-compatible) by rewriting a managed block in `~/.codex/config.toml`
and running a local Responses-API proxy on `127.0.0.1:8765`. Subscription
mode strips the managed block entirely; the shim gets out of the path.

## Requirements

macOS 13+, Python 3.11+, Codex Desktop already installed.

## Install

Homebrew (CLI + Python package):

    brew install MundaneMann1776/shim/shim
    codex-shim-app &

From source (also produces a standalone `~/Applications/shim.app`):

    git clone https://github.com/MundaneMann1776/shim ~/Documents/shim
    cd ~/Documents/shim
    python3 -m venv .venv
    .venv/bin/pip install -e .
    bin/build-app
    open ~/Applications/shim.app

## Use

Menubar:

    ● Active: ChatGPT Subscription (native)
    ─────
    ChatGPT Subscription (native, no shim)
    ─────
    DeepSeek                                  ▶
    OpenRouter (334 models)                   ▶
    ─────
    Reasoning (medium)                        ▶
    Manage API Keys…

Click a provider's model → managed block written, shim started, Codex picks
it up on its next request. Click ChatGPT Subscription → block stripped,
shim stopped, Codex talks to chatgpt.com directly via OAuth.

CLI:

    codex-shim start
    codex-shim model use <slug>
    codex-shim status
    codex-shim stop
    codex-shim restart
    codex-shim list
    codex-shim app [path]
    codex-shim codex -- <args>

All commands accept `--settings <path>` and `--port <port>`.

## Routing

    Codex Desktop ── /v1/responses ──▶ shim (127.0.0.1:8765)
                                         │
                                         ├── slug "gpt-5.5"  (optional, shim-routed)
                                         │       └─▶ chatgpt.com/backend-api/codex/responses
                                         │           Bearer access_token from ~/.codex/auth.json
                                         │
                                         ├── provider "openai" | "generic-…"
                                         │       └─▶ baseUrl/chat/completions
                                         │           Bearer apiKey
                                         │
                                         └── provider "anthropic"
                                                 └─▶ baseUrl/messages
                                                     x-api-key, anthropic-version

Extended-thinking blocks round-trip through `reasoning.encrypted_content`
items. Streaming is per-chunk; no buffer-the-whole-response paths.

## Codex picker patch (optional, one-time)

Codex Desktop's frontend filters models against a Statsig allowlist that
hides anything not on a hardcoded list. Without the patch, shim-routed
models work but don't render in Codex's own picker dropdown.

    codex-shim patch-app
    codex-shim restore-app   # undo

Unnecessary if you only switch from the menubar.

## What lives where

    ~/.factory/settings.json                provider keys, models, reasoningEffort
    ~/.codex/config.toml                    Codex config; managed block written here
    ~/.codex/auth.json                      OAuth tokens; read-only for the shim
    ~/Library/Application Support/codex-shim/  installed Python package + entry
    ~/Library/Logs/codex-shim/app.log       menubar launcher log
    ~/Applications/shim.app                 the menubar app
    .codex-shim/                            per-repo runtime state (gitignored)

## License

MIT. Codex Desktop is a trademark of OpenAI. This project is unaffiliated.
