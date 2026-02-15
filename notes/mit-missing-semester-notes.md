# MIT Missing Semester: Intro to Shell

## Feb 6, 2026

---

## What the shell is

- The shell is a text interface for running programs, giving them input, and reading output.
- A terminal is the app/window you type into. It displays the shell prompt and output.
- On Linux/macOS you often start in bash by default.

## Why it matters (for cloud/support work)

- Faster than clicking around for many tasks.
- Lets you combine commands to automate checks and troubleshooting.
- Common in open-source install steps, CI scripts, and debugging.

---

## Prompt basics

**Example prompt:**

```
missing:~$
```

**Meaning:**

- `missing` = machine name (example)
- `~` = current directory is home
- `$` = normal user (not root)

---

## Running commands + arguments

**Command format:**

```bash
program arg1 arg2 ...
```

**Examples:**

- `date` prints date/time
- `echo hello` prints `hello`

### Spaces in arguments

**Quote the argument:**

- `"My Photos"` or `'My Photos'`

**Or escape spaces:**

- `My\ Photos`

---

## Help for commands

- `man <command>` opens the manual page
  - Example: `man date`
- Many commands also support:
  - `<command> --help`

---

## Moving around (directories)

### `cd` (change directory)

- `cd /` goes to filesystem root
- `cd ~` goes to home directory
- `cd /bin` goes to `/bin`

**Notes:**

- `cd` is a shell builtin, not a separate program.

### Where am I?

- `pwd` prints current working directory
- `echo $PWD` prints the same info from an environment variable

### Absolute vs relative paths

**Absolute path** starts with `/` (from root)

- `/bin/echo`

**Relative path** does not start with `/` (from current directory)

- `cd bin` (when you are already in `/`)

**Special path parts:**

- `.` = current directory
- `..` = parent directory

---

## How the shell finds programs: `$PATH`

- The shell searches directories listed in the `$PATH` environment variable.

**View it:**

```bash
echo $PATH
```

**See which executable runs:**

```bash
which echo
```

**You can run a command by full path to bypass `$PATH`:**

```bash
/bin/echo hello
```

---

## Listing files and available programs

- `ls <dir>` lists files in a directory

**Example:**

```bash
ls /bin
```

---

## Useful "everyday" commands (file + text tools)

- `cat file` prints file contents
- `head file` prints first lines
- `tail file` prints last lines
- `sort file` sorts lines
- `uniq file` removes consecutive duplicate lines (often after `sort`)
- `grep "pattern" file` finds matching lines
  - For recursive search:
    ```bash
    grep -r "pattern" .
    ```

---

## Preview of more powerful tools (mention, not mastery yet)

These showed up in the lecture as "power tools" you will see later:

### `sed` (automated edits)

**Common form:**

```bash
sed -i 's/pattern/replacement/g' file
```

- Replaces text in a file (in-place with `-i`).

### `find` (search for files)

**Examples:**

```bash
find ~/Downloads -type f -name "*.zip" -mtime +30
find ~ -type f -size +100M -exec ls -lh {} \;
```

### `awk` (extract columns / parse structured text)

**Example:**

```bash
awk '{print $2}' file
```

- Prints 2nd whitespace-separated column.

---

## Key Idea

- Shell power comes from combining small commands to get a result.

---

# Feb 7, 2026

## Pipes and composition

- A pipe (`|`) sends the stdout (output) of one command into the stdin (input) of the next command.
- This works because many CLI tools read from stdin when no file is provided.
- Pipes let you compose simple tools into powerful workflows.

**Example pattern:**

```bash
command1 | command2 | command3
```

---

## Redirects (save output, feed input)

- `> file` writes stdout to a file (overwrites).
- `>> file` appends stdout to a file.
- `< file` uses a file as stdin (instead of your keyboard).

**Common examples:**

```bash
echo "hello" > out.txt
echo "more" >> out.txt
sort < out.txt
```

---

