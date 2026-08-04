
---

# Lesson 1: Package Management with APT

- **Key Takeaway:** Every Linux distro has its own package manager (Ubuntu/Debian: APT/dpkg, RHEL: RPM/yum, Alpine: APK). `apt` is the modern, user-friendly front-end; `apt-get` is the older, still-supported tool.
    
- **Essential Commands & Examples:**
    

```bash
sudo apt install <pkg>          # install a package (needs sudo)
sudo !!                          # rerun last command with sudo prepended
apt search <term>                # search available packages
apt show <pkg>                   # show package details (maintainer, version, priority)
sudo apt install aptitude        # nice TUI front-end for browsing/managing packages
sudo apt autoremove              # clean up orphaned/unneeded packages
```

- **Quick Practice Challenge:**
    1. Run `sudo apt update` then `apt search htop`.
    2. Install it: `sudo apt install htop`.
    3. Inspect it with `apt show htop`.
    4. Clean up unused deps with `sudo apt autoremove`.

---

# Lesson 2: APT Internals & Cross-Platform Package Managers

- **Key Takeaway:** `apt` is a wrapper around lower-level tools (`apt-get`, `apt search`, `apt cache`); `update` refreshes the package list/registry, `upgrade` actually updates installed packages — always `update` before `upgrade`.
    
- **Essential Commands & Examples:**
    

```bash
sudo apt update                  # refresh list of available packages (registry)
sudo apt upgrade [pkg]           # upgrade installed package(s) to latest available
sudo apt full-upgrade            # combines autoremove + upgrade in one go
```

- macOS: **Homebrew** (`brew info <pkg>`) — community-run, not official.
- Windows: **winget** (new, official MS) or Chocolatey (older, third-party).

- **Quick Practice Challenge:**
    1. Run `sudo apt update` and note how many packages are upgradeable (`apt list --upgradable`).
    2. Upgrade a single package: `sudo apt upgrade <pkg>`.
    3. Run `sudo apt full-upgrade` to catch everything at once.
    4. Compare: explain in one sentence the difference between `update` and `upgrade`.

---

# Lesson 3: Snaps vs Debs

- **Key Takeaway:** Snaps are Canonical's sandboxed, cross-distro package format (self-contained, auto-updating, larger, no manual-review bottleneck); Debs (via APT) are smaller, must be fully reviewed before publishing, and don't auto-update.
    
- **Essential Commands & Examples:**
    

```bash
sudo snap install <pkg>                 # install from snap store
sudo snap install node --channel=14/stable --classic
                                          # pin a channel; --classic breaks sandbox (trust needed)
snap info <pkg>                          # show available channels/versions
sudo apt remove <pkg>                    # remove the Deb version (snap install is separate)
```

- Snaps: cross-distro, auto-update (no opt-out), more secure sandboxing, bigger size (bundles deps).
- Debs/APT: Ubuntu-only, manually reviewed, smaller, no auto-update, install GUI apps generally via snap instead (VS Code, Spotify, Chrome, Firefox).
- `snapd` = the daemon required to run snaps (install manually on Mint/RHEL).

- **Quick Practice Challenge:**
    1. Install a tool via snap: `sudo snap install <pkg>`.
    2. Check its info/channels: `snap info <pkg>`.
    3. Remove the Deb version of a package you have (if any) and reinstall via snap for comparison.
    4. Explain in one sentence why `--classic` reduces security.

---

# Lesson 4: Writing Your First Bash Script

- **Key Takeaway:** Bash scripts bundle repeated command-line tasks into one runnable file; `source`/`.` run the script in the _current_ shell process (state persists), while `bash script.sh` runs it in a _subprocess_ (state like `cd` is discarded after exit).
    
- **Essential Commands & Examples:**
    

```bash
mkdir -p ~/temp        # -p: don't error if dir already exists
cd ~/temp
touch file{1..5}.txt   # brace-expansion range
echo done
```

