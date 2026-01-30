```
ls
```

Lists files and directories in the current location.

---

```
git ls-files
```

Shows the list of files tracked by Git in the current repository.

---

```
git add .
```

Stages all changes (new, modified, deleted files) in the current directory for the next commit.

---

```
git status
```

Displays the current state of the working directory and staging area.

---


```
git commit -m "initial commit"
```

Creates a new commit with the staged changes and the message "initial commit".

---


```
git log
```

Shows the commit history of the repository.

---

```
git log --oneline
```

Shows the commit history in a condensed one-line-per-commit format.

---


```
git log file.txt
```

Shows the commit history that affected the file `file.txt`.

`etc` → similar `git log` options filter or format the commit history in different ways.

---
branch is a linear sequence of commits (initial branch named "Master" )
---

---

```
git diff
```

Displays changes in tracked files that are not yet staged for commit.


---

```
git diff --staged
```

Shows the changes in staged files compared to the last commit.


---

```
git show a0b1
```

Displays details of the commit with hash starting `a0b1`, including changes made.


---

```
git diff a0b1..935d
```

Shows the changes between commits `a0b1` and `935d`.

---

```
rm -rf .git/
```

Forcefully deletes the `.git` directory, removing all Git history and configuration from the project.

---

```
git rm --cached file.txt
```

Stops tracking `file.txt` in Git but keeps the file in your working directory.

---


```
git restore file.txt
```

Restores `file.txt` in the working directory to match the staged version (index).  

---

```
git restore --staged file.txt
```

Restores `file.txt` in the staging area (index) to match the last committed state, without changing the working directory.


---


```
git reset HEAD~1
git reset HEAD~x
```

Moves the current branch pointer back by 1 (or _x_) commits **and updates the staging area** to match, but keeps changes in the working directory.

#### **(move backward)**

---


```
git reset --hard HEAD~1
git reset --hard HEAD~x
```

Moves the current branch pointer back by 1 (or _x_) commits **and updates both the staging area and working directory**, discarding all changes.

#### **(move hard backward)**

---

```
git reflog
```

Shows a log of all recent Git reference updates, including commits that are no longer visible in `git log`.

---

```
git reset HEAD@{1}
git reset HEAD@{x}
```

Restores the branch to its state at reflog entry **1** (or _x_) **and updates the staging area** to match, but keeps changes in the working directory.

#### **(move forward)**

---

```
git reset --hard HEAD@{1}
git reset --hard HEAD@{x}
```

Restores the branch to its state at reflog entry **1** (or _x_), updating both the staging area and working directory to match that snapshot.

#### **(move hard forward)**


---

```
git tag -a v2.0 -m "version 2.0 of file"
```

Creates an **annotated tag** named `v2.0` with the message "version 2.0 of file".

---

```
git show v2.0
```

Displays details of the `v2.0` tag and the commit it points to.

---

```
git branch testing
```

Creates a new branch named `testing` without switching to it.

---


```
git branch
```

Lists all local branches, highlighting the current branch with an asterisk.

---

```
git switch testing
```

Switches the working directory to the `testing` branch.

---

```
git merge testing
```

Merges changes from the `testing` branch into the current branch (usually `master` if you switched back first).

---

```
git branch --merged
```

Lists branches whose changes have already been merged into the current branch.

---

```
git branch -d testing
```

Deletes the `testing` branch if it has already been merged.

---

```
git clone ./url
```

Creates a copy of a repository from the specified URL (or local path) into a new directory.

---

```
git remote
```

Lists the short names of remote repositories linked to the local repository.

---

```
git remote -v
```

Lists remote repositories with their URLs for fetch and push operations.

---

```
git remote add <name> <url>
```

Adds a new remote repository with the given name and URL (you can repeat with different names to add multiple remotes).

---
```
git fetch <name>
```

Downloads commits, branches, and tags from the specified remote **without** merging them into your local branches.

---

```
git push <name>
```

Uploads local commits from the current branch to the specified remote repository.

---

```
git push -u <remote-name> <branch-name>
```

Pushes a new local branch (not yet on remote) to the specified remote and sets it as upstream — usually done once; afterward, `git push`/`git pull` works without extra arguments.


---

```
git pull <name>
```

Fetches changes from the specified remote and immediately merges them into the current branch.

---