## Conditionals (run something only if another command succeeds)

**Bash pattern:**

```bash
if command1; then
  command2
  command3
fi
```

- `command1` runs first.
- If it succeeds (exit code 0), the `then` block runs.
- Often `command1` is `test` or `[ ... ]`, like "file exists" or "strings match".

**Common tests:**

```bash
[ -f file ]          # file exists
[ "$var" = "text" ]  # string equals
```

---

## Loops

### while loop

```bash
while command1; do
  command2
done
```

- Repeats as long as `command1` succeeds.

### for loop

```bash
for x in a b c; do
  echo "$x"
done
```

---

## Command substitution

- `$(...)` runs a command and substitutes its output into the current command.
- Prefer `$(...)` over old backticks.

**Example:**

```bash
for i in $(seq 1 10); do
  echo "$i"
done
```

---

## Shell scripts (.sh files)

- You can type long commands in the terminal, but scripts are cleaner and reusable.
- Scripts often start with a shebang so the system knows what interpreter to use:
  - `#!/bin/bash` runs the file using bash.

### Making scripts safer (strict mode)

```bash
set -euo pipefail
```

- `-e`: exit if a command fails
- `-u`: error on unset variables
- `-o pipefail`: fail if any command in a pipeline fails

---

## Background jobs

- Adding `&` runs a command in the background so you can keep using the terminal.

**Example:**

```bash
some_long_command &
```

---

## Exercises

### Setup / basics

- Check you're using a Unix shell: `echo $SHELL`
- Use `man` pages to learn flags (example: `man ls`)

### `ls -l` output

- Understand file type + permissions from the first characters in `ls -l`.

### Globs (pattern matching)

- Patterns like `*.txt`, `file?.txt`, `{a,b}.txt` to match filenames.

### Quoting

- Difference between single quotes, double quotes, and ANSI quotes.
- How to safely include special characters like `$`, `!`, and newlines.

### stdin / stdout / stderr

- Separate normal output vs errors using redirects.
- Practice redirecting both streams to the same file.

### Exit codes and chaining

- `$?` is last exit status (0 = success).
- `&&` runs next only on success, `||` runs next only on failure.

### Why `cd` is built-in

- `cd` must change the shell's own working directory (a child process can't change its parent's state).

### Simple scripting practice

- Use `$1` (first argument), `test -f`, and branching.
- Use `chmod +x` to make a script executable, and understand why that matters.

### Debugging scripts

- Adding `-x` to `set` prints commands as they run, useful for debugging.

### Practical text processing

- Use pipes with tools like `find`, `grep`, `sed`, `awk`, `sort`, `uniq`, `head/tail`.
- Learn `xargs` as another way to pass stdin lines as command arguments.

### Working with web + JSON

- `curl` to fetch content, `grep` to count patterns.
- `jq` to filter JSON fields based on conditions.

### Applied pipeline thinking

- Break down a multi-step SSH log pipeline.
- Build something similar from your own shell history.

---

# Feb 8, 2026

## Command-line Environment

### Big idea

- Shell scripts are not just "command launchers". They are a small programming language built around running programs and connecting them using shared conventions.

---

## Core concepts CLI programs use

- Arguments
- Streams (stdin, stdout, stderr)
- Environment variables
- Return codes (exit codes)
- Signals (interrupt/terminate)

---

## Arguments

- Programs receive arguments as strings.
- Example: `ls -l folder/` runs `/bin/ls` with args `-l` and `folder/`.

### Script argument variables

- `$0` = script name
- `$1..$9` = positional args
- `$@` = all args (list)
- `$#` = number of args

### Flags

- Short: `-a`, `-l` (often combinable: `-la`)
- Long: `--all`, `--help`, `--version`, `--verbose`
- Flag order usually does not matter.

### Variable number of args

- Many commands accept multiple targets: `mkdir src docs`

---

## Globbing (shell expands patterns before running the program)

- `*` = zero or more characters
- `?` = exactly one character
- `{a,b,c}` = expands to multiple items

