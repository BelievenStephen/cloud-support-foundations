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

