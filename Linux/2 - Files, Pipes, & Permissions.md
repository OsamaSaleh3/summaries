
---

## 1. Reading Files: `less`, `more`, `man`, `cat`, `head`, `tail`

### Concept Overview

Linux gives you several tools for _viewing_ file contents without opening a full editor like `vim`. Which one you use depends on the file's length and whether you need to watch it change in real time.

### Commands & Syntax

|Command|Purpose|Notes|
|---|---|---|
|`less <file>`|Opens a file in a scrollable, read-only pager (like a stripped-down `vim`).|Press `q` to quit. Supports `/searchterm` to search forward. Best for long files.|
|`more <file>`|Older pager, predecessor to `less`.|`less` was written specifically to be a more powerful "opposite" of `more` (hence the name). Prefer `less`.|
|`man <program>`|Opens the full manual page for a program.|Internally, `man` uses `less` to display its output — so if you know `less`, you already know how to navigate `man`. Manuals can be very long (hundreds/thousands of lines). Not every program has a man page.|
|`<program> --help`|Prints an abbreviated summary of a command's flags.|Faster to scan than `man`; use this when you just need a quick flag reference.|
|`cat <file>`|Dumps the **entire** file contents straight to standard output (no paging/scrolling UI).|Best for short files. Name comes from **conc**at**enate** — it concatenates the file's contents onto standard out.|
|`tail <file>`|Prints the **last 10 lines** of a file by default.|Great for glancing at the end of log files.|
|`tail -n <N> <file>`|Prints the last `N` lines.|e.g., `tail -n 3 file.txt` shows only the last 3 lines.|
|`head <file>`|Prints the **first 10 lines** by default.|Same `-n` flag works to control line count.|
|`tail -f <file>`|"Follows" a file, streaming new lines to your terminal in real time as they're appended.|This is the origin of the phrase "tailing the logs." Exit with `Ctrl+C`.|

### Code & Script Examples

```bash
# Open a file for scrollable reading
less textfile.txt
# navigate with arrow keys, search with /searchterm, quit with q

# Older, less-capable pager
more textfile.txt

# Full manual for a program
man less

# Quick abbreviated help
less --help

# Dump a short file directly to the terminal
cat textfile.txt

# Show last 10 lines (default)
tail textfile.txt

# Show only the last 3 lines
tail -n 3 textfile.txt

# Show first 50 lines
head -n 50 textfile.txt

# Watch a file for new content in real time (e.g., log tailing)
tail -f textfile.txt

# In a second terminal, appending new content:
echo hi >> textfile.txt
# The first terminal's `tail -f` session immediately shows "hi"
```

> **Note:** `less` mimics several `vim` keybindings, which is intentional — the author wanted it to feel familiar to `vim` users.

> **Note:** `man` gives exhaustive documentation but is often _more_ information than you need. In practice, developers often reach for `--help` or a web search first, and fall back to `man` only when offline or needing deep detail.

### Developer Takeaway / Best Practices

- Default to `less` for anything more than a screenful of text; use `cat` only for short files you want dumped immediately.
- `tail -f` is a daily-driver command for watching live application/server logs.
- You won't memorize every flag for every tool — invest deep learning time into the tools you use constantly (the instructor uses `git` heavily but freely admits he doesn't know all of `less`'s shortcuts).

### Hands-on Practical Exercise

1. Create a file called `practice.txt` and add at least 60 lines of arbitrary text to it (you can use a loop: `for i in $(seq 1 60); do echo "line $i" >> practice.txt; done`).
2. Open it with `less` and scroll down, then back up. Use `/line 45` to search for a specific line, then quit with `q`.
3. Open the same file with `more` and compare the experience.
4. Run `cat practice.txt` and notice the difference from the paged view.
5. Print only the first 5 lines with `head`, then only the last 5 lines with `tail`.
6. In one terminal, run `tail -f practice.txt`. In a second terminal (or a second `multipass shell` session), run `echo "new log entry" >> practice.txt` and confirm it appears live in the first terminal. Exit the tail with `Ctrl+C`.
7. Run `man tail` and `tail --help` and compare how much information each gives you.

---

## 2. File & Directory Management: `mkdir`, `touch`, `rm`, `cp`, `mv`

### Concept Overview

These are the fundamental commands for creating, duplicating, renaming, and deleting files and directories.

### Commands & Syntax