**Examples:**

```bash
rm *.py                          # expands to matching files (program never sees *.py)
touch folder/{a,b,c}.py          # creates a.py b.py c.py
mv *{.py,.sh} folder             # moves all .py and .sh
```

---

## Streams and pipelines

- A pipeline like `cat file | grep ... | uniq ...` runs processes concurrently.
- `stdin` = standard input
- `stdout` = standard output (what pipes forward)
- `stderr` = error output (not piped by default)

### "Read from stdin" convention

- Many tools accept `-` meaning "read from stdin".

### Redirection basics

- `>` overwrite stdout to file
- `>>` append stdout to file
- `2>` redirect stderr
- `&>` redirect stdout+stderr
- `<` redirect file into stdin
- `/dev/null` discards output
  - Example: `cmd > /dev/null 2>&1`

### Tool mention

- `fzf` reads from stdin and provides an interactive filter/select UI.

---

## Variables and environment variables

### Shell variables

**Set:**

```bash
foo=bar    # no spaces
```

**Read:**

```bash
$foo
```

**Quoting:**

- `"..."` expands variables and substitutions
- `'...'` is literal, no expansion

### Command substitution

```bash
files=$(ls)    # captures stdout into a variable
```

### Process substitution

- `<(CMD)` runs CMD and provides a temp file path to programs that require a filename.

**Example:**

```bash
diff <(ls src) <(ls docs)
```

### Environment variables

**View:**

```bash
printenv
```

**Set for one command only:**

```bash
TZ=Asia/Tokyo date    # does not persist
```

**Persist for current shell + children:**

```bash
export DEBUG=1
```

**Remove:**

```bash
unset DEBUG
```

---

## Return codes (exit codes)

- Convention: `0` = success, nonzero = failure.
- Check last exit code: `echo $?`
- Exit in scripts: `exit 1` (or other number)

### Conditional execution with exit codes

- `cmd1 && cmd2` runs `cmd2` only if `cmd1` succeeded
- `cmd1 || cmd2` runs `cmd2` only if `cmd1` failed

**Examples:**

```bash
grep -q "pattern" file && echo "Found"
grep -q "pattern" file || echo "Not found"
```

**Note:** `if` and `while` also use command return codes for control flow.

---

## Signals (intro)

- `Ctrl+C` usually stops a running program by sending an interrupt signal.
- Sometimes it does not stop a program, depending on how the program handles signals.

---

# Feb 15, 2026

## Command-line Environment (cont'd)

## Signals (deep dive)

### What happens when you press Ctrl+C

1. You press `Ctrl+C`
2. Shell identifies the special character combination
3. Shell process sends SIGINT signal to the running process
4. Signal interrupts execution of the process
5. Process deals with signal and potentially changes execution flow

**Key concept:** Signals are software interrupts - a special communication mechanism between processes.

---

### Signal types and how to send them

**Common signals:**

| Signal | Keyboard | Purpose |
|--------|----------|---------|
| SIGINT | `Ctrl+C` | Interrupt (usually stops program) |
| SIGQUIT | `Ctrl+\` | Quit (with core dump) |
| SIGTERM | `kill -TERM <PID>` | Request graceful exit |
| SIGSTOP | (cannot catch) | Pause process |
| SIGTSTP | `Ctrl+Z` | Terminal stop (pause from terminal) |
| SIGKILL | `kill -9 <PID>` | Force immediate termination (cannot be caught) |
| SIGHUP | (when terminal closes) | Hangup |

---

### Example: Catching SIGINT

**Python program that ignores SIGINT:**
```python
#!/usr/bin/env python
import signal, time

def handler(signum, time):
    print("\nI got a SIGINT, but I am not stopping")

signal.signal(signal.SIGINT, handler)
i = 0
while True:
    time.sleep(.1)
    print("\r{}".format(i), end="")
    i += 1
