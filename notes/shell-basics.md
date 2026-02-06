# MIT Missing Semester: Intro to Shell (Feb 6, 2026)

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
