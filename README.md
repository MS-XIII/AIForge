<!--
  AI FORGE — README
  Placeholder assets live in assets/. Drop your animated SVGs there and
  swap the plain-text banner below for <img src="assets/banner-animated.svg">
  when you're ready. The reference repo uses pure SMIL (no JS) so they
  render natively on github.com.
-->

<div align="center">

# AI FORGE

**Red-team workbench. Two tabs. One endpoint. Generates captive portals and BadUSB payloads from natural language, then drops them on a WiFi Pineapple with one click.**

[![Platform](https://img.shields.io/badge/platform-Windows-0078d4?style=flat-square)](#)
[![Python](https://img.shields.io/badge/python-3.10%2B-3776ab?style=flat-square)](#)
[![Built with Nuitka](https://img.shields.io/badge/built%20with-Nuitka-orange?style=flat-square)](#)
[![License](https://img.shields.io/badge/license-MIT-green?style=flat-square)](#license)
[![Status](https://img.shields.io/badge/status-active-brightgreen?style=flat-square)](#roadmap)

</div>

---

## Overview

**AI FORGE** is a local, self-contained red-team GUI that turns a plain-text prompt into a finished offensive-tooling artifact. Two tabs, each targeting a different engagement surface: **Evil Portal** for captive-portal phishing infrastructure, **Bad USB** for keystroke-injection payloads. Both sit on top of any OpenAI-compatible chat-completion endpoint — OpenAI, OpenRouter, Groq, Together, vLLM, llama.cpp, Ollama, or a self-hosted box — so the LLM backend is whatever you point it at, not whatever someone else decided.

The generated artifact lands in an output pane, gets validated by you, and ships to a **WiFi Pineapple** over its REST API with one button. No copy-paste-into-SSH, no scp dance. The Pineapple is pinged, authenticated, and written to — all on a background thread so the UI never blocks.

Every generation is routed through a fixed **Repository** system prompt — a persona doctrine that produces code-first output in a consistent format. It is the same prompt on both tabs; a short tab-context suffix tells the model which workspace it's on.

The build pipeline compiles the whole thing to a native Windows binary with **Nuitka**, stripping Python bytecode from the shipped artifact and wrapping it in a set of free runtime anti-analysis guards. No `.pyc` on disk. No `pyinstxtractor` recovery. Constants are the last mile and the repo ships with an XOR path for hiding the system prompt from `strings`.

---

## Features

- **Dual-tab workbench** — `Evil Portal` and `Bad USB` share a single window, a single config file, and a single AI backend. Switch tabs, the context switches with you.

- **Pluggable LLM backend** — any endpoint speaking the OpenAI `/v1/chat/completions` spec. Set `base_url`, `api_key`, and `model` in Settings. Works against hosted providers or a local `llama.cpp` server on `localhost:8080`.

- **Repository system prompt** — every request prepends a fixed operator doctrine. Code-first, consistent format, no hedging. The prompt is a single string at the top of `evilcreations.py` and can be swapped or XOR-obfuscated without touching the rest of the code.

- **Tab-scoped context injection** — a one-line suffix is appended to the system message on each call so the model knows whether it's generating portal code or DuckyScript. Same doctrine, different surface.

- **One-click Pineapple deploy** — hit `SEND // PINEAPPLE`, the tool checks reachability at `172.16.42.1:1471`, logs in with your creds, and writes the current output to `/root/` with a timestamped filename. `.html` for portals, `.txt` for BadUSB payloads. If the Pineapple isn't up, you get an explicit error instead of a silent timeout.

- **Animated splash** — borderless, centered, fades in from alpha 0. Devil face drawn at runtime from a 16×16 colour grid, blown up 5× on a layered crimson halo. Letter-spaced `L O A D I N G`, then a pill-shaped progress bar with a glowing head that fills over ~2.5 seconds.

- **Runtime-generated icon** — `devil.ico` is written on first launch by a pure-stdlib ICO writer. No binary blobs in the repo, no PIL dependency, no external asset pipeline. The same pixel grid that draws the splash drives the `.ico`, the taskbar thumbnail, and the title-bar icon.

- **Dark red-team skin** — near-black base (`#0a0b0e`), crimson accent (`#ff3344`), mono typography throughout. Cards, accent bars, hover states, a status pill that tracks state with a coloured dot. The OS title bar is flipped to dark mode via a single `DwmSetWindowAttribute` call.

- **Config persistence** — API key, base URL, model, and all Pineapple credentials live in `~/.aiforge.json`. Loaded on start, saved on Settings dialog close. Corrupt file falls back to defaults instead of crashing.

- **Runtime hardening** — `hardening.py` runs anti-analysis guards on startup when the binary is compiled. Debugger detection (`IsDebuggerPresent`, `CheckRemoteDebuggerPresent`, `NtQueryInformationProcess` for `ProcessDebugPort` and `ProcessDebugFlags`), VM/sandbox heuristics (MAC OUI, registry markers, driver files), RE-tool process scan, and a timing check for single-stepping. Any trip exits silently with code `0xBEEF`.

- **Nuitka build, no bytecode on disk** — compile with the provided command and the shipped artifact is a native Windows binary. `pyinstxtractor` and `uncompyle6` are useless against it. Standard Nuitka already defeats the common RE pipeline; the hardening layer is the second fence.

- **XOR-hideable system prompt** — optional base64+XOR encoding path for the Repository prompt so `strings AIForge.exe | grep Repository` returns nothing. Decode path is built into the file; set `_SYSTEM_PROMPT_B64` and delete the raw literal when you want it.

---

## Repository Structure

```
ai-forge/
├── evilcreations.py          # Entry point. GUI, tabs, AI calls, Pineapple deploy.
├── hardening.py              # Runtime anti-analysis guards (called from main).
├── make_icon.py              # Standalone devil.ico generator (no GUI needed).
├── devil.ico                 # Generated on first run. Not committed.
│
├── assets/                   # Optional. Reference-repo style animated SVGs.
│   ├── banner-animated.svg   # Pure SMIL. No JS. Renders on github.com.
│   ├── logo-animated.svg     # Rotating rings / orbiting satellites.
│   └── divider-scan.svg      # Section divider.
│
├── .github/
│   ├── ISSUE_TEMPLATE/       # bug, feature, new-tab templates
│   ├── workflows/
│   │   └── build.yml         # Nuitka build + release artifact
│   └── PULL_REQUEST_TEMPLATE.md
│
├── LICENSE                   # MIT
└── README.md                 # You are here.
```

---

## Tab Reference

Flat catalog of every generation surface. Each tab is scoped to one engagement class and one output format. Both ship to the Pineapple at `/root/`.

| # | Tab | Generates | Output | Ext | Deploy Path |
|---|-----|-----------|--------|-----|-------------|
| 1 | **Evil Portal** | Captive-portal HTML/CSS/JS, login page clones, Flask/Node/PHP capture backends, deployment notes | Portal artifact | `.html` | `/root/portal_<ts>.html` |
| 2 | **Bad USB** | DuckyScript (Rubber Ducky / Hak5), Flipper Zero `.badusb`, O.MG cable payloads, companion PowerShell / bash stagers | Payload script | `.txt` | `/root/badusb_<ts>.txt` |

**Note:** Both tabs read from the same Repository prompt but append a tab-scoped context line before the model sees the request. Portal requests are biased toward complete HTML with a working backend; BadUSB requests are biased toward target-specific execution chains and clean exits.

---

## Status Indicators

| Signal | Status | Description |
|--------|--------|-------------|
| ✅ | Active | Shipped, tested, and part of the current build. |
| 🧪 | Experimental | Works, but the behaviour is subject to change between builds. |
| 🔧 | Build-only | Only fires in the compiled Nuitka binary; no-op when running `python evilcreations.py`. |
| 🚧 | Planned | Scoped, not implemented. See [Roadmap](#roadmap). |

**Feature status:**

| Feature | Status |
|---------|--------|
| Dual-tab generation (Evil Portal / Bad USB) | ✅ Active |
| OpenAI-compatible backend | ✅ Active |
| Repository system prompt | ✅ Active |
| Pineapple auto-deploy | ✅ Active |
| Animated splash | ✅ Active |
| Runtime `devil.ico` writer | ✅ Active |
| Dark red-team skin | ✅ Active |
| Runtime hardening guards | 🔧 Build-only |
| XOR system-prompt encoding | 🧪 Experimental |
| Multi-tab expansion (WiFi scanning, BLE) | 🚧 Planned |

---

## Quick Start

```bash
Download exe and run. simple.
```

First launch opens a splash, then the main window. Hit **SETTINGS** in the top-right and fill in:

- **API Key** — your provider key (OpenAI, OpenRouter, Groq, whatever you're pointing at).
- **Base URL** — the API root. Examples:
  - OpenAI: `https://api.openai.com/v1`
  - OpenRouter: `https://openrouter.ai/api/v1`
  - Local llama.cpp / Ollama: `http://localhost:8080/v1`
- **Model** — e.g. `gpt-4o-mini`, `claude-sonnet-4`, `qwen2.5-coder-32b-instruct`.
- **Pineapple Host / User / Pass / Write Endpoint** — defaults target a Mark VII on `172.16.42.1:1471`.

Save. Type a prompt on either tab. Hit **GENERATE**. Hit **SEND // PINEAPPLE** when the output looks right.

---

## Configuration

Config lives at `~/.aiforge.json`:

```json
{
  "api_key": "sk-...",
  "base_url": "https://api.openai.com/v1",
  "model": "gpt-4o-mini",
  "pineapple_host": "http://172.16.42.1:1471",
  "pineapple_user": "root",
  "pineapple_pass": "",
  "pineapple_write_endpoint": "/api/files/write"
}
```

Delete the file to reset to defaults. A corrupt file is caught and silently replaced with defaults on the next save — no crash, no partial-config limbo.

---

## Build

Compile to a single native Windows binary with Nuitka. PowerShell (backtick continuations):

```powershell
python -m nuitka --standalone --onefile `
  --windows-disable-console `
  --windows-icon-from-ico=devil.ico `
  --output-filename=AIForge.exe `
  --enable-plugin=anti-bloat `
  --nofollow-import-to=tkinter.test `
  --nofollow-import-to=unittest `
  --nofollow-import-to=doctest `
  --nofollow-import-to=pydoc `
  --remove-output `
  --assume-yes-for-downloads `
  evilcreations.py
```

CMD users: swap the backtick for `^`. The command is also a single line if you'd rather paste it that way.

What each flag does:

- `--standalone --onefile` — self-contained exe, no Python required on the target.
- `--windows-disable-console` — no cmd window alongside the GUI.
- `--windows-icon-from-ico=devil.ico` — bakes the devil face into the binary. Run `make_icon.py` first.
- `--enable-plugin=anti-bloat` — strips unused stdlib imports. Smaller binary, less surface to read.
- `--nofollow-import-to=*` — stops Nuitka pulling in test/doc modules that don't ship.
- `--remove-output` — deletes the generated `.c` folder after compiling.
- `--assume-yes-for-downloads` — lets Nuitka fetch its C compiler deps without prompting.

**First build is slow** (Python → C → exe). Subsequent builds hit Nuitka's cache and finish in seconds.

If the compiled exe won't launch, rebuild **without** `--windows-disable-console` and run it from a terminal — you'll see the traceback that was being swallowed.

---

## Hardening

`hardening.py` fires only when `__compiled__` is set in the module globals — i.e. only in the Nuitka-built exe. During development, `python evilcreations.py` runs guard-free, so you don't fight your own anti-analysis.

Layers, in order:

1. **Debugger detection** — `IsDebuggerPresent`, `CheckRemoteDebuggerPresent`, and two `NtQueryInformationProcess` calls (`ProcessDebugPort = 0x07`, `ProcessDebugFlags = 0x1F`).
2. **VM / sandbox heuristics** — MAC OUI match against common hypervisors (VMware, VirtualBox, Hyper-V, Xen, QEMU), registry markers for guest tools, and driver files in `System32`.
3. **RE-tool process scan** — `tasklist` output checked against a list of common RE / packet-capture tool names.
4. **Timing check** — a fixed-cost native loop measured for a debugger-stepping signature. Only enabled when `aggressive=True`, which is only set from the compiled path.

Any trip calls `_fail()`, which redirects stdout/stderr to `os.devnull` and exits with code `0xBEEF`. No message. No hint.

**What this stops:** casual `strings` inspection, `pyinstxtractor` / `uncompyle6` (no bytecode to recover), debugger attachment, VM-sandbox automated analysis, and a good chunk of would-be RE traffic.

**What it doesn't stop:** a Ghidra session against the native binary, or a memory dump of the running process. The Repository prompt has to exist as a Python string in RAM at generation time — that's the fundamental limit. For Nuitka-grade obfuscation of constants, that's a Commercial-license feature and not part of this repo's free stack.

---

## Contributing

PRs welcome. Before submitting:

- Every new tab or generation surface should follow the existing pattern — a `ForgeTab` subclass, a `TAB_CONTEXT` entry, a `TAB_FILENAME` entry, and a row in the [Tab Reference](#tab-reference) table above.
- Keep the UI consistent with the palette at the top of `evilcreations.py`. Don't introduce new accent colours without a reason.
- Don't add external binary assets. The repo is meant to build clean from source — the only generated binary is `devil.ico`, and it's regenerated every launch if missing.
- If you touch `hardening.py`, test against both dev (uncompiled) and compiled builds. The guards should never fire in dev mode.

When you add a tab, update:

- [README.md](README.md) — tab counts, structure tree, tab reference table.
- The `TAB_CONTEXT` dict in `evilcreations.py`.
- The `TAB_FILENAME` dict in `evilcreations.py`.
- The placeholder and subtitle strings in `App.__init__`.

---

## Tech Stack

| Layer | Choice |
|-------|--------|
| Language | Python 3.10+ |
| GUI | Tkinter / ttk (stdlib) |
| HTTP client | `requests` |
| LLM backend | Any OpenAI-compatible `/v1/chat/completions` endpoint |
| Deployment target | WiFi Pineapple Mark VII REST API |
| Build | Nuitka (`--standalone --onefile`) |
| Icon pipeline | Pure stdlib (no PIL, no external assets) |
| Hardening | `ctypes` against `kernel32`, `ntdll`, `winreg`, `uuid` |

---

## Roadmap

- [x] Dual-tab workbench (Evil Portal / Bad USB)
- [x] OpenAI-compatible backend
- [x] Repository system prompt + per-tab context injection
- [x] WiFi Pineapple auto-deploy
- [x] Animated splash with runtime-drawn devil icon
- [x] Dark red-team skin
- [x] Runtime `devil.ico` writer (pure stdlib)
- [x] Nuitka build pipeline
- [x] Runtime hardening guards
- [ ] XOR system-prompt encoding — shipped but undocumented in the default build
- [ ] Additional tabs (WiFi scanning surface, BLE payloads, shell stagers)
- [ ] Streaming token output on the current tab
- [ ] Payload history / recall across sessions
- [ ] Multi-target deploy queue (write to N pineapples in sequence)
- [ ] Animated SVG banner + logo (assets/ — pure SMIL, no JavaScript)

---

## Socials & Community

Issues, PRs, and anything that touches the code belong here on GitHub. For walkthroughs of new tabs, payload breakdowns, and general red-team chatter — drop your links:

- **YouTube** — `<your channel>`
- **Discord** — `<invite>`
- **Telegram** — `<handle>`
- **X / Twitter** — `<handle>`

---

## License

MIT. Do what you want, keep the copyright notice, no warranty.

---

<div align="center">

*Built for the operator who'd rather type a prompt than hand-write a DuckyScript.*

</div>
