# Lesson 1: Shells, Emulators & Basic Navigation

- **Key Takeaway:** A terminal _emulator_ (Terminal.app, iTerm, Windows Terminal) hosts a _shell_ (bash, zsh); bash is a REPL you also use to navigate the filesystem via `pwd`/`cd`.
    
- **Essential Commands & Examples:**
    

```bash
pwd              # print current absolute path
cd /home/ubuntu  # absolute path navigation
cd ..            # up one directory (relative)
cd ../..         # up two directories
cd home          # relative path (no leading slash)
pwd --help       # --help works on almost every command
```

- **Quick Practice Challenge:**
    1. Run `pwd` to see your current location.
    2. `cd /` to go to root, then `cd` back to your home dir.
    3. Move up two levels using only `../..`.
    4. Confirm final location with `pwd`.

---

# Lesson 2: `ls`, `echo`, `which` & Arguments

- **Key Takeaway:** Arguments are extra text fed to a command (e.g., a path or string) to control its behavior — most commands accept zero, one, or many.
    
- **Essential Commands & Examples:**
    

```bash
ls               # list current directory
ls /bin          # list a specific path (argument)
echo "hi there"  # print text (quote multi-word strings)
which ls         # show path to a command's binary -> /bin/ls
```

- **Quick Practice Challenge:**
    1. `ls` your home directory.
    2. `ls /bin` without `cd`-ing there.
    3. `echo` a sentence with quotes.
    4. `which echo` and `which pwd` to find their binaries.

---

# Lesson 3: Flags — Short vs Long Form

- **Key Takeaway:** `-x` (single dash) = short flags, combinable (`-la`); `--xxxx` (double dash) = one long flag; flags are case-sensitive and behavior varies per program.
    
- **Essential Commands & Examples:**
    

```bash
ls -l                    # long format (owner, size, date, perms)
ls -a                    # show hidden (dotfile) entries
ls -la                   # combined short flags
ls --all                 # long-form equivalent of -a
ls -lsah                 # common combo: long+size+all+human-readable
touch .hidden            # create a hidden file
ls --ignore=snap         # exclude a match (long flag w/ value)
ls --ignore=snap --ignore=home  # repeat flag to ignore multiple
ls -I snap                # short-form equivalent of --ignore (case-sensitive!)
```

- **Quick Practice Challenge:**
    1. `touch .secret` then confirm it's hidden with plain `ls`.
    2. Reveal it with `ls -a`.
    3. Combine flags: `ls -lah`.
    4. Use `--ignore=` to exclude a file from a listing.

---

# Lesson 4: Navigation Shortcuts, History & Tab Completion

- **Key Takeaway:** `~` jumps home from anywhere, `Tab` auto-completes paths/programs, and `Ctrl+R` searches your command history — all core speed tools.
    
- **Essential Commands & Examples:**
    

```bash
cd ~          # jump to home directory
cd ~/snap     # relative path off home
cd /          # jump to filesystem root
# Up/Down arrows -> cycle through command history
# Tab           -> autocomplete path/program; Tab Tab -> list options
# Ctrl+R        -> reverse search history (type to filter, Enter to run)
```

> **Warning:** Anything typed (including plaintext passwords) is saved to bash history — avoid typing secrets directly on the command line.

- **Quick Practice Challenge:**
    1. From `/`, use `cd ~` to return home.
    2. Create a nested folder, `cd` into it using `Tab` completion.
    3. Run a few `ls` variations, then recall one with `Ctrl+R`.
    4. Try tab-completing a partial program name (e.g., `pyt<Tab>`).

---

# Lesson 5: Bash History Internals, `!!`, Clear & Safe Copy/Paste

- **Key Takeaway:** History lives in `~/.bash_history` (flushed on logout, not live); `!!` reruns your last command (classic use: `sudo !!`); be cautious pasting commands from untrusted web sources.
    
- **Essential Commands & Examples:**
    

```bash
tail ~/.bash_history   # view recent history entries
ls -a
!!                      # re-run the exact last command
sudo !!                 # re-run last command with sudo (permission-fix pattern)
clear                   # clear screen (or Ctrl+L)
# macOS copy/paste: Cmd+C / Cmd+V
# Windows terminal copy/paste: Ctrl+Shift+C / Ctrl+Shift+V (plain Ctrl+C = signal!)
```

> **Warning:** Malicious sites can hijack your clipboard and even auto-inject a newline so a pasted command runs instantly — paste untrusted commands into a text editor first to inspect them.

- **Quick Practice Challenge:**
    1. Run any command, then repeat it instantly with `!!`.
    2. Check `tail ~/.bash_history` for recent entries.
    3. Clear your screen with `Ctrl+L`.
    4. Practice OS-correct copy/paste of a short trusted snippet.

---

# Lesson 6: Command-Line Editing Shortcuts (`Ctrl` Power)