|Command|Purpose|Notes|
|---|---|---|
|`mkdir <name>`|Creates a new directory.||
|`mkdir -p <path/with/multiple/levels>`|Creates nested directories in one shot, including any missing parent directories.|Without `-p`, `mkdir` fails if parent directories don't exist.|
|`touch <file>`|Creates the file if it doesn't exist (empty file).|If the file **already exists**, `touch` simply updates its "last modified" timestamp — it does not alter file contents. This is often used by scripts as a cheap way to signal "this needs to be reprocessed."|
|`rm <file>`|Removes (deletes) a file.||
|`rm <directory>`|**Fails** — `rm` refuses to remove directories by default.||
|`rm -r <directory>`|Recursively removes a directory and everything inside it.|`-r` / `-R` generally means "recursive" — apply the operation not just to the target but everything nested inside it.|
|`rm -f`|Force — suppresses the "are you sure?" confirmation prompts.|Useful (and common) when deleting something like `node_modules` with thousands of files, so you're not prompted per file.|
|`rm -rf <target>`|Recursive + force delete.|**Extremely destructive.** There is no "recycle bin" — deleted files are gone permanently.|
|`rm -i`|Interactive — forces a yes/no confirmation for every file before deleting.|Opposite of `-f`; a safety net.|
|`cp <source> <destination>`|Copies a file.||
|`cp -R <source_dir> <destination_dir>`|Recursively copies an entire directory tree.||
|`mv <source> <destination>`|Moves a file/directory; also used to **rename** (moving something to a new name in the same directory is effectively a rename).||

> **Warning:** `rm` is one of the most dangerous commands in Linux. Unlike deleting files in a GUI (which typically goes to a Trash/Recycle Bin you can recover from), `rm` deletes are **permanent and unrecoverable**. Running something like `rm -rf *` or `rm -rf /` can delete your entire filesystem — Linux will do exactly what you tell it to, with no built-in undo. There is a third-party package called `trash` that mimics OS-level trash-bin behavior (auto-empties after ~30 days) as a safer alternative, but it must be installed separately.

### Code & Script Examples

```bash
# Create a single new directory
mkdir my-new-folder
cd my-new-folder

# Create deeply nested directories in one command
mkdir -p hi/my/name/is/Brian

# Create a file (or bump its modified time if it already exists)
touch my-new-file.txt
touch my-new-file.txt   # running again just updates the timestamp

# Remove a file
rm my-new-file.txt

# Attempting to remove a directory without -r fails:
rm my-new-folder        # ERROR: it's a directory

# Correct way to remove a directory and its contents
rm -r my-new-folder

# Force-delete without confirmation prompts (dangerous!)
rm -rf some-folder

# Interactive delete — asks for confirmation per file
rm -i file*

# Copy a file
cp txtfile.txt destination-file.txt

# Recursively copy a directory
cp -R hi bye

# Move/rename a directory
mv bye something-else

# Move/rename a file
mv destination-file.txt another-file.txt
```

> **Warning:** Never run `rm -rf *` or `rm -rf /` "just to test it." These are frequently cited as the classic catastrophic mistakes that erase entire systems.

### Developer Takeaway / Best Practices

- Use `mkdir -p` any time you need to scaffold a nested directory structure in one command.
- `touch` is a handy way to force scripts/build tools that check "modified time" to re-run.
- Always double- (or triple-) check `rm -rf` commands before hitting Enter — there is no undo.
- Consider installing the `trash` CLI package as a safer default for interactive deletion work.

### Hands-on Practical Exercise