```

**Running it:**
```bash
$ python sigint.py
24^C
I got a SIGINT, but I am not stopping
26^C
I got a SIGINT, but I am not stopping
30^\[1]    39913 quit       python sigint.py
```

**Note:** `^` is how Ctrl is displayed in terminal

---

### SIGKILL - the nuclear option

**Special properties:**
- Cannot be caught or ignored by process
- Always terminates immediately
- Can have bad side effects:
  - Leaves orphaned child processes
  - No cleanup possible

**When to use:**
- Last resort when process won't respond to SIGTERM
- Process is completely hung

---

### Job control

**Pause and resume processes:**

| Command | Purpose |
|---------|---------|
| `Ctrl+Z` | Pause (suspend) current process |
| `fg` | Resume in foreground |
| `bg` | Resume in background |
| `jobs` | List unfinished jobs in current session |
| `&` | Run command in background from start |

**Referring to jobs:**
- By PID: Use `pgrep` to find it
- By job number: `%1`, `%2`, etc. (from `jobs` output)
- Last backgrounded job: `$!`

---

### Example job control session
```bash
$ sleep 1000
^Z
[1]  + 18653 suspended  sleep 1000

$ nohup sleep 2000 &
[2] 18745
appending output to nohup.out

$ jobs
[1]  + suspended  sleep 1000
[2]  - running    nohup sleep 2000

$ kill -SIGHUP %1
[1]  + 18653 hangup     sleep 1000

$ kill -SIGHUP %2   # nohup protects from SIGHUP

$ jobs
[2]  + running    nohup sleep 2000

$ kill %2
[2]  + 18745 terminated  nohup sleep 2000
```

---

### Background processes and terminal closure

**Problem:**
- Backgrounded processes are still children of terminal
- Die when terminal closes (receives SIGHUP)

**Solutions:**

**Option 1: nohup**
```bash
$ nohup long_running_command &
```
- Wrapper that ignores SIGHUP
- Output appended to `nohup.out`

**Option 2: disown**
```bash
$ long_running_command &
$ disown
```
- Use if process already started
- Removes job from shell's job table

**Option 3: Terminal multiplexer**
- tmux or screen
- Covered in next section

---

### Trap for cleanup in scripts

**What trap does:**
- Execute commands when script receives signals
- Useful for cleanup operations

**Example:**
```bash
#!/usr/bin/env bash
cleanup() {
    echo "Cleaning up temporary files..."
    rm -f /tmp/mytemp.*
}
trap cleanup EXIT              # Run on script exit
trap cleanup SIGINT SIGTERM    # Also on Ctrl-C or kill
```

**Learn more:**
- `man signal`
- `kill -l` (list all signals)

---

## Remote Machines (SSH)

### Basic SSH connection

**Connect to remote server:**
```bash
ssh alice@server.mit.edu
```

**What this does:**
- Connects as user `alice`
- To server `server.mit.edu`
- Provides shell interface on remote machine

---

### Running commands remotely

**Non-interactive command execution:**

**Example 1: Command runs remotely, output processed locally**
```bash
ssh alice@server ls | wc -l
```
- `ls` runs on remote server
- `wc` runs on local machine

**Example 2: Both commands run remotely**
```bash
ssh alice@server 'ls | wc -l'
```
- Both `ls` and `wc` run on server
- Note the single quotes

**Key insight:** SSH correctly handles stdin and stdout, enabling powerful remote command composition

---

### SSH authentication

**Two methods:**

**1. Password authentication:**
- Simple but less secure
- Need to type password each time

**2. Key-based authentication (preferred):**
- More convenient
- More secure
- Uses public-key cryptography

---

### SSH key generation

**Generate key pair:**
```bash
ssh-keygen -a 100 -t ed25519 -f ~/.ssh/id_ed25519
```

**Key files:**
- Private key: `~/.ssh/id_ed25519` (treat like a password - never share!)
- Public key: `~/.ssh/id_ed25519.pub` (safe to share)

**Check if key has passphrase:**
```bash
ssh-keygen -y -f /path/to/key
```

**Note:** If you've configured GitHub SSH, you likely already have keys

---

### Copying public key to server

**Method 1: Manual copy**
```bash
cat .ssh/id_ed25519.pub | ssh alice@remote 'cat >> ~/.ssh/authorized_keys'
```

**Method 2: Using ssh-copy-id (simpler)**
```bash
ssh-copy-id -i .ssh/id_ed25519 alice@remote
```

**Server-side:**
- SSH looks in `.ssh/authorized_keys` to determine authorized clients

---

### File transfer with SSH

**scp (traditional tool):**
```bash
scp path/to/local_file remote_host:path/to/remote_file
```

**rsync (recommended - more efficient):**
```bash
rsync path/to/local_file remote_host:path/to/remote_file
```

**Why rsync is better:**
- Detects identical files (doesn't re-copy)
- Fine-grained control over symlinks and permissions
- `--partial` flag resumes interrupted transfers
- Similar syntax to scp

---

### SSH configuration file

**Location:** `~/.ssh/config`

**What it does:**
- Declare hosts
- Set default settings
- Used by ssh, scp, rsync, mosh, etc.

**Example configuration:**
```
Host vm
    User alice
    HostName 172.16.174.141
    Port 2222
    IdentityFile ~/.ssh/id_ed25519

