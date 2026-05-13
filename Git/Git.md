## Before You Start: Setup

```bash
# Install Git (if not already installed)
# Ubuntu/Debian:
sudo apt install git

# Verify installation
git --version

# Configure your identity (required for commits)
git config --global user.name "Your Name"
git config --global user.email "you@example.com"

# Set your default editor (optional, but helpful)
git config --global core.editor nano

# View your config
git config --list
```

---

## Chapter 1: Creating a Repository

A Git repository is just a folder that Git is tracking.

```bash
# Create a new project folder
mkdir my-project
cd my-project

# Initialize a Git repository
git init
```

You'll see: `Initialized empty Git repository in .../my-project/.git/`

The `.git` folder is where Git stores everything. **Never delete it manually.**

```bash
# See the hidden .git folder
ls -la
```

> **Practice:** Create a folder called `git-practice` and initialize a repo inside it. Use `ls -la` to confirm the `.git` directory exists.

---

## Chapter 2: Understanding Repository Status

`git status` is your best friend. Run it constantly.

```bash
# Check what's going on in your repo at any time
git status
```

Git files exist in one of four states:

```
Untracked    →  Git doesn't know about this file yet
Tracked      →  Git is watching this file
Modified     →  A tracked file has been changed
Staged       →  A changed file is ready to be committed
```

```bash
# Create a file
echo "Hello, Git!" > hello.txt

# Check status — you'll see hello.txt is "untracked"
git status
```

> **Practice:** Create 2-3 text files in your repo. Run `git status` after each one.

---

## Chapter 3: Adding Files to the Index (Staging Area)

The "index" or "staging area" is a holding zone between your working folder and the repository. You choose _which_ changes go into the next commit.

```bash
# Stage a single file
git add hello.txt

# Stage multiple files
git add file1.txt file2.txt

# Stage all files in the current directory
git add .

# Stage all files in the whole repo
git add -A

# Check status again — hello.txt is now "staged"
git status
```

**Why a staging area?** It lets you commit only part of your changes. Example: you fixed a bug AND started a new feature — you can commit only the bug fix now.

```bash
# Unstage a file (remove from staging without deleting it)
git restore --staged hello.txt

# Check status again
git status
```

> **Practice:**
> 
> 1. Create `notes.txt` and `ideas.txt`
> 2. Stage only `notes.txt`
> 3. Run `git status` and observe which file is staged and which is not
> 4. Stage `ideas.txt` too

---

## Chapter 4: Making a Commit

A commit is a permanent snapshot of your staged changes.

```bash
# Commit staged changes with a message
git commit -m "Add hello.txt and initial notes"

# Stage + commit all tracked files in one step (skips staging)
git commit -am "Quick fix to notes"
# Note: -a only works for already-tracked files, not new ones
```

**Writing good commit messages:**

- Use present tense: "Add feature" not "Added feature"
- Be specific: "Fix login bug when email has uppercase" not "Fix bug"
- Keep it under 72 characters for the first line

```bash
# Multi-line commit message (opens your editor)
git commit
```

> **Practice:**
> 
> 1. Stage all your files
> 2. Make your first commit with message `"Initial commit"`
> 3. Edit `hello.txt`, add some text
> 4. Stage and commit with message `"Update hello.txt with greeting"`

---

## Chapter 5: Renaming and Removing Files

### Renaming Files

```bash
# Rename a file using Git (Git tracks this as a rename, not delete+add)
git mv hello.txt greeting.txt

# Check status — you'll see "renamed: hello.txt -> greeting.txt"
git status

# Commit the rename
git commit -m "Rename hello.txt to greeting.txt"
```

If you rename using your OS (`mv` or file explorer), Git sees it as a delete + new file. Use `git mv` to keep history clean.

### Removing Files

```bash
# Remove a file and stage the deletion
git rm notes.txt

# Check status — shows "deleted: notes.txt"
git status

# Commit the deletion
git commit -m "Remove notes.txt"

# Remove a file from Git tracking but KEEP it on disk
git rm --cached ideas.txt
# Now ideas.txt is "untracked" but still exists locally
```

> **Practice:**
> 
> 1. Create `temp.txt`, add and commit it
> 2. Rename it to `permanent.txt` using `git mv`
> 3. Commit the rename
> 4. Delete `permanent.txt` using `git rm`
> 5. Commit the deletion