1. Create a nested folder structure in one command: `mkdir -p project/src/components`.
2. Inside `project/src`, use `touch` to create `index.js`.
3. Run `touch index.js` again and confirm (via `ls -la`) that only the modified timestamp changed.
4. Copy `index.js` to `index.backup.js` using `cp`.
5. Recursively copy the entire `project` directory to `project-copy` using `cp -R`.
6. Rename `project-copy` to `project-archive` using `mv`.
7. Attempt `rm project-archive` (it should fail because it's a directory).
8. Remove `project-archive` correctly using `rm -r`.
9. Create three empty files in a `scratch` folder, then delete them all with `rm -i` and answer "no" to at least one prompt to see it skipped.

---

## 3. Archiving Files with `tar`

### Concept Overview

`tar` bundles multiple files/directories into a single archive file — conceptually similar to a `.zip`, but historically **not** inherently compressed. A tar archive alone just glues files together; you add compression (commonly gzip) as a separate flag/step.

### Commands & Syntax

|Command|Purpose|
|---|---|
|`tar -cf archive.tar file1 file2 dir1`|Create (`c`) an uncompressed archive into a single **f**ile.|
|`tar -czf archive.tar.gz file1 dir1`|Create + compress with gzip (`z`) into a single file. Order/combination of flags matters.|
|`tar -xzf archive.tar.gz -C destination/`|Extract (`x`) a gzip-compressed (`z`) archive **f**ile into a destination directory (`-C`). The destination directory **must already exist** — `tar` will not create it for you.|

### Code & Script Examples

```bash
# Set up sample files
mkdir folder1
cd folder1
touch file1.txt file2.txt file3.txt file4.txt
cd ..

# Create an uncompressed archive containing a file and a directory
tar -cf archive.tar textfile.txt folder1
# Result: archive.tar (uncompressed, e.g. 12 KB if source files total 12 KB)

# Create a compressed archive (gzip)
tar -czf archive.tar.gz textfile.txt folder1
# Result: archive.tar.gz is significantly smaller (e.g. 4 KB)

# Extract an archive — destination folder must already exist
mkdir extracted
mv archive.tar.gz extracted/
cd extracted
tar -xzf archive.tar.gz -C destination
# NOTE: `destination` must exist beforehand, e.g.: mkdir destination
```

> **Note:** The instructor openly admits `tar`'s flags and their required order are hard to remember and something even experienced engineers look up every time. Don't feel bad relying on `--help`, `man tar`, or a web search for this one.

### Developer Takeaway / Best Practices

- Use `tar` any time you need to bundle a project/log set/build output for transfer between machines (e.g., uploading to a server).
- Always compress (`z`/gzip) unless you have a specific reason not to — the CPU cost of compression is cheap compared to the storage/transfer savings.
- Remember: the extraction destination directory must exist before you run `tar -x ... -C destination`.

### Hands-on Practical Exercise

1. Create a directory `logs/` containing 3 dummy `.log` files with some text in each.
2. Archive `logs/` (uncompressed) into `logs-backup.tar` and note its size with `ls -lh`.
3. Create a compressed version, `logs-backup.tar.gz`, and compare file sizes.
4. Create a new empty directory called `restore/`.
5. Extract `logs-backup.tar.gz` into `restore/` using `tar -xzf logs-backup.tar.gz -C restore`.
6. Confirm the original `logs/` contents now exist inside `restore/`.

---

## 4. Wildcards, Brace Expansion, and Escaping

### Concept Overview

Bash performs **expansion** on certain special characters _before_ passing arguments to a command. This means tools like `touch` or `ls` never actually "see" the wildcard/brace syntax — Bash expands it first and hands the fully expanded list of arguments to the program. Understanding this distinction (Bash's job vs. the program's job) is key to understanding shell behavior generally.

### Commands & Syntax

|Syntax|Meaning|
|---|---|
|`{a,b,c}`|**Brace expansion** — repeats the surrounding text once per comma-separated option. `file{1,2,3}.txt` expands to `file1.txt file2.txt file3.txt`.|
|`{1..30}`|Numeric range expansion — generates every number from 1 to 30.|
|`{a..z}`|Alphabetic range expansion — works forward (`a..z`) or backward (`z..a`), and can skip via a step, e.g. `{1..100..10}`.|
|`{a..z}{1..5}`|Combines two brace expansions to produce **every permutation**: `a1 a2 a3 a4 a5 b1 b2 ...`|
|`*` (wildcard/glob)|Matches **zero or more** of any character. `file*` matches `file`, `file1`, `file-anything`, etc.|
|`?`|Matches **exactly one** character. `file??` matches two-character suffixes only.|
|`\` (backslash)|**Escape character** — forces the next character to be treated literally instead of specially (e.g., escaping a literal space in a filename: `touch a\ file`).|
|`\\`|A literal backslash character.|

### Code & Script Examples

```bash
# Brace expansion — creates file1.txt, file2.txt, file3.txt, file4.txt
touch file{1,2,3,4}.txt

# Works with arbitrary strings too
touch file{mn,wa,mt,ut}.txt

# A trailing empty entry in the list creates a "bare" name
touch file{ca,ny,}.txt   # creates file-ca.txt, file-ny.txt, and file-.txt

# Wildcard matching — anything starting with "file-"
ls file-*

# Wildcard anywhere in the pattern
ls file*.txt

# ? matches exactly one character (useful for fixed-width matching)
ls file??.txt

# Numeric range expansion (creates file1.txt ... file30.txt)
touch file{1..30}.txt

# Alphabetic range expansion
echo {a..z}          # a b c d e f g ... z
echo {z..a}          # reversed
echo {1..100..10}    # every 10th number: 1 11 21 31 ...
echo {A..c}          # ranges span the ASCII table, including symbols between Z and a

# Permutations from combined braces
echo {a..z}{1..5}    # a1 a2 a3 a4 a5 b1 b2 b3 ...

# Escaping a literal space in a filename
touch a\ file
# Result: a file with a literal space character in its name

# Escaping a literal backslash
touch file\\
```

> **Note:** Expansion happens entirely in Bash, _before_ the program runs. This is why `touch` "supports" brace expansion even though `touch` itself has no idea what a curly brace is — Bash already turned `touch file{1,2}.txt` into `touch file1.txt file2.txt` by the time `touch` receives it.

### Developer Takeaway / Best Practices

- Use brace expansion (`{1..30}`, `{a,b,c}`) any time you need to bulk-create or bulk-reference many similarly-named files/folders quickly — a huge time-saver over typing each name manually.
- `*` is used constantly in real-world filtering (`ls file*.txt`); `?` is used far less often but useful for fixed-length matching.
- Escape special characters (spaces, wildcards, backslashes) with `\` whenever a filename needs to contain them literally.

### Hands-on Practical Exercise

1. In a fresh directory, use a single `touch` command with brace expansion to create `report1.txt` through `report20.txt`.
2. Use a wildcard with `ls` to list only files ending in `.txt`.
3. Use `?` matching to list only files whose number portion is exactly one digit (e.g., `report1.txt` through `report9.txt`).
4. Use brace expansion to create files named `data-a1.txt` through `data-c5.txt` using the "permutation" trick (`{a..c}{1..5}`).
5. Create a file with a literal space in its name using the `\` escape character, then `cat` it to confirm it exists.
6. Clean up everything you created using a wildcard-based `rm` (e.g., `rm report*.txt data-*.txt`).

---

## 5. Streams & Redirection: stdout, stderr, stdin, `/dev/null`

### Concept Overview

Every running program has (at minimum) three standard I/O streams:

- **Standard Out (stdout, `1`)** — where normal output goes.
- **Standard Error (stderr, `2`)** — where error messages go, kept _separate_ from stdout so you can handle successes and failures differently (e.g., normal logs vs. error logs).
- **Standard In (stdin, `0`)** — where a program reads input from, if it accepts input.

By default, stdout and stderr both print to your terminal screen, and stdin reads from your keyboard. Redirection lets you reroute these streams to/from files instead.

### Commands & Syntax

|Syntax|Meaning|
|---|---|
|`command > file` (or `command 1> file`)|Redirect **stdout** to `file`, **overwriting** it.|
|`command >> file` (or `command 1>> file`)|Redirect **stdout** to `file`, **appending** to it.|
|`command 2> file`|Redirect **stderr** to `file`, overwriting.|
|`command 2>> file`|Redirect **stderr** to `file`, appending.|
|`command > out.txt 2> err.txt`|Redirect stdout and stderr to two **separate** files.|
|`command > combined.txt 2>&1` / `command &> combined.txt`|Redirect **both** stdout and stderr to the **same** file.|
|`command < file`|Redirect a file's contents **into** a program's stdin.|
|`command > /dev/null`|Discard stdout entirely — `/dev/null` is a special "black hole" file; anything written to it vanishes.|
|`command 2> /dev/null`|Discard only stderr (suppress error messages while keeping normal output visible).|

### Code & Script Examples

```bash
# Basic stdout redirect: overwrite
echo "this is my text" 1> new-file.txt
cat new-file.txt
# -> this is my text

# Redirecting cat's stdout effectively performs a copy
cat new-file.txt > another-file.txt

# Append (>>) instead of overwrite (>)
cat new-file.txt >> another-file.txt
cat another-file.txt
# -> "this is my text" now appears twice

# Redirecting the output of any command
ls -lsah 1> ls.txt
cat ls.txt   # note: no color codes — programs detect when output is
             # redirected (not a terminal) and switch to "machine-friendly" formatting

# stderr behaves independently of stdout
cat non-existent-file.txt              # error prints to screen
cat non-existent-file.txt 1> cat.txt   # error STILL prints to screen; cat.txt stays empty
cat non-existent-file.txt 2> error.txt # NOW the error message goes into error.txt
cat error.txt

# Redirect stdout and stderr to two different files
ls -lsah 1> ls.txt 2> ls-error.txt

# Redirect stdout and stderr to the SAME file
ls -lsah > everything.txt 2>&1

# Discard errors you don't care about
ls -lsah 2> /dev/null

# Discard normal output, keep only errors visible
cat somefile.txt > /dev/null
# (if somefile.txt doesn't exist, you'll still see the error on screen)

# Feed a file's contents INTO a program via stdin (< )
grep "ls-error.txt" < ls.txt
# Reads ls.txt, feeds it into grep's stdin, grep searches for the given string
```

> **Note:** `1` = stdout, `2` = stderr — simply because stdout is numbered "first" in the convention Bash's authors chose; there's no deeper rationale. Omitting the number entirely (`>`) defaults to stdout.

> **Note:** `echo` does **not** read from standard input — only certain programs (like `cat`, `grep`) are built to consume stdin. Trying to pipe/redirect something into `echo` won't do what you might expect.

### Developer Takeaway / Best Practices

- Separate stdout/stderr redirection (`1>` / `2>`) is the standard pattern for production logging: normal application output goes to a metrics/log stream, while errors go to a separate error log.
- `/dev/null` is your go-to when you explicitly don't care about a stream's output (silencing noisy but harmless command output, e.g. in scripts/cron jobs).
- Remember `>` **overwrites**, `>>` **appends** — this distinction has caused many an accidental data loss.

### Hands-on Practical Exercise

1. Run `echo "hello world" > greeting.txt` and confirm the file's content with `cat`.
2. Run the same `echo` command again but with `>>`, then `cat` the file — confirm the line now appears twice.
3. Run `ls -la /nonexistent-directory 2> errors.log` and confirm the error was captured in the log file instead of printing to your screen.
4. Run a command that produces both output and an error in the same invocation (e.g., `ls -la /home /nonexistent-directory`) and redirect stdout and stderr to two separate files using `1>` and `2>`.
5. Repeat step 4, but this time redirect both streams to a single combined file using `&>` (or `> file 2>&1`).
6. Run `cat greeting.txt > /dev/null` and confirm nothing prints to your terminal.
7. Use `grep` with input redirection (`<`) to search for the word "hello" inside `greeting.txt`.

---

## 6. Pipes: Connecting Programs Together

### Concept Overview

Where redirection (`>`, `<`) connects a program to a **file**, a **pipe** (`|`) connects the stdout of one program directly to the stdin of another. This is the core of the "Unix philosophy" mentioned early in the course: assume the output of any program could become the input to another program. Pipes let you chain small, focused tools together into powerful one-liners.

### Commands & Syntax

|Syntax|Meaning|
|---|---|
|`commandA \| commandB`|Feed the stdout of `commandA` directly into the stdin of `commandB`.|
|`commandA \| commandB \| commandC`|Chain any number of commands — pipes can be stacked indefinitely.|
|`grep "pattern"`|Searches text for a given pattern/string; extremely common pipe target.|
|`ps aux`|Lists all currently running processes on the system.|
|`kill -9 <PID>`|Forcefully terminates a process by its process ID.|
|`yes`|Continuously outputs `y` (or a given string) forever — useful for auto-answering interactive prompts.|
|`command &`|Runs a command in the **background**, freeing up your terminal (and printing a background job/process ID).|

### Code & Script Examples

```bash
# Basic pipe: feed cat's output into grep
cat ls.txt | grep "ls-error.txt"

# Chained pipe combining redirection AND piping
grep "ls-error.txt" < ls.txt > grep.txt 2> /dev/null
# Reads ls.txt into stdin, greps for the string, writes matches to grep.txt,
# and silently discards any errors

# Find a specific running process by name
ps aux | grep "node"

# Notice: grep also matches itself in the process list (it shows up as
# a running process performing the search) — this is expected behavior

# Run a long-lived background process, discarding its output
yes > /dev/null &
# [1] 1224   <- background job's process ID (PID)

ps aux | grep "yes"
# locate the PID, then kill it:
kill -9 1224

# Auto-answer "yes" to every interactive rm confirmation prompt
yes | rm -i file*
# (Contrast with `rm -i file*` alone, which stops and asks for every file,
#  and with `rm -f file*`, which skips prompts entirely without needing `yes`)
```

> **Note:** Programs behave differently when piped vs. run interactively. For example, `ls` and `grep` disable colorized output automatically when their output is being redirected/piped rather than shown directly on a terminal — this "machine-friendly" mode makes downstream parsing easier.

> **Note:** `echo` does not read from stdin, so piping _into_ `echo` (`something | echo`) does not work the way piping into `cat` or `grep` does.

### Developer Takeaway / Best Practices

- `ps aux | grep <processname>` is one of the most common real-world pipe patterns — used constantly to locate a rogue or stuck process before killing it.
- Piping `yes` into an interactive command (like `rm -i`) is a handy trick for auto-confirming prompts, though `-f` is more direct when you fully trust the operation.
- Think of each command in a pipeline as a small, single-purpose tool — this compositional style is central to how Linux tooling is designed.

### Hands-on Practical Exercise

1. Run `ps aux` and pipe it into `grep` to find any process related to `bash`.
2. Start a harmless long-running background process: `sleep 300 &` and note its PID.
3. Use `ps aux | grep sleep` to locate that process in the list.
4. Kill it using `kill -9 <PID>` and confirm with another `ps aux | grep sleep` that it's gone.
5. Create 5 empty files (`touch demo{1..5}.txt`), then run `yes | rm -i demo*.txt` to delete them all without being individually prompted.
6. Combine everything: run `ls -la /etc | grep ".conf" > conf-files.txt 2> /dev/null` and inspect the resulting file.

---

## 7. Users & `sudo`

### Concept Overview

Linux is a **multi-user** operating system by design — "users" aren't only human logins; they can represent background services, scoped automation accounts, or restricted service accounts. The **principle of least power/privilege** underlies why systems use many limited-permission users instead of running everything as `root`: if a low-privilege account is ever compromised (or you make a scripting mistake, like an accidental `rm -rf`), the damage is contained because that account simply doesn't have the permissions to cause wider harm.

### Commands & Syntax

|Command|Purpose|
|---|---|
|`whoami`|Prints the currently logged-in username.|
|`cat /etc/passwd`|Lists all user accounts configured on the system, along with metadata: user ID, group ID, home directory, and login shell.|
|`sudo <command>`|"Super user do" — temporarily elevates **just this one command** to run as `root`, then reverts.|
|`sudo su`|Switches your entire session to the `root` user (stays elevated until you `exit`). **Avoid habitually staying logged in as root.**|
|`exit`|Leaves the current elevated/switched session, returning to the previous user.|
|`su <username>`|Switch users (prompts for that user's password).|
|`sudo useradd <username>`|Creates a new user account (root privilege required).|
|`sudo passwd <username>`|Sets/changes a user's password.|
|`sudo usermod -aG <group> <username>` (or `--append --groups`)|Adds a user to an additional group, without removing them from their existing groups.|

### Code & Script Examples

```bash
# Who am I currently logged in as?
whoami
# -> ubuntu

# List all configured users on the system
cat /etc/passwd
# Each line: username:x:UID:GID:comment:home_directory:login_shell
# e.g. accounts like "nobody" have no home/login shell — they can't log in
# interactively (their shell is set to something like /usr/sbin/nologin or false)

# Temporarily elevate ONE command to root
sudo mkdir /hi
whoami        # -> ubuntu (still normal user)
sudo whoami   # -> root   (only for that single command)

# Fully switch into the root user's session (use sparingly!)
sudo su
whoami   # -> root
exit     # back to your normal user

# Create a brand-new user
sudo useradd brian

# Set that user's password (you'll be prompted to enter/confirm it)
sudo passwd brian

# Switch into that user's session
su brian
whoami   # -> brian

# By default, a freshly created user has NO sudo privileges:
sudo echo hi
# -> "brian is not in the sudoers file. This incident will be reported."

# Grant brian sudo privileges by adding him to the "sudo" group
exit               # back to ubuntu
sudo usermod -aG sudo brian
# or the long-form: sudo usermod --append --groups sudo brian

su brian
sudo whoami   # now succeeds after entering brian's password -> root
```

> **Note:** Attempting `sudo` as an unauthorized user produces the (famous, meme-worthy) message: _"is not in the sudoers file. This incident will be reported."_ In practice, this "report" goes to the local mail system for an admin's mailbox — but the instructor notes he's never actually seen this followed up on in any real job.

> **Warning:** Don't stay logged in as `root`/`sudo su` for extended periods. Running as an unprivileged user by default is a safety net against your _own_ mistakes (e.g., an accidental `rm -rf`) as much as it is protection against attackers.

### Developer Takeaway / Best Practices

- Use `sudo <command>` for one-off privileged operations rather than `sudo su` for an entire session — this limits your blast radius if you fumble a command.
- When setting up service/automation accounts, scope them to the absolute minimum privileges needed for their specific job (principle of least privilege).
- Adding a user to the `sudo` group is the standard way to grant broad admin rights — but should be done deliberately, not by default.

### Hands-on Practical Exercise

1. Run `whoami` and `cat /etc/passwd` to inspect your current user and all configured accounts on the system.
2. Create a new user called `intern` using `sudo useradd intern`.
3. Set a password for `intern` with `sudo passwd intern`.
4. Switch to the `intern` user with `su intern` and confirm with `whoami`.
5. Attempt `sudo whoami` as `intern` and confirm you get the "not in the sudoers file" message.
6. Exit back to your original user, then grant `intern` sudo access via `sudo usermod -aG sudo intern`.
7. Switch back to `intern` and confirm `sudo whoami` now succeeds and returns `root`.

---

## 8. File Permissions: `rwx`, `chmod`, `chown`, and Octal Notation

### Concept Overview

Every file/directory in Linux has an **owning user**, an **owning group**, and a set of **read/write/execute** permissions defined separately for three categories: the **owner (user)**, the **group**, and **everyone else (other)**. This is what you're seeing in the leading column of `ls -la` output (e.g., `-rwxr-xr--`).

### Reading the Permission String

```
-  rwx  rwx  rwx
^   ^    ^    ^
|   |    |    +-- "other" permissions (everyone else)
|   |    +------- "group" permissions
|   +------------ "user"/owner permissions
+---------------- file type: "-" = normal file, "d" = directory
                   (other letters exist for special file types, but
                    normal files and directories cover the vast majority
                    of what you'll encounter)
```

Each `rwx` triplet means:

- `r` — can **read** the file / list a directory's contents.
- `w` — can **write to** the file / create-delete files inside a directory.
- `x` — can **execute** the file (run it as a program/script) / "enter" (`cd` into) a directory.

A `-` in any position means that permission is **absent**, e.g. `rw-` means read+write but not execute.

### Commands & Syntax

|Command|Purpose|
|---|---|
|`ls -la` (or `ls -lsah`)|Shows permissions, owner, and group for files/directories in long-listing format.|
|`chmod u=rwx,g=rw,o=r <file>`|Set exact symbolic permissions per category: user (`u`), group (`g`), other (`o`).|
|`chmod +x <file>`|Add execute permission for **all** categories.|
|`chmod -x <file>`|Remove execute permission for all categories.|
|`chmod +w <file>`|Add write permission (commonly to user + group).|
|`chmod <octal> <file>`|Set permissions using a 3-digit numeric shorthand (see below).|
|`chown <user>:<group> <path>`|Change the **owner** user and group of a file/directory.|
|`sudo chown -R <user>:<group> <dir>`|Recursively change ownership of a directory and everything inside it.|

### Octal (Numeric) Permission Notation

Each digit is a sum of: `4` = read, `2` = write, `1` = execute. The three digits represent **user, group, other**, in that order.

|Value|Meaning|
|---|---|
|`7`|`rwx` (4+2+1)|
|`6`|`rw-` (4+2)|
|`5`|`r-x` (4+1)|
|`4`|`r--` (4)|
|`0`|`---` (none)|

Common combos:

|Octal|Meaning|
|---|---|
|`777`|Everyone can read/write/execute — **very permissive, generally insecure**. Equivalent to `u=rwx,g=rwx,o=rwx`.|
|`700`|Only the owner has any access at all (read/write/execute); group and other have none.|
|`600`|Owner can read/write; no one else has any access. Common for private text files (not executable).|
|`640`|Owner: read/write. Group: read-only. Other: none.|
|`664`|Owner + group: read/write. Other: read-only.|

### Code & Script Examples

```bash
# View permissions
ls -la
# -rwxr-xr--  1 ubuntu ubuntu  ... another-file.txt

# Symbolic form: set exact permissions per category
sudo chmod u=rw,g=rw,o=rw hello.txt
# equivalent numeric form:
sudo chmod 666 hello.txt

# The infamous (and generally discouraged) "everyone can do everything"
sudo chmod 777 file.txt
# equivalent to:
sudo chmod u=rwx,g=rwx,o=rwx file.txt

# Owner-only full access, nobody else has any permission
sudo chmod 700 file.txt

# Owner read/write only (typical for a private non-executable file)
sudo chmod 600 file.txt

# Make a script executable for everyone
touch my-new-program
sudo chmod +x my-new-program
# remove execute permission again
sudo chmod -x my-new-program

# Add execute permission to a DIRECTORY (needed for others to `cd` into it)
sudo chmod +x .    # (or specify the path explicitly, e.g. sudo chmod +x /hello)

# Change ownership of a directory from root:root to a specific user
sudo mkdir /hello
sudo chown ubuntu:ubuntu /hello
# now the "ubuntu" user fully owns /hello and can read/write/execute freely

# Recursive ownership change
sudo chown -R ubuntu:ubuntu /some/directory
```

### Walking Through a Real Scenario From the Transcript

```bash
# Starting point: /root is owned by root:root, and the ubuntu user has no
# write permission there.
whoami                  # ubuntu
mkdir /hi               # PERMISSION DENIED — ubuntu can't write to /root

sudo touch /root/brian.txt
# Works, but now the file is owned by root:root — even the ubuntu user
# can't read or write it afterward (only root can), which is often
# MORE restrictive than intended.

# Better approach: create the directory as root, then hand ownership
# over to the user who actually needs to use it:
sudo mkdir /hello
sudo chown ubuntu:ubuntu /hello
# now ubuntu can freely touch/create files inside /hello

# To let a DIFFERENT user (e.g. "brian") write into a file you own,
# open up permissions on the FILE:
sudo chmod u=rw,g=rw,o=rw /hello/hello.txt
# brian can now edit the file's CONTENTS, but still can't create NEW
# files in the /hello directory unless the DIRECTORY's permissions
# are also opened up (directories require the "x" bit to be entered/traversed):
sudo chmod +x /hello
# NOW brian can `touch /hello/brian.txt` and it will succeed —
# and the new file will be owned by brian:brian, even though the
# parent directory is owned by ubuntu.
```

> **Note:** For a **directory**, the `x` (execute) permission controls whether a user is allowed to `cd` into it / traverse it at all — it's not about "running" the directory. Without `x` on a directory, a user cannot create files inside it even if they otherwise have write permission on the directory.

> **Warning:** `chmod 777` is common in Stack Overflow answers because it's the "path of least resistance," but it's a poor security practice — it grants full read/write/execute to literally anyone on the system. Prefer the least-permissive setting that still gets the job done (e.g., `700`, `600`, `750`).

### Developer Takeaway / Best Practices

- In real-world use, the most common `chmod` operations are: making a script executable (`chmod +x script.sh`) and tightening/loosening access for the **user** category specifically — modifying **group** permissions is comparatively rare.
- `chown` is most often used to hand ownership of a root-created directory/file over to the actual application/service user that needs to work with it.
- Remember: a directory needs the `x` bit for a user to enter/traverse it, separate from the `r`/`w` bits controlling listing/creating files.
- Avoid `777` in production; prefer scoping permissions as tightly as the use case allows.

### Hands-on Practical Exercise

1. Create a file `secret.txt` and inspect its default permissions with `ls -la`.
2. Use symbolic `chmod` to set it to `u=rw,g=,o=` (owner read/write only, nobody else has any access). Verify with `ls -la`.
3. Convert that same permission scheme to octal notation and apply it with `chmod 600 secret.txt` — confirm the result is identical.
4. Create a script file `hello.sh` containing `echo "hello"`, then make it executable using `chmod +x hello.sh`, and run it with `./hello.sh`.
5. As `root` (via `sudo`), create a directory `/shared` and a file inside it. Then use `chown` to hand ownership of `/shared` over to your normal (non-root) user.
6. Create a second user (e.g. `su brian` from the previous section's exercise, or a new one) and, without giving them ownership, use `chmod` to grant that user's group **write** access to a specific file inside `/shared`. Confirm they can edit it but still can't create a _new_ file in `/shared` until you also grant `+x` on the directory itself.
7. Finally, grant the directory `+x` and confirm the second user can now create new files inside `/shared`.

---

## Capstone Challenge: Putting It All Together

This final exercise combines file management, wildcards, redirection, pipes, users, and permissions into one realistic scenario.

1. As your normal user, create a project structure: `mkdir -p webapp/{src,logs,backup}`.
2. Inside `webapp/src`, use brace expansion to create `module{1..5}.js`.
3. Make `module1.js` a "script" by adding `chmod +x` to it.
4. Simulate an application log by running a background process that continuously appends timestamps: `while true; do date >> webapp/logs/app.log; sleep 1; done &`
5. Let it run for a few seconds, then use `tail -f webapp/logs/app.log` briefly to watch it live, and exit with `Ctrl+C`.
6. Kill the background loop: find its PID with `ps aux | grep "while true"`, then `kill -9 <PID>`.
7. Archive the whole `webapp` directory into a compressed tarball: `tar -czf webapp-backup.tar.gz webapp`.
8. Extract that tarball into `webapp/backup` (which must already exist) using `tar -xzf webapp-backup.tar.gz -C webapp/backup`.
9. Create a new restricted user `deploy` (`sudo useradd deploy`, `sudo passwd deploy`).
10. `chown` the `webapp` directory to `deploy:deploy` so that user fully owns it, then confirm (as `deploy`) that they can create a new file inside `webapp/src` but that a **different**, non-owning user cannot — unless you explicitly loosen permissions via `chmod`.
11. Finally, redirect a full recursive listing of the `webapp` directory to a report file, sending any errors to a separate error log: `ls -laR webapp 1> webapp-report.txt 2> webapp-errors.txt`.