# Configs can also take wildcards
Host *.mit.edu
    User alice
```

**Benefits:**
- Short aliases for complex connections
- Consistent settings across tools
- Easy to manage multiple remote hosts

---

## Terminal Multiplexers

### Why use a terminal multiplexer?

**Common needs:**
- Run multiple things at once
- Run editor and program side-by-side
- Detach sessions and reattach later
- Avoid using `nohup` for remote work

**Solution:** Terminal multiplexers like tmux

---

### tmux basics

**Key benefits:**
- Multiplex terminal windows using panes and tabs
- Detach sessions and reattach later
- Essential for remote work
- Sessions persist even if connection drops

---

### tmux keybinding pattern

**All keybindings follow this pattern:**
1. Press `Ctrl+b`
2. Release `Ctrl+b`
3. Press command key

**Notation:** `<C-b> x` means this three-step process

---

### tmux hierarchy

**Three levels:**

**1. Sessions** - Independent workspaces
```bash
tmux                    # Start new session
tmux new -s NAME        # Start with specific name
tmux ls                 # List sessions
tmux a                  # Attach to last session
tmux a -t NAME          # Attach to specific session
```

**Within tmux:**
- `<C-b> d` - Detach current session

---

**2. Windows** - Like tabs in browser/editor
- `<C-b> c` - Create new window
- `<C-b> N` - Go to Nth window (numbered)
- `<C-b> p` - Previous window
- `<C-b> n` - Next window
- `<C-b> ,` - Rename current window
- `<C-b> w` - List current windows
- `<C-d>` - Close window (terminate shell)

---

**3. Panes** - Split views (like vim splits)
- `<C-b> "` - Split horizontally
- `<C-b> %` - Split vertically
- `<C-b> <arrow>` - Move to pane in direction
- `<C-b> z` - Toggle zoom for current pane
- `<C-b> [` - Start scrollback mode
  - `<space>` - Start selection
  - `<enter>` - Copy selection
- `<C-b> <space>` - Cycle through pane arrangements

---

## Customizing the Shell

### Dotfiles overview

**What are dotfiles:**
- Plain-text configuration files
- Names begin with `.` (hidden by default)
- Examples: `~/.vimrc`, `~/.bashrc`, `~/.gitconfig`

**Shell startup:**
- Shell reads many files on startup
- Loading process can be complex
- Depends on shell type and login/interactive mode

---

### Common dotfiles

**Shell configuration:**
- bash: `~/.bashrc`, `~/.bash_profile`
- zsh: `~/.zshrc`