- **Key Takeaway:** These `Ctrl` shortcuts let you edit/navigate the current line without a mouse (mouse text-select doesn't reliably work in terminals).
    
- **Essential Commands & Examples:**
    

```bash
# Ctrl+A -> jump to start of line
# Ctrl+E -> jump to end of line
# Ctrl+K -> delete (yank) from cursor to end of line
# Ctrl+U -> delete (yank) from cursor to start of line
# Ctrl+Y -> paste last yanked text (separate from OS clipboard)
```

- **Quick Practice Challenge:**
    1. Type a long fake command, don't hit Enter.
    2. `Ctrl+A` to jump to start, `Ctrl+E` to jump to end.
    3. `Ctrl+K` to delete to end of line, then `Ctrl+Y` to restore it.
    4. `Ctrl+U` to delete to start of line, then `Ctrl+Y` to restore it.

---

# Lesson 7 Signals — `SIGINT`, `SIGQUIT`, `SIGTERM`, `SIGKILL`

- **Key Takeaway:** Signals are requests, not commands — a program chooses how to respond; escalate `Ctrl+C` → `Ctrl+D` → `kill -9` when a process won't stop.
    
- **Essential Commands & Examples:**
    

```bash
tail -f ~/.bash_history   # runs forever; Ctrl+C sends SIGINT to stop it
yes > /dev/null &          # background process; note the PID printed
kill -l                    # list all signal names/numbers
kill -9 <PID>               # SIGKILL — force-kill, no cleanup
kill -SIGKILL <PID>         # same as above, named form
```

|Signal|Trigger|Meaning|
|---|---|---|
|SIGINT|`Ctrl+C`|Polite "please stop"|
|SIGQUIT|`Ctrl+D`|Stronger "shut down now" (e.g., exits Python REPL)|
|SIGTERM|OS shutdown|"Wrap up, we're shutting down"|
|SIGKILL|`kill -9`|Immediate force-kill, no cleanup allowed|

> **Warning:** `kill -9` gives zero chance for the program to save data or clean up — use only as a last resort.

- **Quick Practice Challenge:**
    1. Run `tail -f` on any file, stop it with `Ctrl+C`.
    2. Start `yes > /dev/null &`, note the PID.
    3. Look up SIGKILL's number via `kill -l`.
    4. Kill the background process with `kill -9 <PID>`.

---

# Lesson 8: The `nano` Text Editor

- **Key Takeaway:** `nano` is tiny, ubiquitous (descended from Pico/Pine), and shows shortcuts on-screen — good for quick edits, not real development.
    
- **Essential Commands & Examples:**
    

```bash
nano file.txt   # open/create file
# Ctrl+O -> Write Out (save), then Enter to confirm filename
# Ctrl+X -> Exit (prompts to save if modified)
# Ctrl+G -> Get Help
```

- **Quick Practice Challenge:**
    1. `nano practice.txt`, type a couple lines.
    2. Save with `Ctrl+O`.
    3. Add another line, then exit+save with `Ctrl+X`.
    4. Reopen the file to confirm changes persisted.

---

# Lesson 9: `vim` — How to Quit (Survival Basics)

- **Key Takeaway:** `vim` is modal — you must know `:q`/`:wq`/`:q!` or you can get stuck; `Ctrl+C`/`Ctrl+D` do **not** work to quit it.
    
- **Essential Commands & Examples:**
    

```bash
vim file.txt   # open/create file
i              # enter insert mode (start typing)
<Esc>          # back to command mode
:w             # save
:q             # quit (fails if unsaved changes)
:wq            # save + quit
:q!            # force quit, discard changes
:qa!           # nuclear option — quit everything, no matter what
vimtutor       # interactive vim tutorial
```

- **Quick Practice Challenge:**
    1. `vim practice.txt`, press `i`, type a line, `<Esc>`.
    2. Save without quitting (`:w`).
    3. Add another line, then save+quit in one step (`:wq`).
    4. Reopen, edit again, discard changes with `:q!`.

---

# Lesson 10: History of `vim` (`ed` → `ex` → `vi` → `vim`)

- **Key Takeaway:** `vim`'s modal design traces back to `ed` (1969, Bell Labs), a line-oriented editor built for memory-constrained machines; `vi` (Bill Joy) added a screen-oriented mode, later cloned/improved into `vim`.
    
- **Essential Commands & Examples:**
    

```bash
ed file.txt      # ancient line editor (rarely used today)
ex file.txt      # line-editor descendant of ed
:visual          # inside ex, switches to full-screen (vim-like) mode
vi file.txt      # on modern systems, usually just launches vim
```

Lineage: `qed` → `ed` (Ken Thompson, 1969) → `ex` → `vi` (Bill Joy, visual mode) → `vim` ("Vi IMproved") → Neovim.

- **Quick Practice Challenge:**
    1. Try `ex practice.txt` and observe the line-mode prompt.
    2. Type `:visual` to drop into full-screen mode.
    3. Save and quit using `:wq`.
    4. Reopen with `vim practice.txt` and confirm it's the same editor.