---

## Chapter 6: How Git Compares File Changes

Git tracks changes using a **diff** — a line-by-line comparison showing what was added or removed.

```bash
# See unstaged changes (what changed since last commit, not yet staged)
git diff

# See staged changes (what's in staging, ready to commit)
git diff --staged
# or
git diff --cached

# See all changes (staged + unstaged) vs last commit
git diff HEAD
```

**Reading diff output:**

```
--- a/greeting.txt     ← old version
+++ b/greeting.txt     ← new version
@@ -1,3 +1,4 @@       ← line numbers (old: line 1, 3 lines; new: line 1, 4 lines)
 Hello, Git!           ← unchanged line
-Old line              ← removed line (red, starts with -)
+New line              ← added line (green, starts with +)
+Another new line      ← added line
```

```bash
# Compare two specific commits
git diff abc1234 def5678

# Compare a file between two commits
git diff abc1234 def5678 -- greeting.txt
```

> **Practice:**
> 
> 1. Edit `greeting.txt` — change some text, add a new line
> 2. Run `git diff` to see unstaged changes
> 3. Stage the file with `git add`
> 4. Run `git diff` (nothing — changes are staged now)
> 5. Run `git diff --staged` (shows what you're about to commit)

---

## Chapter 7: Viewing Commit History

```bash
# Full commit history
git log

# Compact one-line format (very useful)
git log --oneline

# Show last N commits
git log -5
git log -5 --oneline

# Show history with a graph (great for branches)
git log --oneline --graph --all

# Show commits by a specific author
git log --author="Your Name"

# Show commits from last 2 weeks
git log --since="2 weeks ago"

# Show commits that touched a specific file
git log -- greeting.txt

# Show commits with what files changed
git log --stat
```

> **Practice:** Make 4-5 commits (edit files, add files, etc.) and then explore the history using different `git log` flags.

---

## Chapter 8: Viewing Commit Details

Each commit has a unique hash (like `a3f9c12`). You can inspect any commit.

```bash
# Show details of the last commit
git show

# Show a specific commit by its hash
git show a3f9c12

# Show only the files changed in a commit
git show --stat a3f9c12

# Show a specific file as it looked at a commit
git show a3f9c12:greeting.txt
```

**Anatomy of a commit:**

```
commit a3f9c12b...          ← unique SHA hash
Author: Jane Doe <jane@...> ← who made it
Date:   Mon May 13 2026     ← when

    Update greeting message  ← commit message

diff --git a/greeting.txt ...  ← what changed
```

> **Practice:**
> 
> 1. Run `git log --oneline` and copy a commit hash
> 2. Run `git show <hash>` to see that commit's details
> 3. Try `git show <hash>:greeting.txt` to see the file as it was then

---

## Chapter 9: Restoring Files

Accidentally deleted something? Made changes you want to undo?

```bash
# Discard unstaged changes to a file (restore to last commit)
git restore greeting.txt

# Discard ALL unstaged changes
git restore .

# Restore a file from a specific commit
git restore --source=a3f9c12 greeting.txt

# Unstage a file (keep changes, just remove from staging)
git restore --staged greeting.txt
```

**Important:** `git restore` on an unstaged file **permanently discards** your changes. There's no undo for this unless you have the changes committed somewhere.

```bash
# Restore a deleted file
git rm greeting.txt           # accidentally delete
git restore greeting.txt      # bring it back!
```

> **Practice:**
> 
> 1. Edit `greeting.txt` with some garbage text
> 2. Use `git restore greeting.txt` to undo it
> 3. Verify the file is back to its original state
> 4. Delete a file, then restore it

---

## Chapter 10: Branching

A branch is an independent line of development. Think of it as a parallel universe for your code.

```
main:     A → B → C
                    \
feature:             D → E → F
```

```bash
# List all branches (* marks the current one)
git branch

# Create a new branch
git branch feature-login

# Switch to a branch
git switch feature-login
# (older command: git checkout feature-login)

# Create AND switch in one step
git switch -c feature-signup

# Delete a branch (safe — won't delete if unmerged)
git branch -d feature-login

# Force delete a branch
git branch -D feature-login

# Rename current branch
git branch -m new-name

# See all branches including remotes
git branch -a
```

**How branches work under the hood:**

- A branch is just a pointer to a commit
- `HEAD` points to the branch you're currently on
- When you commit, the branch pointer moves forward

```bash
# See where HEAD and branches point
git log --oneline --graph --all
```

> **Practice:**
> 
> 1. On `main`, create and commit a file `app.txt`
> 2. Create branch `feature-a` and switch to it
> 3. Create and commit `feature.txt` on that branch
> 4. Switch back to `main` — notice `feature.txt` is gone!
> 5. Switch back to `feature-a` — it's back!

---

## Chapter 11: Working with Branches

```bash
# See which branch you're on
git status
git branch

# See commit history across all branches
git log --oneline --graph --all

# Compare two branches
git diff main feature-a

# See commits in feature-a that are NOT in main
git log main..feature-a --oneline
```

**A typical workflow:**

```bash
# 1. Start on main, make sure it's clean
git switch main
git status

# 2. Create feature branch
git switch -c feature-navbar

# 3. Do your work, make commits
echo "Navbar code" > navbar.txt
git add navbar.txt
git commit -m "Add navbar component"

# 4. More commits...
echo "Navbar styles" >> navbar.txt
git commit -am "Add navbar styles"

# 5. Go back to main when done
git switch main
```

> **Practice:**
> 1. Create two branches: `feature-header` and `feature-footer`
> 2. On each branch, create a different file and make 2 commits
> 3. Use `git log --oneline --graph --all` to visualize all three branches

---

## Chapter 12: Merging — Fast Forward Merge

A **fast-forward merge** happens when the branch you're merging in is directly ahead of the current branch — no diverging commits.

```
Before:  main: A → B
                    \
         feature:    C → D

After ff-merge:  main: A → B → C → D
```

No new "merge commit" is created — Git simply moves the pointer forward.

```bash
# Setup example
git switch main
git switch -c hotfix
echo "Bug fix" > fix.txt
git add fix.txt
git commit -m "Fix critical bug"

# Switch back to main and merge
git switch main
git merge hotfix
# Output: "Fast-forward"

git log --oneline
# You'll see fix.txt commit now on main
```

```bash
# Clean up the branch
git branch -d hotfix
```

**When does fast-forward happen?** When `main` has had no new commits since you created the branch. Main is the "ancestor" of the feature branch.

> **Practice:**
> 1. On `main`, create `base.txt` and commit
> 2. Create branch `ff-test`
> 3. On `ff-test`, add `extra.txt` and commit
> 4. Switch to `main`, run `git merge ff-test`
> 5. Confirm it says "Fast-forward" and the commit is now on main

---

## Chapter 13: Merging — Fast Forward in Action

Let's see what `git log` looks like before and after:

```bash
# Setup
git switch main
echo "Line 1" > story.txt
git add story.txt
git commit -m "Start story"

git switch -c chapter2
echo "Line 2" >> story.txt
git commit -am "Add chapter 2"
echo "Line 3" >> story.txt
git commit -am "Add chapter 3"

# See the divergence
git log --oneline --graph --all
# * abc1234 (chapter2) Add chapter 3
# * def5678 Add chapter 2
# * 111aaaa (HEAD -> main) Start story

# Merge
git switch main
git merge chapter2

# See the result
git log --oneline --graph --all
# * abc1234 (HEAD -> main, chapter2) Add chapter 3
# * def5678 Add chapter 2
# * 111aaaa Start story
```

Notice: no extra merge commit, history is perfectly linear.

---

## Chapter 14: Merging — Content Merge (No Conflict)

A **content merge** happens when both branches have new commits and Git can combine them automatically because they changed _different parts_ of the code.

```
main:    A → B → C
              \
feature:       D → E

Result:  A → B → C → M   (M is a new "merge commit")
                  \   /
                   D → E
```

```bash
# Setup
git switch main
echo "Main line 1" > collab.txt
git add collab.txt
git commit -m "Initial collab.txt"

# Create branch and add to TOP of file
git switch -c feature-top
sed -i '1s/^/TOP LINE\n/' collab.txt
git commit -am "Add top line"

# Switch to main and add to BOTTOM
git switch main
echo "BOTTOM LINE" >> collab.txt
git commit -am "Add bottom line"

# Now merge — Git can figure this out
git merge feature-top -m "Merge feature-top into main"
```

Git will create a **merge commit** automatically. Open `collab.txt` and you'll see both changes.

```bash
git log --oneline --graph --all
# Shows the branching and merge point clearly
```

> **Practice:**
> 1. Create `shared.txt` on `main` with 10 lines of text
> 2. On a branch, edit line 1
> 3. On `main`, edit line 10
> 4. Merge — it should succeed automatically

---

## Chapter 15: Merging — Content Merge Conflict

A **conflict** occurs when both branches changed the **same line** of the same file. Git can't decide which version to keep — it asks YOU to resolve it.

```bash
# Setup conflict scenario
git switch main
echo -e "Line 1\nLine 2\nLine 3" > conflict.txt
git add conflict.txt
git commit -m "Add conflict.txt"

# Branch A changes Line 2
git switch -c branch-a
sed -i 's/Line 2/Line 2 from branch-a/' conflict.txt
git commit -am "Change line 2 in branch-a"

# Main also changes Line 2
git switch main
sed -i 's/Line 2/Line 2 from main/' conflict.txt
git commit -am "Change line 2 in main"

# Try to merge — CONFLICT!
git merge branch-a
```

Git will say:

```
CONFLICT (content): Merge conflict in conflict.txt
Automatic merge failed; fix conflicts and then commit the result.
```

**Open `conflict.txt`** and you'll see:

```
Line 1
<<<<<<< HEAD
Line 2 from main
=======
Line 2 from branch-a
>>>>>>> branch-a
Line 3
```

**Reading conflict markers:**

- `<<<<<<< HEAD` — start of YOUR version (current branch)
- `=======` — separator
- `>>>>>>> branch-a` — start of THEIR version (branch being merged)

**Resolve the conflict:**

1. Delete the conflict markers
2. Keep what you want (one side, both, or a combination)

```
Line 1
Line 2 combined from main and branch-a
Line 3
```

```bash
# After editing, stage and commit
git add conflict.txt
git commit -m "Resolve merge conflict in conflict.txt"
```

> **Practice:**
> 
> 1. Create a file with 5 lines
> 2. On two branches, change the SAME line differently
> 3. Merge and resolve the conflict manually
> 4. Commit the resolution

---

## Chapter 16: Merging — File List Merge

This is when one branch **adds or deletes a file** while another branch **modifies it**.

### Scenario 1: One branch adds a file, other branch modifies it

Usually no conflict — Git handles this automatically.

### Scenario 2: Both branches delete the same file

```bash
git switch main
echo "temp" > deleteme.txt
git add deleteme.txt
git commit -m "Add deleteme.txt"

git switch -c branch-x
git rm deleteme.txt
git commit -m "Delete deleteme.txt in branch-x"

git switch main
git rm deleteme.txt
git commit -am "Also delete deleteme.txt in main"

git merge branch-x
# Git is fine — both deleted it, same result
```

### Scenario 3: One branch deletes, other modifies

```bash
git switch main
echo "important stuff" > important.txt
git add important.txt
git commit -m "Add important.txt"

git switch -c branch-y
git rm important.txt
git commit -m "Delete important.txt"

git switch main
echo "more stuff" >> important.txt
git commit -am "Modify important.txt"

git merge branch-y
# CONFLICT! One deleted it, one modified it
# Git will ask: keep or delete?
```

Git says:

```
CONFLICT (modify/delete): important.txt deleted in branch-y
and modified in HEAD. Version HEAD of important.txt left in tree.
```

Resolve:

```bash
# Option 1: Keep the file
git add important.txt

# Option 2: Delete the file
git rm important.txt

# Then commit
git commit -m "Resolve modify/delete conflict"
```

---

## Chapter 17: Three-Way Merge in Git

When two branches have **diverged** (both have commits the other doesn't), Git uses a **three-way merge**.

Git looks at three snapshots:

1. **Base** — the common ancestor commit (where both branches split from)
2. **Ours** — tip of current branch (HEAD)
3. **Theirs** — tip of branch being merged

```
         Base
        /    \
     Ours   Theirs
        \    /
         Merge
```

Git's logic per line/section:

- If only Ours changed it → use Ours
- If only Theirs changed it → use Theirs
- If both changed it the same way → use that version
- If both changed it differently → **CONFLICT**, ask the user

```bash
# Visualize the merge base
git merge-base main feature-branch
# Returns the commit hash of the common ancestor
```

**Example:**

```bash
git switch main
echo -e "A\nB\nC" > abc.txt
git add abc.txt
git commit -m "Base: A B C"

git switch -c branch-3way
sed -i 's/B/B-modified-by-branch/' abc.txt
git commit -am "Branch changes B"

git switch main
sed -i 's/A/A-modified-by-main/' abc.txt
git commit -am "Main changes A"

git merge branch-3way
# Git does 3-way merge:
# A-modified-by-main (only main changed A) ✓
# B-modified-by-branch (only branch changed B) ✓
# C (neither changed C) ✓
# No conflict!
cat abc.txt
```

---

## Chapter 18: Working with Merges

### Viewing merge state

```bash
# See if you're in the middle of a merge
git status
# Shows "You have unmerged paths"

# See which files have conflicts
git diff --name-only --diff-filter=U

# See all conflicts at once
git diff
```

### Aborting a merge

```bash
# Cancel the merge and go back to pre-merge state
git merge --abort
```

### Merge strategies

```bash
# Merge but don't fast-forward (always create a merge commit)
git merge --no-ff feature-branch

# Squash all commits from branch into one staged change
git merge --squash feature-branch
git commit -m "Squash feature-branch into one commit"
```

**When to use `--no-ff`:** When you want to preserve the fact that a feature branch existed, even if a fast-forward was possible.

```
With --no-ff:    A → B → C → M
                          \  /
                           D
```

> **Practice:**
> 
> 1. Create a branch with 3 commits
> 2. Merge with `--no-ff`
> 3. Compare the log to a regular fast-forward merge

---

## Chapter 19: Working with Merge Conflicts (Advanced)

### Using a merge tool

```bash
# Configure a merge tool (e.g., vimdiff)
git config --global merge.tool vimdiff

# Launch the tool for all conflicts
git mergetool
```

### Viewing conflict context

```bash
# During a conflict, see the common ancestor version
git show :1:conflicted.txt   # base version
git show :2:conflicted.txt   # our version (HEAD)
git show :3:conflicted.txt   # their version
```

### Keeping one side entirely

```bash
# Accept ALL of our changes for a file
git checkout --ours conflicted.txt
git add conflicted.txt

# Accept ALL of their changes for a file
git checkout --theirs conflicted.txt
git add conflicted.txt
```

### Conflict prevention tips

1. Pull/merge `main` into your feature branch frequently
2. Keep branches short-lived
3. Communicate with teammates about who owns which files
4. Use smaller, focused commits

---

## Chapter 20: Undoing Commits

This is where many people get confused. There are several ways to undo, each for a different situation.

### Option 1: `git revert` — Safe undo (recommended)

Creates a _new_ commit that undoes a previous commit. Safe for shared branches.

```bash
# Undo the last commit (creates a new "revert" commit)
git revert HEAD

# Undo a specific commit
git revert a3f9c12

# Revert without auto-committing (lets you edit the message)
git revert --no-commit HEAD
git commit -m "Revert broken feature"
```

### Option 2: `git reset` — Rewrite history (use with caution)

Moves the branch pointer backwards. **Never use on shared/pushed branches.**

```bash
# --soft: undo commit, keep changes staged
git reset --soft HEAD~1
# Your changes are staged, ready to recommit

# --mixed (default): undo commit, unstage changes, keep files
git reset HEAD~1
git reset --mixed HEAD~1

# --hard: undo commit AND discard all changes (DANGEROUS)
git reset --hard HEAD~1

# Reset to a specific commit
git reset --hard a3f9c12
```

`HEAD~1` means "1 commit before HEAD". `HEAD~3` means 3 commits back.

### Option 3: `git commit --amend` — Fix the last commit

```bash
# Change the last commit message
git commit --amend -m "Better commit message"

# Add a forgotten file to the last commit
git add forgotten.txt
git commit --amend --no-edit
```

**Only amend commits that haven't been pushed/shared.**

### Summary table

|Command|Undoes commit?|Keeps changes?|Safe for shared?|
|---|---|---|---|
|`revert`|Yes (new commit)|Yes|✅ Yes|
|`reset --soft`|Yes|Staged|❌ No|
|`reset --mixed`|Yes|Unstaged|❌ No|
|`reset --hard`|Yes|No (deleted)|❌ No|
|`amend`|Modifies last|Yes|❌ No|

> **Practice:**
> 
> 1. Make 3 commits
> 2. Use `git revert HEAD` to undo the last one
> 3. Make 3 more commits
> 4. Use `git reset --soft HEAD~2` and examine the state
> 5. Use `git reset --hard HEAD~1` (careful — changes are gone!)

---

## Bonus Chapter 21: The `.gitignore` File

Tell Git which files to never track.

```bash
# Create .gitignore in your project root
touch .gitignore
```

Example `.gitignore`:

```
# OS files
.DS_Store
Thumbs.db

# Logs
*.log
logs/

# Build output
/dist
/build

# Dependencies
node_modules/

# Environment variables
.env
.env.local

# Editor settings
.vscode/
.idea/
```

```bash
# After creating .gitignore
git add .gitignore
git commit -m "Add .gitignore"

# Check if a file is ignored
git check-ignore -v filename.log
```

---

## Bonus Chapter 22: Stashing

Temporarily shelve changes without committing them.

```bash
# Save current changes to stash
git stash

# List all stashes
git stash list

# Apply the most recent stash (keeps it in stash)
git stash apply

# Apply and remove from stash
git stash pop

# Stash with a name
git stash save "work in progress on login form"

# Apply a specific stash
git stash apply stash@{2}

# Delete a stash
git stash drop stash@{0}

# Clear all stashes
git stash clear
```

**When to use stash:** You're halfway through work when you need to switch branches urgently.

```bash
# Typical workflow
git stash          # shelve current work
git switch main    # switch branches
# ... do urgent fix, commit ...
git switch feature-branch
git stash pop      # get your work back
```

---

## Bonus Chapter 23: Tags

Mark specific commits (like version releases) with a memorable name.

```bash
# Create a lightweight tag
git tag v1.0

# Create an annotated tag (recommended — stores extra info)
git tag -a v1.0 -m "First stable release"

# Tag a specific commit
git tag -a v0.9 a3f9c12 -m "Beta release"

# List all tags
git tag

# Show tag details
git show v1.0

# Delete a tag
git tag -d v1.0
```

---

## Quick Reference Card

```
SETUP
git init                          Create a new repo
git config --global user.name ""  Set your name

STATUS & HISTORY
git status                        See file states
git log --oneline                 See commit history
git log --oneline --graph --all   Visual branch history
git diff                          See unstaged changes
git diff --staged                 See staged changes
git show <hash>                   See commit details

STAGING & COMMITTING
git add <file>                    Stage a file
git add .                         Stage everything
git commit -m "message"           Commit staged changes
git commit -am "message"          Stage+commit tracked files

FILE OPERATIONS
git mv old.txt new.txt            Rename a file
git rm file.txt                   Delete a file
git restore file.txt              Discard unstaged changes
git restore --staged file.txt     Unstage a file

BRANCHES
git branch                        List branches
git branch <name>                 Create branch
git switch <name>                 Switch branch
git switch -c <name>              Create+switch
git branch -d <name>              Delete branch

MERGING
git merge <branch>                Merge branch in
git merge --no-ff <branch>        Merge without fast-forward
git merge --abort                 Cancel a merge

UNDOING
git revert HEAD                   Safe undo (new commit)
git reset --soft HEAD~1           Undo commit, keep staged
git reset --hard HEAD~1           Undo commit, discard changes
git commit --amend                Fix last commit

STASHING
git stash                         Shelve changes
git stash pop                     Restore shelved changes
```

---

## Suggested Practice Project

Build a small story collaboratively with yourself across branches:

1. `main`: Create `story.txt` with an opening paragraph
2. Branch `chapter-1`: Add chapter 1
3. Branch `chapter-2`: Add chapter 2 (from main, so it diverges from chapter-1)
4. Merge `chapter-1` into main (fast-forward)
5. Merge `chapter-2` into main (content merge — possibly conflict)
6. Branch `edit-chapter-1`: Rewrite chapter 1
7. Also edit chapter 1 on `main`
8. Merge `edit-chapter-1` (conflict — resolve it)
9. Use `git log --oneline --graph --all` to see the full history

Sources:
[Inter git](https://inter-git.com/home)

Tags:
#development 