**Other tools:**
- git: `~/.gitconfig`
- vim: `~/.vimrc` and `~/.vim/` folder
- ssh: `~/.ssh/config`
- tmux: `~/.tmux.conf`

---

### Modifying PATH

**Common pattern when installing software:**
```bash
export PATH="$PATH:path/to/append"
```

**What this does:**
- Sets PATH to current value + new path
- Child processes inherit new PATH value
- Allows children to find programs in new location

---

### Package managers

**Why use them:**
- Handle downloading, installing, updating software
- Make customization easy

**Common package managers:**
- macOS: Homebrew
- Ubuntu/Debian: apt
- Fedora: dnf
- Arch: pacman

**Example (Homebrew):**
```bash
# ripgrep: faster grep with better defaults
brew install ripgrep

# fd: faster, user-friendly find
brew install fd
```

---

### Finding installation commands

**command-not-found.com:**
- Search for any command
- Shows how to install across different package managers

**tldr - simplified man pages:**
```bash
$ tldr fd
  An alternative to find.
  Aims to be faster and easier to use than find.

  Recursively find files matching a pattern in the current directory:
      fd "pattern"

  Find files that begin with "foo":
      fd "^foo"

  Find files with a specific extension:
      fd --extension txt
```

---

### Shell aliases

**What aliases do:**
- Create shortcuts for commands
- Shell replaces automatically before evaluation

**Basic syntax:**
```bash
alias alias_name="command_to_alias arg1 arg2"
```

---

### Useful alias examples

**Shorthands:**
```bash
alias ll="ls -lh"
alias gs="git status"
alias gc="git commit"
```

**Prevent mistakes:**
```bash
alias sl=ls
```

**Better defaults:**
```bash
alias mv="mv -i"           # Prompt before overwrite
alias mkdir="mkdir -p"     # Make parent dirs as needed
alias df="df -h"           # Human readable format
```

**Composed aliases:**
```bash
alias la="ls -A"
alias lla="la -l"
```

---

### Managing aliases

**Ignore an alias temporarily:**
```bash
\ls
```

**Remove an alias:**
```bash
unalias la
```

**Get alias definition:**
```bash
alias ll
# Prints: ll='ls -lh'
```

**Limitation:** Aliases cannot take arguments in middle of command
- For complex behavior, use shell functions instead

---

### History search improvements

**Basic reverse search:**
- `Ctrl+R` - Search through previous commands

**With fzf (fuzzy finder):**
- Much more powerful
- Interactive fuzzy search through entire history
- Configure fzf shell integration for this

---

### Dotfiles organization best practices

**Recommended approach:**
1. Keep dotfiles in their own folder
2. Put under version control (git)
3. Symlink into place using a script

**Benefits:**
- Easy installation on new machine
- Portability across systems
- Synchronization across machines
- Change tracking over your career

---

### What to put in dotfiles