```bash
source gen_files.sh    # run in current shell (cd persists)
. gen_files.sh          # same as source (portable form)
bash gen_files.sh       # run in subprocess (cd does NOT persist)
```

- **Quick Practice Challenge:**
    1. Write `make_dirs.sh` that `mkdir -p`s a folder, `cd`s into it, and creates 3 files.
    2. Run it with `bash make_dirs.sh`, then `pwd` — confirm you're back where you started.
    3. Run it with `source make_dirs.sh`, then `pwd` — confirm you're now inside the new folder.
    4. Explain why the two behave differently.

---

# Lesson 5: Shebang (`#!`) Lines

- **Key Takeaway:** The shebang (`#!/bin/bash` or path to any interpreter) on line 1 of a script tells the OS which program to use to execute it, letting you run the file directly (`./script`) without prefixing `bash`, `source`, or naming the interpreter.
    
- **Essential Commands & Examples:**
    

```bash
#!/bin/bash             # first line of a bash script
chmod +x myscript        # make it executable
./myscript                # run directly — OS reads shebang to pick interpreter
```

```bash
#!/snap/bin/node          # shebang can point to ANY interpreter (needs full path)
console.log("hi from node")
```

- **Quick Practice Challenge:**
    1. Create a script with `#!/bin/bash` as line 1, `chmod +x` it, and run via `./scriptname`.
    2. Find node's full path with `which node`.
    3. Write a file with `#!/path/to/node` shebang and a `console.log(...)` line; `chmod +x` and run it directly.
    4. Remove the shebang line and observe the failure when running `./scriptname`.

---

# Lesson 6: `$PATH`, Custom Bins, Comments & Variables

- **Key Takeaway:** `$PATH` is a colon-separated list of directories Bash searches (left-to-right) for executables; adding your own script folder to `$PATH` lets you run your scripts as normal commands (with tab-completion). Bash vars are declared `NAME=value` (no spaces) and read with `${NAME}`.
    
- **Essential Commands & Examples:**
    

```bash
echo $PATH                                  # view current search path
mkdir ~/my_bin && mv myscript ~/my_bin/     # put custom scripts in one place
# In ~/.bashrc:
export PATH=~/my_bin:$PATH                  # prepend (don't overwrite!) — then re-source
source ~/.bashrc
myscript                                     # now runnable from anywhere
```

```bash
# comment example
DESTINATION=~/temp        # var assignment, no spaces around =
echo "${DESTINATION}"     # curly braces required when concatenating (e.g. ${VAR}_suffix)
```

- **Quick Practice Challenge:**
    1. Make `~/my_bin`, move an executable script into it.
    2. Add `export PATH=~/my_bin:$PATH` to `~/.bashrc`, then `source ~/.bashrc`.
    3. Run the script by name from any directory (no `./`).
    4. Add a `DESTINATION` variable to the script and reference it with `${DESTINATION}`.

---

# Lesson 7: Script Arguments & Interactive Input

- **Key Takeaway:** `$1`, `$2`, … are positional arguments passed to a script (`$0` is the script name itself); `read -p "prompt: " VAR` pauses execution and stores user input into `VAR`.
    
- **Essential Commands & Examples:**
    

```bash
DESTINATION=$1                       # first CLI argument
echo "$0 was called"                 # script's own invocation name

read -p "Enter a file prefix: " FILE_PREFIX   # interactive prompt -> stores into FILE_PREFIX
```

```bash
./gen_files.sh some-folder           # $1 = "some-folder"
```

- **Quick Practice Challenge:**
    1. Modify a script to accept a folder name as `$1` instead of hardcoding it.
    2. Run it: `./script.sh my-folder` and confirm it uses that path.
    3. Add a `read -p` prompt asking for a filename prefix.
    4. Run without arguments and observe what `$1` evaluates to (empty).

---

# Lesson 8: Conditionals (`if`/`test`) in Bash

