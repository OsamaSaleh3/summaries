# Lesson 1: Environment Variables & Shell Config Files

- **Key Takeaway:** Bash sessions carry environment variables (`printenv`) that can be referenced with `$VAR`; set them per-session with `VAR=value`, permanently per-user in `~/.bashrc`, or system-wide in `/etc/environment` (avoid — affects all users).
    
- **Essential Commands & Examples:**
    

```bash
printenv                     # list all current environment variables
echo $USER                   # use a variable (bash expands $VAR, not echo)
USER=Brian                   # set variable for current session only
GREETING=Hello                # session-only var (no spaces = no quotes needed)
echo "$GREETING $USER"

sudo vi /etc/environment     # SYSTEM-WIDE vars (shared by ALL users) - avoid unless intentional
vi ~/.bashrc                 # PER-USER persistent vars/config (recommended)
export ANOTHER_VAR="something cool"   # add to ~/.bashrc, 'export' makes it available to subprocesses
source ~/.bashrc             # reload .bashrc without logging out

# .bash_profile only runs on first interactive login (rarely needed).
# Standard pattern inside .bash_profile:
#   if -f ~/.bashrc; then source ~/.bashrc; fi
# → Always put customizations in .bashrc, not .bash_profile.
```

- **Quick Practice Challenge:**
    1. Run `printenv` and find your `$HOME` value.
    2. Set a temporary var: `FOO=bar` then `echo $FOO`.
    3. Add `export PERM_VAR="hello"` to `~/.bashrc`, save, then `source ~/.bashrc` and confirm with `echo $PERM_VAR`.
    4. Open a new terminal (without sourcing) and confirm the variable persists automatically.

---

# Lesson 2: Processes, Jobs, Background/Foreground

- **Key Takeaway:** Every running program is a process (`ps aux` lists all of them); use `Ctrl+Z`, `bg`, `fg`, and `jobs` to manage long-running commands without blocking your shell.
    
- **Essential Commands & Examples:**
    

```bash
ps                    # processes for current user only
ps aux                # ALL processes, all users (most commonly used)
ps aux | grep name    # find a specific process
ps aux | less         # scroll through the list

sleep 100 &           # run in background immediately (&)
kill -9 <PID>         # force-kill a process by PID (SIGKILL)

# Foreground job running too long?
Ctrl+Z                # suspend (stop) it, return to shell
jobs                  # list stopped/background jobs
jobs -l                # list jobs WITH their PIDs
bg 1                  # resume job #1 in the background
fg 1                  # bring job #1 back to foreground
```

- `screen`/`tmux` exist for multiple shells/split panes but aren't required — `jobs`/`bg`/`fg` cover most day-to-day needs.
    
- Redirect output when backgrounding, or output still prints to your interactive terminal: `long_command > out.txt &`
    
- **Quick Practice Challenge:**
    
    1. Run `sleep 300 &` and confirm it with `jobs -l`.
    2. Run `sleep 60`, then press `Ctrl+Z` to suspend it, then `bg` to resume in background.
    3. Bring it back with `fg`.
    4. Kill the remaining background sleep job using `kill -9 <PID>`.

---

# Lesson 3: Exit Codes & Command Operators

- **Key Takeaway:** Every command returns an exit code — `0` = success, anything else = failure (`echo $?` shows it). Use `&&` (AND), `||` (OR), and `;` (always run) to chain commands based on success/failure.
    
- **Essential Commands & Examples:**
    

```bash
date; echo $?          # $? shows exit code of last command (0 = success)
true; echo $?          # always exits 0
false; echo $?         # always exits 1 (nonzero)

cmd1 && cmd2            # run cmd2 ONLY if cmd1 succeeded (exit 0)
touch f.txt && date >> f.txt && uptime >> f.txt   # chain multiple steps

cmd1 || cmd2             # run cmd2 ONLY if cmd1 FAILED (nonzero exit)

cmd1 ; cmd2              # run both regardless of success/failure
```

- Common exit code meanings: `0`=success, `1`=general error, `2`=bash misuse, `126`=file not executable, `130`=Ctrl+C interrupt.
    
- **Quick Practice Challenge:**
    
    1. Run `ls /fakepath` then `echo $?` — note the nonzero code.
    2. Run `mkdir testdir && cd testdir && touch file1.txt` as one chained command.
    3. Run `false || echo "fallback ran"` and confirm the fallback executes.
    4. Run `true ; echo "always runs"` to see unconditional execution.

---

# Lesson 4: Subcommands (Command Substitution)

- **Key Takeaway:** `$(command)` runs a command and substitutes its stdout output inline — useful for building strings/logs dynamically. Prefer `$()` over legacy backticks `` `command` `` (nestable, more readable).
    
