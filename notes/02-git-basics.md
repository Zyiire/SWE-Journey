# 02 · Git: What a Commit Really Is

> **Phase 1 · Week 1** — Not "how do I save my work," but **what Git is actually storing.**

Most Git confusion comes from learning the commands before the model. Learn the model first: Git
is a **content tracker** that stores **snapshots**, and every snapshot **points at the one before
it**. Every command below is just a way of moving files between three places, or of inspecting
that chain of snapshots.

**Contents**
1. [The three areas](#1-the-three-areas)
2. [What a commit really is](#2-what-a-commit-really-is)
3. [The commands](#3-the-commands)
4. [`.gitignore`](#4-gitignore)
5. [Cheat sheet](#cheat-sheet)

---

## 1. The three areas

Every file you touch lives in one of three places. **This is the whole mental model.**

```text
  WORKING DIRECTORY        STAGING AREA            REPOSITORY
  (your actual files)      (the "index")           (.git — permanent history)

  ┌───────────────┐        ┌───────────────┐       ┌───────────────┐
  │  edited.py    │        │               │       │   ● commit    │
  │  notes.md     │        │   next        │       │   ● commit    │
  │  draft.txt    │        │   commit      │       │   ● commit    │
  │               │        │   in progress │       │               │
  └───────┬───────┘        └───────┬───────┘       └───────────────┘
          │                        │                       ▲
          │  git add               │  git commit           │
          └───────────────────────►└──────────────────────►┘

          ◄─────────────────── git diff ──────────────────►
             (working vs staged)      (staged vs repo:
                                       git diff --staged)
```

| Area | Also called | Holds | You put things there with |
|---|---|---|---|
| **Working directory** | the worktree | The files you're editing right now | Your editor |
| **Staging area** | the **index** | The *draft* of your next commit | `git add` |
| **Repository** | `.git/` | Every commit ever made, permanently | `git commit` |

> **Why staging exists:** it lets you commit *some* of your changes and not others. You edited
> five files but only three belong in this commit — stage those three. A commit should be **one
> logical change**, not "everything I did today."

**The file lifecycle** — a file is always in exactly one of these states:

```text
untracked ──git add──► staged ──git commit──► committed ──edit it──► modified ──git add──► staged
 (Git has                                    (clean —                (Git sees
  never seen it)                              matches HEAD)           a difference)
```

---

## 2. What a commit really is

A commit is **two things**:

1. A **snapshot** — the complete state of every tracked file at that moment. *Not* a diff, not a
   list of changes. A full picture. (Git deduplicates unchanged files internally, so this is
   cheap, but conceptually: **full snapshot**.)
2. A **pointer to its parent** — the commit that came immediately before it.

That's it. Chain those together and you get history:

```text
                                    HEAD
                                      │
                                      ▼
                                  ┌────────┐
                                  │ main   │   ← a branch is just a moving pointer
                                  └───┬────┘
                                      │
   ┌──────────┐    ┌──────────┐    ┌──▼───────┐
   │ a1b2c3d  │◄───┤ e4f5g6h  │◄───┤ i7j8k9l  │
   │          │    │          │    │          │
   │ snapshot │    │ snapshot │    │ snapshot │
   │ + parent │    │ + parent │    │ + parent │
   └──────────┘    └──────────┘    └──────────┘
    first commit                    newest commit
    (no parent)

   ARROWS POINT BACKWARD. Each commit knows its parent;
   a commit never knows its children.
```

**Three consequences worth internalizing:**

- **History is append-only and backward-linked.** Git walks *backward* from `HEAD`. This is why
  `git log` shows newest first — it's reading the chain in the only direction it can.
- **A commit hash is a fingerprint of its content *and* its parent.** Change anything in the past
  and every hash after it changes. This is why history is tamper-evident, and why rewriting
  published history causes problems.
- **A branch is not a container.** `main` is a 40-character file pointing at one commit. Making a
  branch is cheap because it's just a new pointer.

### Vocabulary

| Term | Meaning |
|---|---|
| **`HEAD`** | A pointer to *where you are now* — usually to a branch, which points to a commit |
| **Hash / SHA** | The commit's unique ID (`a1b2c3d…`). Short form (7 chars) is usually enough |
| **Parent** | The commit this one was built on top of |
| **Tracked** | Git knows about this file — it was in the last commit or has been staged |
| **Clean** | Working directory matches `HEAD`; nothing to commit |

---

## 3. The commands

### `git init` — start tracking

```bash
git init            # creates the .git/ folder in the current directory
```

Creates one hidden folder, `.git/`. **That folder *is* the repository** — the entire history
lives there. Delete it and you have plain files again.

> Run it in the project root, **once**. Running it inside an existing repo by accident creates a
> nested repo that will confuse you later.

### `git status` — where am I

```bash
git status          # the one you run constantly
git status -s       # short format
```

**Run this before and after every other command.** It tells you what's modified, what's staged,
and what's untracked. It's how you build the three-area model into intuition.

### `git add` — stage changes

```bash
git add file.py         # one file
git add .               # everything in the current folder, recursively
git add -p              # interactively — stage HUNK BY HUNK within a file
```

> **`git add -p` is the one to actually learn.** It walks you through each change and asks
> whether to stage it. It's how you keep commits to one logical idea, and it forces you to read
> your own diff before committing.

### `git commit` — take the snapshot

```bash
git commit -m "Add user login validation"
git commit                  # opens your editor — use this for real messages
git commit -am "..."        # stages tracked-and-modified files, then commits
```

> [!WARNING]
> `git commit -am` **skips untracked files entirely.** A new file you never `git add`-ed will not
> be committed and you won't be told. Run `git status` first.

**Message convention** — imperative mood, as if completing "This commit will…":

```text
✅  Add password reset endpoint
✅  Fix off-by-one in pagination
❌  added stuff
❌  fixes
❌  Fixed the thing I broke earlier lol
```

### `git log` — read the chain

```bash
git log                                  # full history, newest first
git log --oneline                        # one line per commit — the daily driver
git log --oneline --graph --all          # visual branch structure
git log -p                               # each commit WITH its diff
git log -3                               # last three only
```

You are literally walking the parent pointers backward from `HEAD`.

### `git diff` — what changed, exactly

**Which diff you get depends on which two areas you're comparing** — this is where the three-area
model pays off:

| Command | Compares | Answers |
|---|---|---|
| `git diff` | working ↔ staging | "What have I changed but **not staged**?" |
| `git diff --staged` | staging ↔ last commit | "What's **about to be committed**?" |
| `git diff HEAD` | working ↔ last commit | "What's changed since my last commit, total?" |
| `git diff a1b2c3d e4f5g6h` | two commits | "What changed between these two points?" |

> **Habit worth building:** `git diff --staged` right before every `git commit`. It's the last
> chance to catch a debug print or a stray API key.

---

## 4. `.gitignore`

A plain text file at the repo root listing what Git should **never track**: secrets, dependencies,
build output, OS junk.

```gitignore
# Secrets — never commit these
.env
*.key

# Python
__pycache__/
venv/
*.pyc

# Node
node_modules/

# macOS
.DS_Store

# Build output
dist/
build/
```

**Pattern rules:** `folder/` = a directory · `*.log` = wildcard · `!keep.log` = negate/exception ·
`#` = comment.

> [!WARNING]
> **`.gitignore` only affects *untracked* files.** If you already committed `.env`, adding it to
> `.gitignore` changes nothing — it's tracked now. Untrack it with:
> ```bash
> git rm --cached .env
> ```
> And if a secret was ever pushed, **treat it as compromised and rotate it.** It's in the history
> permanently, even after you delete the file.

**Commit the `.gitignore` itself** — it's part of the project, and everyone cloning needs it.

---

## Cheat sheet

```text
START      git init                    create .git/ — once, in the project root

LOOK       git status                  what's modified / staged / untracked   ← run constantly
           git status -s               short version
           git log --oneline           the history chain, newest first
           git diff                    working  vs  staged
           git diff --staged           staged   vs  last commit   ← before every commit
           git diff HEAD               working  vs  last commit

SAVE       git add file                stage one file
           git add .                   stage everything here
           git add -p                  stage hunk by hunk         ← learn this one
           git commit -m "Add X"       snapshot the staging area

IGNORE     .gitignore                  never-track list (commit it)
           git rm --cached file        untrack a file already committed
```

### The one-sentence version

> **A commit is a full snapshot of your tracked files plus a pointer to its parent; `git add`
> moves changes from your working directory into the staging area, and `git commit` freezes the
> staging area into the repository forever.**

### Practice before moving on

1. `git init` an empty folder and run `git status` after *every single step* below.
2. Create a file → check status → `git add` it → check status again. **Name the state each time.**
3. Commit it. Edit it. Run `git diff`. Stage it. Run `git diff` again *(empty — why?)*, then
   `git diff --staged`.
4. Make three commits, then `git log --oneline` and trace the parent chain by hand.
5. Add a `.gitignore` with `.env` in it, create a `.env`, and confirm `git status` ignores it.
