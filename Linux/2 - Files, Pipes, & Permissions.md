# Reading Files (`less`, `more`, `man`, `cat`, `head`, `tail`)

- **Key Takeaway:** Use `less` for scrolling through long files, `cat` for short files, and `head`/`tail` to grab just the start/end — `tail -f` is the standard way to watch live logs.
    
- **Essential Commands & Examples:**
    

```bash
less file.txt        # scrollable pager (q=quit, /term=search) — best for long files
more file.txt        # older, weaker version of less; prefer less
man less              # full manual (uses less internally); not all programs have one
less --help           # quick flag summary, faster than man
cat file.txt          # dump entire file to screen — good for short files
head file.txt         # first 10 lines
head -n 50 file.txt    # first 50 lines
tail file.txt          # last 10 lines
tail -n 3 file.txt      # last 3 lines
tail -f file.txt        # follow file live (Ctrl+C to exit) — "tailing the logs"
```

- **Quick Practice Challenge:**

1. Create a 50-line file and view it with `less`, searching for a word with `/`.
2. Print just the first 5 and last 5 lines using `head`/`tail`.
3. Run `tail -f` on the file, then in another terminal `echo "new" >> file.txt` and watch it appear live.

---

# File & Directory Management (`mkdir`, `touch`, `rm`, `cp`, `mv`)

- **Key Takeaway:** `touch` creates/updates files, `rm -rf` deletes permanently (no recycle bin — be careful), and `cp`/`mv` copy/rename files and directories.
    
- **Essential Commands & Examples:**
    

```bash
mkdir new-folder                 # create a directory
mkdir -p a/b/c                   # create nested dirs in one shot
touch file.txt                   # create file, or update its modified time if it exists
rm file.txt                      # delete a file
rm -r some-dir                   # recursive delete (required for directories)
rm -rf some-dir                  # recursive + force (skip confirmation) — DANGEROUS, permanent
rm -i file*                      # interactive delete, confirm each file
cp src.txt dest.txt               # copy a file
cp -R src-dir dest-dir             # recursively copy a directory
mv old-name new-name               # move OR rename
```

> **Warning:** `rm -rf` is permanent — no undo, no recycle bin. Never run `rm -rf *` or `rm -rf /` carelessly.

- **Quick Practice Challenge:**

1. `mkdir -p demo/sub` then `touch demo/sub/file.txt`.
2. Copy `demo` to `demo-copy` with `cp -R`.
3. Rename `demo-copy` to `demo-archive` with `mv`.
4. Delete `demo-archive` with `rm -r`.

---

# Archiving with `tar`

- **Key Takeaway:** `tar` bundles files like a zip; add `z` to compress with gzip. Destination folder for extraction must already exist.
    
- **Essential Commands & Examples:**
    

```bash
tar -cf archive.tar file1 dir1        # create uncompressed archive
tar -czf archive.tar.gz file1 dir1    # create compressed (gzip) archive
tar -xzf archive.tar.gz -C dest/       # extract into existing dest folder
```

- **Quick Practice Challenge:**

1. Create a folder with 3 dummy files.
2. Archive it compressed: `tar -czf backup.tar.gz folder/`.
3. `mkdir restore` then extract into it with `tar -xzf backup.tar.gz -C restore`.

---

# Wildcards, Brace Expansion & Escaping

- **Key Takeaway:** Bash expands `{}`, `*`, and `?` _before_ the command runs — this is a shell feature, not a program feature.
    
- **Essential Commands & Examples:**
    

```bash
touch file{1,2,3}.txt      # brace expansion -> file1.txt file2.txt file3.txt
touch file{1..30}.txt      # numeric range 1-30
echo {a..z}                 # alphabetic range (also reversible: {z..a})
echo {1..100..10}           # range with step
echo {a..c}{1..3}           # permutations: a1 a2 a3 b1 b2 b3 c1 c2 c3
ls file*.txt                 # * = zero or more of any char
ls file??.txt                 # ? = exactly one char
touch a\ file                  # \ escapes a literal space
touch file\\                    # \\ = literal backslash
```

- **Quick Practice Challenge:**