- **Key Takeaway:** `if [ condition ]; then ... fi` is syntax sugar for the `test` command; exit status `$?` (0 = true, 1 = false) drives the branch. Common test flags: `-z` (empty string), `=`/`-eq` (equal), `-gt`/`-lt`/`-ge`/`-le` (numeric compare), `-e` (file exists), `-w` (writable).
    
- **Essential Commands & Examples:**
    

```bash
if [ -z "$DESTINATION" ]; then
  echo "No path provided, defaulting to ~/temp"
  DESTINATION=~/temp
fi                                    # 'fi' = if backwards, always closes an if-block
```

```bash
test 15 = 15; echo $?      # 0 = true
test 15 -gt 10             # numeric greater-than
test Brian = Mark          # string comparison (false)
test -e output.txt         # file exists?
test -w output.txt         # file exists AND writable by current user?
man test                   # full reference of test flags
```

- **Quick Practice Challenge:**
    1. Write a script with `if [ -z "$1" ]; then echo "arg required"; fi`.
    2. Run it with and without an argument to confirm both branches.
    3. Add a numeric check: `if [ $1 -gt 10 ]; then echo "big"; fi`.
    4. Test file existence with `[ -e somefile ]` and `echo $?`.

---

# Lesson 9: Bash Arrays, For-Loops & While-Loops

- **Key Takeaway:** Bash supports arrays (space-separated, clumsy string-splitting under the hood), `for...in` loops, and `while` loops driven by `test`-style conditions; `read -p` pauses execution to collect user input.
    
- **Essential Commands & Examples:**
    

```bash
friends=(Kyle Mark Jim "Brian Holt" Sarah)   # declare array (quote multi-word items)
echo "My second friend is ${friends[1]}"     # index access (0-based)
echo "I have ${#friends[*]} friends"         # array length

for friend in ${friends[*]}; do              # for-in loop
  echo "$friend"
done

NUM_TO_GUESS=$(( $RANDOM % 10 + 1 ))         # $(( )) invokes `let` for arithmetic
GUESSED_NUM=0
while [ $NUM_TO_GUESS != $GUESSED_NUM ]; do
  read -p "your guess: " GUESSED_NUM         # no $ before var name being SET by read
done
echo "you got it"
```

- **Quick Practice Challenge:**
    1. Create `friends.sh`, declare an array of 4 names, print the 3rd one.
    2. Loop over the array with a `for...in` loop, echoing each name.
    3. Write a `while` loop that asks "guess a number 1-5" until correct.
    4. Run it and verify `$?` is 0 after a correct guess.

---


# Lesson 10: `if`/`elif`/`else` and `case` Statements; Bash Scoping

- **Key Takeaway:** Bash supports `if/elif/else/fi` chains and `case/esac` switch-style matching (each case ends in `;;`, wildcard `*)` is the default); variables are local to a script unless explicitly `export`ed to the global/environment scope.
    
- **Essential Commands & Examples:**
    

```bash
#!/bin/bash
if [ $1 -gt 10 ]; then
  echo "greater than 10"
elif [ $1 -lt 10 ]; then
  echo "less than 10"
else
  echo "equals 10"
fi
```

```bash
#!/bin/bash
case $1 in
  smile) echo ":)" ;;
  sad)   echo ":(" ;;
  laugh) echo "haha" ;;
  *)     echo "I don't know that one yet" ;;
esac                       # 'esac' = case backwards
```

- Local vs global scope: variables set in a script stay local to that script's process unless you `export VAR`.

- **Quick Practice Challenge:**
    1. Write `check_num.sh` using `if/elif/else` to classify `$1` as greater/less/equal to 10.
    2. Test it with 3 different inputs.
    3. Write `mood.sh` using a `case` statement matching `happy`, `sad`, and a wildcard default.
    4. Run `mood.sh` with an unmatched input and confirm the wildcard `*)` branch fires.

