# 01 · The Terminal

> **Phase 1 · Week 1** — Getting fluent in the shell so everything after this is faster.

The terminal is the interface every other tool in this stack assumes you know. Git, Python,
Docker, deploys — all of it runs here. The goal of this note isn't memorizing commands, it's
**not having to think about navigation anymore**.

**Contents**
1. [Navigation & file operations](#1-navigation--file-operations)
2. [Efficiency shortcuts](#2-efficiency-shortcuts)
3. [Mac-specific power moves](#3-mac-specific-power-moves)
4. [Upgrade your setup](#4-upgrade-your-setup)
5. [Cheat sheet](#cheat-sheet)

---

## 1. Navigation & file operations

The core commands for moving around and manipulating files without touching Finder.

### Where am I / what's here

| Command | Stands for | What it does |
|---|---|---|
| `pwd` | **p**rint **w**orking **d**irectory | Shows exactly where you are right now |
| `ls` | **l**i**s**t | Lists everything in the current folder |
| `ls -la` | list · **l**ong · **a**ll | Hidden files, sizes, and **permissions** |

### Moving around

`cd` — **c**hange **d**irectory — is the one you'll type most:

```bash
cd projects      # into a folder
cd ~             # your home / user profile folder
cd ..            # up one level
cd -             # back to the previous folder
```

> **Remember the three symbols:** `~` = home · `..` = parent · `.` = right here.

### Creating, copying, deleting

```bash
mkdir notes          # make a new folder
touch index.html     # create a new, blank file
cp a.txt b.txt       # copy
mv a.txt b.txt       # move — and also how you RENAME
rm old.txt           # delete a file
```

> [!WARNING]
> **`rm` does not use the Trash.** There is no undo. `rm -rf [folder]` force-deletes an entire
> folder and everything inside it, silently. Read the path twice before you hit Enter.

**Flags to know:** `-r` = recursive (applies to a folder's contents) · `-f` = force (no prompts)
· `-a` = all (including hidden) · `-l` = long/detailed.

---

## 2. Efficiency shortcuts

Speed is the actual point of terminal fluency. These four are worth drilling until they're muscle
memory: **Tab**, **Ctrl + R**, **Ctrl + C**, **Ctrl + A/E**.

| Shortcut | Action |
|---|---|
| **`Tab`** | **Auto-completes** file, folder, and command names — press twice to list options |
| `Ctrl + A` / `Ctrl + E` | Jump to the **beginning** / **end** of the line |
| `Option + ←` / `→` | Move the cursor **word by word** |
| **`Ctrl + R`** | **Reverse-search your history** for a command you typed before |
| `Ctrl + C` | **Cancel** the running command (or clear the line you're typing) |
| `clear` or `Cmd + K` | Wipe the screen of clutter |

> **Rule of thumb:** if you're typing a full path by hand, you're doing it wrong — `Tab` it.
> If you're retyping a long command, you're doing it wrong — `Ctrl + R` it.

---

## 3. Mac-specific power moves

macOS ships terminal commands that hook straight into the GUI. These are the bridge between the
shell and the rest of your Mac.

### `open` — the Finder bridge

```bash
open .                                  # open the current folder in Finder
open report.pdf                         # open a file in its default app
open -a "Safari" https://google.com     # open a URL in a specific app
```

### `pbcopy` & `pbpaste` — the clipboard

Anything that prints to the screen can be **piped** (`|`) straight into your clipboard:

```bash
ls | pbcopy              # copy the file list
pbpaste > notes.txt      # paste the clipboard into a file
```

### `caffeinate` — stop the Mac sleeping

```bash
caffeinate -u -t 3600    # stay awake for 3600 seconds (one hour)
```

### `screencapture` — screenshots from the CLI

```bash
screencapture -T 5 shot.png    # screenshot after a 5-second delay
```

---

## 4. Upgrade your setup

Power users rarely stay on stock settings. These three change the experience completely.

| Tool | What it is | Why it matters |
|---|---|---|
| **Homebrew** | The package manager for macOS | Install *anything* from the terminal |
| **Oh My Zsh** | A Zsh config framework | Themes, color coding, smart autocompletion |
| **iTerm2** | A replacement Terminal app | Split panes, hotkeys, real search |

**Homebrew** is the foundation — install it first, from the setup script on [brew.sh](https://brew.sh):

```bash
brew install wget                          # a CLI tool
brew install --cask visual-studio-code     # a GUI app
```

---

## Cheat sheet

```text
WHERE          pwd · ls · ls -la
MOVE           cd folder · cd ~ · cd .. · cd -
MAKE           mkdir name · touch file
MOVE/COPY      cp src dst · mv src dst     (mv = rename)
DELETE         rm file · rm -rf folder     ⚠️ permanent
MAC            open . · ls | pbcopy · caffeinate -u -t 3600
KEYS           Tab · Ctrl+R · Ctrl+C · Ctrl+A/E · Option+←/→
```

### Terms to remember

| Term | Meaning |
|---|---|
| **Shell** | The program reading your commands (on modern macOS: **Zsh**) |
| **Flag** | An option modifying a command — the `-la` in `ls -la` |
| **Argument** | What the command acts on — the `notes` in `mkdir notes` |
| **Path** | A file's address — *absolute* from `/`, *relative* from where you are |
| **Pipe** (`\|`) | Sends one command's output into the next: `ls \| pbcopy` |
| **Hidden file** | Any name starting with `.` (e.g. `.zshrc`) — needs `ls -a` to see |
