# Runbook: Disk Full (No space left on device)

## Goal

Restore normal operation when the system cannot write files because disk space is exhausted.

This runbook helps tell the difference between:

- Disk is actually full
- Inodes are exhausted (lots of tiny files)
- A single directory is consuming space (logs, caches, temp files)

---

## Scope / Assumptions

- Linux host (Ubuntu VM or server)
- You have shell access (SSH or console)
- You can use `sudo` if needed

---

## Symptoms (What I see)

### A) Write failures

**Examples:**

- `No space left on device`
- `Error writing ... (28: No space left on device)`
- Cannot create files, folders, or edit configs

**Common places this shows up:**

- `apt update` / package installs fail
- logs stop updating or spam errors
- apps crash when writing temp files

### B) System feels unstable

- Services fail to restart
- SSH may still work, but commands that write fail

---

## Quick Triage (Do these in order)

### 1) Confirm disk usage

```bash
df -h
df -h /
```

**Interpretation:**

- If `/` (or another filesystem) is ~95–100% used, disk space is the likely root cause.

### 2) Check inode exhaustion (less common, but important)

```bash
df -i
```

**Interpretation:**

- If `IUse%` is ~100%, you are out of **inodes** (too many small files), even if `df -h` shows free space.

### 3) Identify what is consuming space

**Top-level (safe starting point):**

```bash
sudo du -xhd 1 / | sort -h | tail -n 20
```

**If you do not want to scan all of `/`, start with common hotspots:**

```bash
sudo du -sh /var/* 2>/dev/null | sort -h | tail -n 20
sudo du -sh /home/* 2>/dev/null | sort -h | tail -n 20
```

**Common culprits:**

- `/var/log` (logs)
- `/var/cache` (package caches)
- `/tmp` and `/var/tmp` (temp files)
- large files in home directories

### 4) Confirm the biggest files (when needed)

```bash
sudo find /var -type f -size +200M -exec ls -lh {} \; 2>/dev/null | sort -k5 -h | tail -n 20
```

---

## Fix Actions (Choose based on what you find)

### Case 1: Safe temporary cleanup (recommended first)

Remove known temp/lab files:

```bash
sudo rm -rf /var/tmp/labfill
sudo rm -rf /tmp/*
sync
df -h /
```

### Case 2: Logs are too large

Check log sizes:

```bash
sudo du -sh /var/log/* 2>/dev/null | sort -h | tail -n 20
```

If `journalctl` logs are huge:

```bash
sudo journalctl --disk-usage
sudo journalctl --vacuum-size=200M
```

Re-check:

```bash
df -h /
```

### Case 3: apt cache is large

```bash
sudo du -sh /var/cache/apt 2>/dev/null
sudo apt-get clean
df -h /
```

### Case 4: Too many tiny files (inode exhaustion)

Find directories with lots of files:

```bash
sudo find /var -xdev -type f | wc -l
sudo find /tmp -xdev -type f | wc -l
```

Then remove the directory you confirm is safe (usually temp files), and re-check:

```bash
df -i
```

---

## Verification (Confirm recovery)

### 1) Confirm free space is back

```bash
df -h /
df -i
```

### 2) Confirm writes work again

```bash
echo test | sudo tee /var/tmp/disk-test.txt >/dev/null
sudo rm -f /var/tmp/disk-test.txt
```

### 3) Confirm packages/logs are functional (optional)

```bash
sudo apt-get update
sudo journalctl -n 20 --no-pager
```

---

## Fast Mapping (What it usually means)

| Symptom                                | Most likely cause         | First checks                       |
|----------------------------------------|---------------------------|------------------------------------|
| "No space left on device"              | Disk full                 | `df -h`, `du -sh /var/*`           |
| Disk not full but still errors         | Inodes full               | `df -i`                            |
| `apt update` fails writing lists/cache | `/var` full               | `df -h /var`, `sudo apt-get clean` |
| Logs not updating, journal errors      | `/var/log` / journal huge | `journalctl --disk-usage`          |

---

## Notes

- "Disk full" breaks many unrelated commands because everything needs to write: logs, caches, temp files, config edits.
- `df -h` tells you **where** you are full. `du` and `find` tell you **what** is consuming space.
- Prefer safe cleanup first: temp files and caches, then investigate logs and large files.
