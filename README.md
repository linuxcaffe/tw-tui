- Project: https://github.com/linuxcaffe/tw-tui
- Issues:  https://github.com/linuxcaffe/tw-tui/issues

# tw-tui

A terminal UI for **Taskwarrior 2.6.x** — vim-keyed, fast, context-aware.

---

## Standing on shoulders

tw-tui is a fork of [taskwarrior-tui](https://github.com/kdheepak/taskwarrior-tui) by
[Dheepak Krishnamurthy (@kdheepak)](https://github.com/kdheepak). The original is a beautifully
built Rust/ratatui application and the definitive TUI for Taskwarrior. Full documentation, feature
showcase, and the complete key-binding reference live at
[kdheepak.com/taskwarrior-tui](https://kdheepak.com/taskwarrior-tui).

**If you use Taskwarrior 3.x, use the original.** This fork exists for users who stay on 2.6.x.

---

## Why this fork

Taskwarrior 3.x introduced breaking changes that are incompatible with a large body of 2.6.x
hooks, scripts, and configs. Many users — particularly those with complex hook setups — have not
migrated and have no near-term plans to.

Upstream taskwarrior-tui followed TW 3.x and dropped Linux x86_64 binary releases in the process.
This fork is pinned at **v0.25.4**, the last upstream release that works cleanly with TW 2.6.x,
and distributed as a static Linux x86_64 binary.

---

## What's different

The source code is v0.25.4 with targeted patches:

**Performance and stability**
- All read-only `task` calls (`export`, `show`, `_get`, context reads) now pass
  `rc.hooks=off rc.gc=off rc.recurrence=off rc.verbose=nothing`. Without this, every display
  refresh fired the full hook fleet — causing 15–20 second freezes with a loaded hook setup.
- Tasks with empty `recur=` (a legacy incompatibility with the recurrence-overhaul hook) no longer
  abort the JSON export.

**Context filtering**
- Fixed: `task _get` fails silently on context names containing `:` or `,` (TW treats `:` as a
  DOM path separator). Context filter lookup now uses `task show rc.defaultwidth=0` which handles
  any context name correctly.
- Fixed: the context menu Enter key now activates the selection and dismisses the menu.
- Fixed: context switching no longer takes 6 seconds (hooks were firing on a config write).

**TASKRC isolation — `tw-tui` wrapper + `tui.rc`**

The binary writes context changes and TW system values (`nag=`) to whatever TASKRC it uses. Left
alone, it silently overwrites `~/.taskrc` on every context switch.

The `tw-tui` wrapper script and `tui.rc` template fix this:

- `tui.rc` includes `~/.taskrc` (so all your contexts, reports, and hooks load), but TW writes
  land in `tui.rc` instead of `~/.taskrc`.
- The wrapper strips `nag=` lines from `tui.rc` on each launch (TW writes these; they don't
  belong in a shared config).
- The wrapper syncs the current context from `~/.taskrc` into `tui.rc` before launch, so tw-tui
  always opens in your active context.

---

## Installation

### Via [awesome-taskwarrior](https://github.com/linuxcaffe/awesome-taskwarrior)

```bash
tw -I tw-tui
```

Installs the binary, wrapper script, and a starter `tui.rc` (preserves an existing one).

### Manual

```bash
SCRIPTS=~/.task/scripts
CONFIG=~/.task/config
RELEASE=https://github.com/linuxcaffe/tw-tui/releases/download/25.4-tw26
BASE=https://raw.githubusercontent.com/linuxcaffe/tw-tui/master

mkdir -p "$SCRIPTS" "$CONFIG"

# Binary (static musl, no runtime dependencies)
curl -fsSL "$RELEASE/taskwarrior-tui" -o "$SCRIPTS/taskwarrior-tui"
chmod +x "$SCRIPTS/taskwarrior-tui"

# Wrapper script
curl -fsSL "$BASE/tw-tui" -o "$SCRIPTS/tw-tui"
chmod +x "$SCRIPTS/tw-tui"

# Config template (skips if tui.rc already exists)
[[ -f "$CONFIG/tui.rc" ]] || curl -fsSL "$BASE/tui.rc" -o "$CONFIG/tui.rc"
```

---

## Launch

```bash
tw-tui          # directly
tw -t           # via the tw wrapper (if using awesome-taskwarrior)
```

---

## Configuration

`~/.task/config/tui.rc` — created on first install, safe to edit:

```ini
include ~/.taskrc

context=    # managed by tw-tui; synced from ~/.taskrc on launch

uda.taskwarrior-tui.task-report.show-info=true
uda.taskwarrior-tui.task-report.looping=false
uda.taskwarrior-tui.task-report.date-time-vague-more-precise=false
```

All other taskwarrior-tui UDA settings (colours, key overrides, select-on-move, etc.) are
documented at [kdheepak.com/taskwarrior-tui/configuration](https://kdheepak.com/taskwarrior-tui/configuration/).
Add them to `tui.rc`.

---

## Key bindings (defaults)

| Key | Action |
|-----|--------|
| `j` / `k` or `↓` / `↑` | Navigate list |
| `g` / `G` | Top / bottom |
| `Enter` | Task details / confirm |
| `a` | Add task |
| `d` | Mark done |
| `s` | Start / stop |
| `e` | Edit (opens `$EDITOR`) |
| `u` | Undo |
| `x` | Delete (with confirm) |
| `m` | Modify |
| `A` | Annotate |
| `c` | Context menu |
| `/` | Filter |
| `]` / `[` | Next / previous tab (Tasks · Projects · Calendar) |
| `?` | Help |
| `q` | Quit |

Full key reference and customisation: [kdheepak.com/taskwarrior-tui/keybindings](https://kdheepak.com/taskwarrior-tui/keybindings/).

---

## Building from source

Requires Rust (stable) and `musl-tools` for a fully static binary:

```bash
rustup target add x86_64-unknown-linux-musl
sudo apt install musl-tools     # or equivalent

cd ~/dev/tw-tui
cargo build --release --target x86_64-unknown-linux-musl

# Binary at:
target/x86_64-unknown-linux-musl/release/taskwarrior-tui
```

---

## Project status

**In re-development. Mostly works. Needs testing.**

The core workflow — browsing, filtering, context switching, add/done/start/stop/edit/annotate/undo
— works well. The Projects and Calendar tabs work. Hook integration (timelog, subtask prompts,
hledger-add) works because write operations deliberately do not suppress hooks.

This is a maintenance fork, not an active feature fork. Bug fixes and 2.6.x compatibility patches
welcome.

---

## Metadata

- License: MIT (upstream)
- Language: Rust
- Requires: Taskwarrior 2.6.x, Linux x86_64
- Version: 25.4-tw26
- Upstream: [kdheepak/taskwarrior-tui v0.25.4](https://github.com/kdheepak/taskwarrior-tui/releases/tag/v0.25.4)
