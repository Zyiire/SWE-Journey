# 03 · Branching and Conflicts

> **Phase 1 · Week 1** — A branch is a pointer. A conflict is a question. Neither is scary once
> you've seen the shape of it three times.

[Note 02](./02-git-basics.md) ended on the key idea: **a branch is just a movable pointer to a
commit.** Everything here follows from that. Creating a branch writes a 41-byte file. Merging
compares three commits. And a conflict is Git saying *"two branches changed the same lines and I
won't guess which one you meant."*

The second half of this note is a **drill**: deliberately break things three times, by hand.

**Contents**
1. [Branches are pointers](#1-branches-are-pointers)
2. [The commands](#2-the-commands)
3. [Two kinds of merge](#3-two-kinds-of-merge)
4. [Anatomy of a conflict](#4-anatomy-of-a-conflict)
5. [The drill — three conflicts by hand](#5-the-drill--three-conflicts-by-hand)
6. [Cheat sheet](#cheat-sheet)

---

## 1. Branches are pointers

```text
                                        HEAD
                                          │
                                          ▼
                                     ┌─────────┐
                                     │ feature │
                                     └────┬────┘
                                          │
                              ┌───────────▼──┐    ┌──────────────┐
                              │   d4e5f6     │◄───┤   g7h8i9     │
                              └──────────────┘    └──────────────┘
                                   ▲
   ┌──────────┐    ┌──────────┐    │
   │  a1b2c3  │◄───┤  c3d4e5  │◄───┘
   └──────────┘    └────┬─────┘
                        │
                   ┌────▼───┐
                   │  main  │
                   └────────┘

   Two branches = two pointers into the SAME commit chain.
   They share history up to c3d4e5 — the COMMON ANCESTOR.
```

| Thing | What it actually is |
|---|---|
| **A branch** | A file in `.git/refs/heads/` containing one commit hash |
| **`HEAD`** | A pointer to the branch you're currently on |
| **Creating a branch** | Writing that one file — **instant, no copying** |
| **Committing** | Moving the current branch's pointer forward to the new commit |
| **Common ancestor** | The last commit two branches share — where they diverged |

> **The mental shift:** you are not "copying the project." You're adding a second bookmark into
> one chain of snapshots. That's why branching in Git is cheap and branching in older tools wasn't.

---

## 2. The commands

### `git branch` — create and list

```bash
git branch                    # list local branches (* marks current)
git branch -v                 # list, with each branch's latest commit
git branch feature-login      # CREATE a branch — but stay where you are
git branch -a                 # include remote-tracking branches
```

> `git branch <name>` **creates without switching.** A common early confusion: you make the
> branch, keep committing, and later find all the commits went to `main`. Check `git status`.

### `git switch` — move between branches

```bash
git switch main               # move HEAD to an existing branch
git switch -c feature-login   # CREATE and switch, in one step  ← the everyday one
git switch -                  # back to the previous branch
```

> **Use `switch`, not `checkout`.** `git checkout` still works and you'll see it everywhere in
> older docs, but it's overloaded — it switches branches *and* restores files *and* detaches
> `HEAD`. `switch` (branches) and `restore` (files) split that into two honest commands.

**Switching rewrites your working directory** to match that branch's snapshot. Git refuses if you
have uncommitted changes that would be overwritten — commit or `git stash` first.

### `git merge` — bring another branch's work into yours

```bash
git switch main               # 1. STAND ON the branch receiving the work
git merge feature-login       # 2. name the branch you're pulling IN
```

> **Direction matters and is always the same:** switch to the **destination** first, then merge
> the **source** into it. Merging is not symmetric — `main ← feature` is not `feature ← main`.

```bash
git merge --abort             # conflict? bail out, return to pre-merge state  ← your escape hatch
git merge --no-ff feature     # force a merge commit even if fast-forward is possible
```

### `git branch -d` — delete

```bash
git branch -d feature-login   # safe delete: refuses if NOT merged
git branch -D feature-login   # force delete: discards unmerged work
```

You're deleting a **pointer**, not commits. If the work was merged, it's safely in `main`'s
history. Delete branches once merged — stale branches are noise.

> [!WARNING]
> `-D` on an unmerged branch orphans those commits. They're recoverable via `git reflog` for a
> couple of weeks, but don't rely on that. Lowercase `-d` exists to protect you — **let it.**

---

## 3. Two kinds of merge

Which one you get is not a choice — it depends on whether `main` moved while you were away.

### Fast-forward — `main` didn't move

```text
BEFORE                                  AFTER  git merge feature

  main                                    main moved forward. No new commit.
   │                                       │
   ▼                                       ▼
   ●───●───●───●                           ●───●───●───●
               ▲                                       ▲
            feature                                 feature
```

Nothing to reconcile — Git just slides the `main` pointer forward. **A fast-forward can never
conflict.**

### Three-way merge — both branches moved

```text
BEFORE                                  AFTER  git merge feature

           ●───●  feature                          ●───●
          ╱                                       ╱     ╲
   ●───●─●                                 ●───●─●───────● ← MERGE COMMIT
          ╲                                       ╲     ╱    (two parents)
           ●───●  main                             ●───●
                                                          ▲
                                                        main
```

Git looks at **three** commits — the common ancestor, your tip, and their tip — and works out what
each side changed since they diverged. It merges automatically **unless both sides changed the
same lines**. That's the only thing that causes a conflict.

> **A merge commit is the one commit with two parents.** Everything from [Note 02](./02-git-basics.md)
> still holds — snapshot plus parent pointers, just two of them here.

---

## 4. Anatomy of a conflict

When both sides touched the same lines, Git stops, leaves the file on disk with **both versions
marked**, and waits for you.

```text
<<<<<<< HEAD                    ← everything below is from the branch you're ON
Welcome to my portfolio.
=======                         ← the dividing line
Welcome to my website.
>>>>>>> feature-copy            ← everything above is from the branch you're MERGING IN
```

| Marker | Means |
|---|---|
| `<<<<<<< HEAD` | Start of **your** version (the branch you're standing on) |
| `=======` | Divider — **not** part of either version |
| `>>>>>>> branch-name` | End of **their** version (the branch being merged in) |

**Resolving = editing the file until it says what you want, and deleting all three marker lines.**

That's the whole thing. There is no special command. **You are just editing a text file.** Git
doesn't check whether you picked one side, the other, both, or wrote something entirely new — it
only checks that you staged the result.

### The loop, every time

```bash
git status                # 1. which files are "both modified"?
# ...open each one, edit, delete the markers...
git diff --check          # 2. paranoia: warns if any markers are left behind
git add conflicted.txt    # 3. staging the file IS how you say "resolved"
git commit                # 4. no -m needed — Git pre-writes the merge message
```

> [!WARNING]
> **The most common beginner mistake is leaving a marker in the file.** `<<<<<<<` and `=======`
> are valid text to Git — it will happily commit them. Run `git diff --check` before you commit.

**Escape hatch, at any point:** `git merge --abort` puts everything back exactly as it was. You
cannot get permanently stuck in a conflict.

---

## 5. The drill — three conflicts by hand

> **Goal: make the markers boring.** Do all three reps in one sitting, in a throwaway folder.
> Type every command. Don't paste.

```bash
mkdir ~/conflict-practice && cd ~/conflict-practice && git init
```

---

### Rep 1 — one line, one file

*The simplest conflict that can exist. Learn the shape.*

```bash
# Setup: a common ancestor both branches will diverge from
echo "Hello world" > greeting.txt
git add . && git commit -m "Add greeting"

# Branch A changes the line
git switch -c branch-a
echo "Hello from branch A" > greeting.txt
git commit -am "Change greeting in A"

# Back to main, change the SAME line differently
git switch main
echo "Hello from main" > greeting.txt
git commit -am "Change greeting in main"

# Collide
git merge branch-a
```

You'll see: `CONFLICT (content): Merge conflict in greeting.txt`

```bash
git status                   # read it. "both modified" is the phrase to recognize.
open -a TextEdit greeting.txt    # or: code greeting.txt
```

**Resolve:** keep only `Hello from branch A`. Delete all three marker lines.

```bash
git diff --check             # silent = clean
git add greeting.txt
git commit
git log --oneline --graph    # SEE the merge commit and the two lines rejoining
```

**Then clean up:** `git branch -d branch-a`

---

### Rep 2 — two conflicts in one file, and an abort

*Real conflicts come in clusters. Also: prove to yourself you can always back out.*

```bash
printf "title: My Site\nauthor: someone\nyear: 2024\n" > config.txt
git add . && git commit -m "Add config"

git switch -c branch-b
printf "title: My Portfolio\nauthor: someone\nyear: 2026\n" > config.txt
git commit -am "Update config in B"

git switch main
printf "title: Zy's Site\nauthor: Zy\nyear: 2025\n" > config.txt
git commit -am "Update config in main"

git merge branch-b
```

**Two separate conflict blocks in one file** — `title` and `year`. (`author` merged cleanly: only
one side changed it. That's the three-way merge doing its job.)

**First, abort on purpose:**

```bash
git merge --abort
git status                   # clean. Nothing happened. Nothing is lost.
```

**Now redo it and resolve for real:**

```bash
git merge branch-b
```

Resolve each block **differently** — this is the point of the rep:
- `title:` → take **main's** version (`Zy's Site`)
- `year:` → take **branch-b's** version (`2026`)

```bash
git diff --check && git add config.txt && git commit
git branch -d branch-b
```

---

### Rep 3 — two files, and a blended resolution

*The version you'll actually hit at work: multiple files, and neither side is fully right.*

```bash
printf "def greet():\n    print('hi')\n" > app.py
echo "Notes about the app." > README.md
git add . && git commit -m "Add app and readme"

git switch -c branch-c
printf "def greet(name):\n    print(f'hi {name}')\n" > app.py
echo "Notes: now supports names." > README.md
git commit -am "Add name support in C"

git switch main
printf "def greet():\n    print('Hello!')\n" > app.py
echo "Notes: friendlier greeting." > README.md
git commit -am "Friendlier greeting in main"

git merge branch-c
git status                   # TWO files listed as "both modified"
```

**Resolve `app.py` by writing something that is in neither version** — combine both ideas:

```python
def greet(name):
    print(f'Hello, {name}!')
```

**Resolve `README.md`** by keeping both facts in one sentence.

```bash
git diff --check
git add app.py README.md
git commit
git log --oneline --graph --all
git branch -d branch-c
```

> **This is the rep that matters.** Resolving a conflict is not "pick a side" — it's *deciding
> what the code should say*. Git gave you both proposals; the answer is yours to write.

---

### After the three reps, you should be able to say:

- [ ] Which marker is "mine" and which is "theirs" — **without looking it up**
- [ ] That `=======` is a divider, not content
- [ ] That resolving is *just editing a file*, with no magic command
- [ ] That **`git add` is how you tell Git "resolved"**
- [ ] That `git merge --abort` always gets you out
- [ ] That a clean merge and a conflicted merge produce the **same kind of merge commit**

If any of those are still fuzzy, do a fourth rep. The markers stop being scary through repetition,
not through reading.

---

## Cheat sheet

```text
BRANCH     git branch                    list (* = current)
           git branch -v                 list + latest commit
           git switch -c name            create AND switch          ← everyday
           git switch name               move to existing branch
           git switch -                  back to previous branch

MERGE      git switch main               STAND on the destination first
           git merge feature             pull the source IN
           git merge --abort             undo a conflicted merge    ← escape hatch
           git merge --no-ff feature     force a merge commit

DELETE     git branch -d name            safe — refuses if unmerged
           git branch -D name            force — discards unmerged work

CONFLICT   git status                    which files are "both modified"
           <<<<<<< HEAD    = yours
           =======         = divider
           >>>>>>> branch  = theirs
           ...edit the file, delete all three marker lines...
           git diff --check              warn on leftover markers
           git add file                  THIS is "I resolved it"
           git commit                    message is pre-written

SEE IT     git log --oneline --graph --all
```

### The one-sentence version

> **A branch is a pointer into the same commit chain; merging replays what each side changed since
> their common ancestor, and a conflict is just Git refusing to guess when both sides edited the
> same lines — you resolve it by editing the file and staging it.**