- **Essential Commands & Examples:**
    

```bash
echo "I think $(whoami) is a cool user"     # inject command output into a string
echo "The current date is $(date)"

# Build a log line combining two subcommands:
echo "$(date +%x) - $(uptime)" >> log.txt

# Legacy syntax (still works, avoid for new scripts):
echo `whoami`
```

- `$()` advantages over backticks: supports **nesting** multiple subcommands, and is visually less ambiguous.
    
- `.bashrc` is _always_ read on every new shell; `.bash_profile`/equivalent only runs on the **first interactive login** of the day — different shells use different rc filenames (`.zshrc`, `.fishrc`, etc.) but the same pattern.
    
- **Quick Practice Challenge:**
    
    1. Run `echo "Today is $(date +%A)"`.
    2. Create a log line: `echo "$(whoami) - $(uptime)" >> mylog.txt` then `cat mylog.txt`.
    3. Try nesting: `echo "Files: $(ls $(pwd))"`.
    4. Repeat step 1 using backticks instead of `$()` to compare syntax.

---

# Lesson 5: SSH Setup — Creating a Second User/VM

- **Key Takeaway:** SSH (Secure Shell) lets you securely connect to and execute commands on a remote machine; Git and most cloud server access use it under the hood.
    
- **Essential Commands & Examples:**
    

```bash
# On the target ("secondary") machine — create a new user with bash shell + home dir:
sudo useradd -s /bin/bash -m -g ubuntu brian
sudo passwd brian          # set password
su brian                   # switch to the new user
```

- SSH keypair = private key (never share) + public key (shareable) → used for cryptographic handshake authentication instead of passwords.
    
- **Quick Practice Challenge:**
    
    1. Create a new local user: `sudo useradd -s /bin/bash -m newuser`.
    2. Set a password with `sudo passwd newuser`.
    3. Switch to that user with `su newuser` and confirm with `whoami`.
    4. Return to your original user with `exit`.

---

# Lesson 6: SSH Key-Based Authentication

- **Key Takeaway:** Generate an SSH keypair on the client, copy the **public** key into the remote user's `~/.ssh/authorized_keys`, then connect password-free with `ssh user@ip`.
    
- **Essential Commands & Examples:**
    

```bash
# On the CLIENT (machine connecting FROM):
ssh-keygen -t rsa                       # generates ~/.ssh/id_rsa (private) + id_rsa.pub (public)
cat ~/.ssh/id_rsa.pub                   # copy this output

# On the REMOTE/TARGET machine (machine connecting TO):
mkdir ~/.ssh
vi ~/.ssh/authorized_keys               # paste the public key here
chmod 700 ~/.ssh                        # lock dir to owner only
chmod 600 ~/.ssh/authorized_keys        # lock file to owner only

ifconfig                                # find remote machine's IP address (inet addr)

# Back on CLIENT — connect:
ssh brian@192.168.64.3                  # first connection asks to trust host fingerprint
exit                                     # disconnect
```

- Multiple clients can connect to the same remote user by appending each of their public keys as new lines in `authorized_keys`.
    
- `127.0.0.1` = loopback address (a machine referring to itself).
    
- **Quick Practice Challenge:**
    
    1. Generate a keypair: `ssh-keygen -t rsa` (accept defaults, no passphrase for practice).
    2. Copy your public key: `cat ~/.ssh/id_rsa.pub`.
    3. On a second machine/VM, append the key to `~/.ssh/authorized_keys` and set correct permissions (`700` dir / `600` file).
    4. SSH in from the first machine: `ssh user@<remote-ip>` and confirm password-free login.

---

# Lesson 7: SFTP — Secure File Transfer

- **Key Takeaway:** SFTP (Secure FTP) rides on the same SSH setup/port — if SSH works, SFTP works automatically; use it to move files between local and remote machines.
    
- **Essential Commands & Examples:**
    

```bash
sftp brian@192.168.64.3     # connect (reuses your SSH key setup)

pwd                          # remote working directory
lpwd                          # LOCAL working directory
ls                            # list remote files
lls                            # list LOCAL files
cd dir/                        # change remote directory
lcd dir/                        # change LOCAL directory

put localfile.txt              # upload local → remote
put localfile.txt newname.txt  # upload + rename
get remotefile.txt             # download remote → local
get remotefile.txt newname.txt # download + rename

!command                       # run a command on the LOCAL machine without leaving SFTP
help                            # list all sftp subcommands

bye / exit / quit               # disconnect
```

- SFTP is more secure than plain FTP since it's tunneled through SSH (encrypted, same keys/ports).
    