1. Create `report1.txt`–`report20.txt` in one command with brace expansion.
2. List only files ending in `.txt` with a wildcard.
3. Create a file with a literal space in its name using `\`.
4. Clean up everything with `rm report*.txt`.

---

# Streams & Redirection (stdout, stderr, stdin, `/dev/null`)

- **Key Takeaway:** `1` = stdout, `2` = stderr — redirect them separately with `>`/`2>`, or discard unwanted output to `/dev/null`.
    
- **Essential Commands & Examples:**
    

```bash
echo "hi" > file.txt          # stdout to file, OVERWRITE
echo "hi" >> file.txt         # stdout to file, APPEND
cmd 2> errors.txt              # stderr only, overwrite
cmd 2>> errors.txt              # stderr only, append
cmd > out.txt 2> err.txt         # stdout and stderr to separate files
cmd > all.txt 2>&1                # stdout AND stderr to same file
cmd 2> /dev/null                   # discard errors, keep normal output
cmd > /dev/null                     # discard normal output, keep errors
grep "term" < file.txt                # feed file into stdin of grep
```

> **Note:** `echo` doesn't read from stdin — piping/redirecting into it won't work like `cat`/`grep`.

- **Quick Practice Challenge:**

1. `echo "hello" > greet.txt` then run it again with `>>` — confirm the line duplicates.
2. Run `ls /fake-dir 2> errors.log` and confirm the error lands in the file, not the screen.
3. Run a command with both output and an error, splitting them into two separate files with `1>`/`2>`.

---

# Pipes (Connecting Programs)

- **Key Takeaway:** `|` feeds one program's stdout into another's stdin — chain small tools together (classic pattern: `ps aux | grep <name>`).
    
- **Essential Commands & Examples:**
    

```bash
cat file.txt | grep "term"        # pipe cat's output into grep
ps aux | grep node                 # find a running process by name
kill -9 <PID>                       # force-kill a process by ID
sleep 300 &                          # run in background, get a PID back
yes | rm -i file*                     # auto-answer "yes" to every prompt
cmdA | cmdB | cmdC                     # chain as many pipes as needed
```

> **Note:** Piped output often loses color/formatting — programs detect non-terminal output and switch to "machine-friendly" mode.

- **Quick Practice Challenge:**

1. Start a background process: `sleep 300 &`.
2. Find it with `ps aux | grep sleep`.
3. Kill it with `kill -9 <PID>` and confirm with `ps aux | grep sleep` again.

---

# Users & `sudo`

- **Key Takeaway:** Linux is multi-user; use `sudo` for one-off root privilege instead of staying logged in as root (principle of least privilege).
    
- **Essential Commands & Examples:**
    

```bash
whoami                          # current user
cat /etc/passwd                  # list all system users
sudo <command>                    # run ONE command as root, then revert
sudo su                            # switch entire session to root (exit to leave)
su <username>                       # switch to another user
sudo useradd brian                   # create new user
sudo passwd brian                     # set that user's password
sudo usermod -aG sudo brian            # grant sudo privileges (add to sudo group)
```

> **Note:** Unauthorized `sudo` attempts show: _"is not in the sudoers file. This incident will be reported."_

- **Quick Practice Challenge:**

1. Create a user: `sudo useradd intern` + `sudo passwd intern`.
2. `su intern` and confirm `sudo whoami` fails.
3. As your original user, run `sudo usermod -aG sudo intern`.
4. Switch back to `intern` and confirm `sudo whoami` now works.

---

# File Permissions (`rwx`, `chmod`, `chown`, Octal)

- **Key Takeaway:** Permissions are `user/group/other` × `read/write/execute`; use symbolic (`u=rwx`) or octal (`755`) notation with `chmod`, and `chown` to change ownership.
    
- **Essential Commands & Examples:**
    

```bash
ls -la                              # view permissions: -rwxr-xr-- user group
chmod u=rw,g=r,o= file.txt           # symbolic: exact perms per category
chmod +x script.sh                    # add execute for everyone
chmod -x script.sh                     # remove execute
chmod 777 file.txt                      # rwxrwxrwx — everyone, all perms (avoid: insecure)
chmod 700 file.txt                       # owner-only, full access
chmod 600 file.txt                        # owner read/write only, no execute
chown user:group path                      # change ownership
sudo chown -R user:group dir/                # recursive ownership change
```

> **Reference:** Octal math: read=4, write=2, execute=1 (e.g., `rwx`=7, `rw-`=6, `r-x`=5, `r--`=4). **Note:** A directory needs `+x` for a user to `cd` into / create files inside it, separate from `r`/`w`.

- **Quick Practice Challenge:**

1. Create `secret.txt` and set it to owner-only access: `chmod 600 secret.txt`.
2. Create `hello.sh` with `echo hello`, then `chmod +x hello.sh` and run `./hello.sh`.
3. As root, `mkdir /shared` then `chown` it to your normal user.
4. Confirm you can now create files inside `/shared` without `sudo`.