**Sources of ideas:**
- Read online documentation
- Read man pages
- Search for blog posts about specific programs
- Browse others' dotfiles on GitHub
- [Most popular dotfiles repo](https://github.com/mathiasbynens/dotfiles)

**Class instructors' dotfiles:**
- Anish's dotfiles
- Jon's dotfiles
- Jose's dotfiles

**Warning:** Don't blindly copy configurations - understand what they do first

---

### Shell frameworks and plugins

**Popular frameworks:**
- prezto
- oh-my-zsh

**Useful zsh plugins:**
- zsh-syntax-highlighting: Colors valid/invalid commands
- zsh-autosuggestions: Suggests commands from history
- zsh-completions: Additional completion definitions
- zsh-history-substring-search: Fish-like history search
- powerlevel10k: Fast, customizable prompt theme

**Note:** Fish shell includes many of these features by default

---

## AI in the Shell

### Command generation

**llm tool for generating commands:**
```bash
$ llm cmd "find all python files modified in the last week"
find . -name "*.py" -mtime -7
```

---

### Pipeline integration

**Use LLMs to extract from inconsistent formats:**

**Example:**
```bash
$ cat users.txt
Contact: john.doe@example.com
User 'alice_smith' logged in at 3pm
Posted by: @bob_jones on Twitter
Author: Jane Doe (jdoe)
Message from mike_wilson yesterday
Submitted by user: sarah.connor

$ INSTRUCTIONS="Extract just the username from each line, one per line, nothing else"
$ llm "$INSTRUCTIONS" < users.txt
john.doe
alice_smith
bob_jones
jdoe
mike_wilson
sarah.connor
```

**Important notes:**
- Use `"$INSTRUCTIONS"` (quoted) because variable contains spaces
- Use `< users.txt` to redirect file content to stdin

---

### AI shells

**Tools like Claude Code:**
- Act as meta-shell
- Accept English commands
- Translate to shell operations, file edits, multi-step tasks

---

## Terminal Emulators

### What to consider

**Why it matters:**
- You'll spend hundreds/thousands of hours in terminal
- Worth investing time in settings

**Aspects to customize:**
- Font choice
- Color scheme
- Keyboard shortcuts
- Tab/pane support
- Scrollback configuration
- Performance (Alacritty, Ghostty offer GPU acceleration)

---

## Exercises

### Arguments and Globs

1. **Understanding `--` separator:**
   - Try: `touch -- -myfile`
   - Then remove without `--`
   - Why is `--` useful?

2. **ls command practice:**
   - List all files (including hidden)
   - Human readable sizes
   - Ordered by recency
   - Colorized output

3. **Process substitution:**
   - Use `diff <(printenv | sort) <(export | sort)`
   - Why are they different?

---

### Environment Variables

1. **marco and polo functions:**
   - `marco`: Save current directory
   - `polo`: Return to saved directory
   - Write in `marco.sh` and load with `source marco.sh`

---

### Return Codes

1. **Capture failing command output:**
   - Run script until it fails
   - Capture stdout and stderr to files
   - Report how many runs it took to fail

---

### Signals and Job Control

1. **Job control practice:**
   - Start `sleep 10000`
   - Background with `Ctrl+Z` and `bg`
   - Use `pgrep` and `pkill` without typing PID

2. **Process waiting:**
   - Start `sleep 60 &`
   - Wait for it to complete before running `ls`
   - Write `pidwait` function that waits for any PID

---

### Files and Permissions

1. **Find most recent file:**
   - Recursively find most recently modified file
   - List all files by recency

---

### Terminal Multiplexers

1. **tmux tutorial:**
   - Follow tutorial
   - Learn basic customizations

---

### Aliases and Dotfiles

1. **Create alias:** `dc` → `cd`

2. **Find top 10 commands:**
```bash
   history | awk '{$1="";print substr($0,2)}' | sort | uniq -c | sort -n | tail -n 10
```
   - Create aliases for frequently used commands

3. **Dotfiles setup:**
   - Create dotfiles folder with version control
   - Add configuration for at least one program
   - Set up install script
   - Test on fresh VM
   - Migrate current configurations
   - Publish on GitHub

---

### Remote Machines (SSH)

1. **Generate SSH keys:**
   - Check `~/.ssh/` for existing keys
   - Generate with `ssh-keygen -a 100 -t ed25519`
   - Use password and ssh-agent

2. **Configure SSH:**
   - Edit `.ssh/config` with host entry
   - Include LocalForward for port forwarding

3. **Copy SSH key:**
   - Use `ssh-copy-id vm`

4. **Test web server:**
   - Start `python -m http.server 8888` on VM
   - Access via `http://localhost:9999` on local machine

5. **Secure SSH server:**
   - Disable password authentication
   - Disable root login
   - Restart sshd service

6. **Challenge: mosh:**
   - Install mosh
   - Test connection recovery after network disconnect

7. **Challenge: Port forwarding:**
   - Learn `-N` and `-f` flags
   - Set up background port forwarding

---