- **Quick Practice Challenge:**
    
    1. Connect: `sftp user@<remote-ip>`.
    2. Check both directories: `pwd` and `lpwd`.
    3. Upload a file: `put somefile.txt`, then confirm remotely with `ls`.
    4. Download a file back with a new name: `get somefile.txt copy.txt`, then `bye` to exit.

---

# Lesson 8: Wget — Downloading Files from the Internet

- **Key Takeaway:** `wget` downloads remote files (think "CP for the internet") and — unlike curl — can recursively crawl and download an entire website's linked assets.
    
- **Essential Commands & Examples:**
    

```bash
wget https://example.com/script.sh        # download file, keeps original filename
chmod +x script.sh                          # make downloaded script executable
./script.sh                                  # run it

wget --help                                   # see all wget options
```

- Wget's unique strength: **recursive downloads** (crawls linked CSS/JS/pages to mirror a whole site) — curl cannot do this.
    
- Otherwise, curl is generally preferred (better pipe/stdin-stdout integration with the rest of the Unix ecosystem).
    
- **Quick Practice Challenge:**
    
    1. Download any public script: `wget <url-to-a-.sh-file>`.
    2. Check its permissions with `ls -l` (not executable by default).
    3. Make it executable: `chmod +x filename.sh`.
    4. Run it: `./filename.sh`.

---

# Lesson 9: cURL Deep Dive (HTTP requests from the CLI)

- **Key Takeaway:** `curl` lets you send any HTTP verb, headers, cookies, and bodies from the terminal — essential for API testing/debugging without Postman. **Never blindly pipe `curl` output into `bash`** from untrusted sources.
    
- **Essential Commands & Examples:**
    

```bash
curl http://<ip>:8000                     # GET request, prints to stdout
curl http://<ip>:8000 > res.txt           # save response via redirect
curl -o res.txt http://<ip>:8000          # save response via curl's own flag
curl -O http://<ip>:8000/file.txt         # save using remote filename

curl -X POST http://<ip>:8000             # change HTTP verb (POST/PUT/PATCH/DELETE/etc.)
curl -d "post body data" http://<ip>:8000 # send POST body (implies POST)
curl -X PUT -d "body" http://<ip>:8000    # combine verb + body

curl -b "name=brian" http://<ip>:8000     # send a cookie
curl -c cookiejar.txt http://<ip>:8000    # save cookies to a "cookie jar" file

curl -L http://short.link                 # follow redirects (curl does NOT by default)
curl -H "Accept-Language: EN-US" \
     -H "Authorization: Bearer 12345" URL # send multiple headers

# ⚠️ SECURITY: never do this blindly:
curl https://untrusted.com/install.sh | bash
# Safer: download first, inspect, then run:
curl https://site.com/install.sh -o my-file.sh
cat my-file.sh        # review it
bash my-file.sh
```

- Browser DevTools → Network tab → right-click a request → **"Copy as cURL"** → paste in terminal to replay real requests exactly.
    
- **Quick Practice Challenge:**
    
    1. Start a test server: `python3 -m http.server 8000`.
    2. In another terminal, run `curl http://localhost:8000` and observe the HTML output.
    3. Run `curl -X POST -d "test=1" http://localhost:8000` and check the server log for the POST entry.
    4. Copy a request from your browser's DevTools as cURL and run it in your terminal.

---


# Lesson 10: cURL for API Development

- **Key Takeaway:** cURL is preferred over Wget for most workflows (pipeable, integrates with Unix tooling) and is the go-to tool for hitting/testing API endpoints without a GUI client like Postman.
    
- **Essential Commands & Examples:**
    

```bash
curl https://example.com/2048.sh > game.sh    # download via redirect (wget-style behavior)

# Spin up a quick local test API/static server:
python3 -m http.server 8000 --bind 0.0.0.0

ifconfig                                        # find your local IP to hit the server from elsewhere

curl http://192.168.64.2:8000                    # GET request against your local test server
```

- Wget is available almost everywhere curl is (and vice versa) except on some minimal distros (e.g., BusyBox/Alpine may ship only wget) — safe to standardize on curl for API/dev work.
    
- A quick Python static server (`python3 -m http.server`) is handy for locally testing curl requests/verbs/headers before hitting a real API.
    
- **Quick Practice Challenge:**
    
    1. Start a local server: `python3 -m http.server 8000`.
    2. In another terminal, run `curl http://localhost:8000` and view the raw HTML response.
    3. Save the response to a file: `curl http://localhost:8000 -o response.html`.
    4. Try a different verb: `curl -X POST http://localhost:8000` and observe the server's error response in the server logs.