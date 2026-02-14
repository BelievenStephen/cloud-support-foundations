# Runbook: Disk Pressure / Low Space Triage

## Purpose

Triage disk space pressure early (80–95%) and stop active growth before it becomes "disk full."

**Key difference from "Disk Full recovery":** This focuses on fast isolation + growth confirmation + containment.

---

## Symptoms

### Common reports and signals:

- Disk usage alerts at **80% / 90% / 95%**
- Slow writes, slow app responses, slow package installs
- Log spam or rapidly growing log files
- Services failing to write state, caches, or temp files
- Late-stage errors: `No space left on device`

---

## Fast Checks (In Order)

### 1) Confirm pressure and which filesystem is impacted

```bash
df -h /
```

**Notes:**
- If `/` is not the problem, run `df -h` and target the affected mount
- Record: filesystem, size, used %, available

**Example output:**
```text
Filesystem                         Size  Used Avail Use% Mounted on
/dev/mapper/ubuntu--vg-ubuntu--lv   30G   27G    3G  90% /
```

⚠️ **90% usage - action required**

---

### 2) Find the biggest consumers under `/var` (most common culprit area)

```bash
sudo du -xh --max-depth=1 /var 2>/dev/null | sort -h | tail -n 15
```

**Common hotspots:**

| Location | Typical Contents |
|----------|------------------|
| `/var/log` | Application and system logs |
| `/var/lib` | Containers, packages, databases |
| `/var/cache` | Package manager caches |
| `/var/tmp` | Temporary application files |

---

### 3) Check home directory usage (user-level culprits)

```bash
du -xh --max-depth=1 ~ 2>/dev/null | sort -h | tail -n 15
```

**Common hotspots:**
- `~/Downloads`
- `~/Videos`
- Build artifacts
- Large local files
- Docker volumes (if using rootless Docker)

---

### 4) Check systemd journal size

```bash
sudo journalctl --disk-usage
```

**Example output:**
```text
Archived and active journals take up 2.5G in the file system.
```

💡 **Important:** Journal can be a contributor even when normal log files look small.

---

## Growth Check (Key Difference)

**Goal:** Confirm what is actively growing right now.

### Process:

1. Pick a suspect directory
2. Measure size now
3. Wait
4. Measure again

```bash
sudo du -sh /var/log
sleep 60
sudo du -sh /var/log
```

**Interpretation:**

| Behavior | Diagnosis | Action |
|----------|-----------|--------|
| Size increases quickly | Active writer problem | Containment first, then clean |
| Size stable | Historical accumulation | Clean up old files |
| Size decreases | Cleanup already happening | Monitor and verify |

---

## Containment Steps (Key Difference)

Use these when growth is active or space is approaching a critical threshold.

### A) Stop the writer service (preferred first move)

**If you can identify the service producing the growth:**

```bash
sudo systemctl stop <service>
sudo systemctl status <service> --no-pager
```

**If you do not know the service yet:**

**Option 1: Check which logs are growing**
- See "Deep dive" section below

**Option 2: Check top processes writing to disk**
```bash
# Deleted-but-open files
sudo lsof +L1

# Processes accessing /var/log
sudo lsof | grep /var/log
```

---

### B) Truncate a log safely (use when a single log is the culprit)

**Why truncate vs delete:**
- Truncate keeps the file path intact (safer for apps than deleting)
- Apps can continue writing without error

```bash
sudo truncate -s 0 /path/to/logfile.log
```

**Confirm it shrank:**
```bash
ls -lh /path/to/logfile.log
```

**Expected:**
```text
-rw-r--r-- 1 root root 0 Feb 14 XX:XX /path/to/logfile.log
```

---

### C) Vacuum systemd journal (if journal is large)

**Time-based vacuum (keep last 7 days):**
```bash
sudo journalctl --vacuum-time=7d
```

**Size-based vacuum (limit to 200M):**
```bash
sudo journalctl --vacuum-size=200M
```

**Re-check:**
```bash
sudo journalctl --disk-usage
```

**Example output after vacuum:**
```text
Archived and active journals take up 184.0M in the file system.
```

---

## Fix / Cleanup Options

Use after containment (or if growth is not active).

### 1) Remove known safe temp files

```bash
# User files
rm -f ~/bigfile

# System temp files (be careful!)
sudo rm -f /tmp/<known-safe-file>
```

⚠️ **Warning:** Never blindly delete from `/tmp` - verify files are safe to remove

---

### 2) Clean caches (careful in production)

**Ubuntu/Debian:**
```bash
sudo apt-get clean
```

**Red Hat/CentOS:**
```bash
sudo yum clean all
# or
sudo dnf clean all
```

---

### 3) Log rotation review (if recurring)

**Check log rotation configuration:**
```bash
ls -la /etc/logrotate.d/
cat /etc/logrotate.d/<service>
```

**Ensure:**
- Rotation is enabled
- Service is logging to a rotating target
- Rotation frequency is appropriate

**Force rotation (if needed):**
```bash
sudo logrotate -f /etc/logrotate.conf
```

---

## Verify (Must Pass All)

### 1) Space recovered

```bash
df -h /
```

**Success criteria:**
- ✅ Available space increases
- ✅ Usage drops below alert threshold (or at least stabilizes)

**Example:**
```text
Before:  30G   27G    3G  90% /
After:   30G   24G    6G  80% /
```

---

### 2) Writer stopped (if applicable)

**Re-run growth check on suspect directory:**

```bash
sudo du -sh /var/log
sleep 60
sudo du -sh /var/log
```

**Success criteria:**
- ✅ Size stable
- ✅ OR growing slowly at expected rate

---

### 3) Service can run and write again

```bash
sudo systemctl start <service>
sudo systemctl status <service> --no-pager
```

**If the service is HTTP:**
```bash
curl -I http://127.0.0.1:<port>
```

**Success criteria:**
- ✅ Service is active (running)
- ✅ App can write logs/state without errors

---

## Deep Dive (Optional, When Fast Checks Are Not Enough)

### Find the largest directory or file under a hotspot

```bash
sudo du -xh /var/log 2>/dev/null | sort -h | tail -n 25
```

**Example output:**
```text
45M     /var/log/nginx
123M    /var/log/apache2
890M    /var/log/myapp
1.2G    /var/log
```

---

### Check for deleted-but-open files (space not freed after delete)

```bash
sudo lsof +L1 | head
```

**What this shows:**
- Files that were deleted but are still held open by a process
- Space won't be freed until the process closes the file or restarts

**If you see large deleted files:**
- Restart the owning process/service to release them
- Use `systemctl restart <service>` for systemd services

---

### Find files actively being written to

```bash
# Watch for writes in real-time
sudo iotop -o

# Check open file handles for writes
sudo lsof -n | grep -E 'REG.*w'
```

---

### Find largest files across entire filesystem

```bash
sudo find / -type f -size +100M 2>/dev/null -exec ls -lh {} \; | sort -k 5 -h | tail -n 20
```

---

## Decision Logic / Notes

### Pressure thresholds and actions

| Disk Usage | Status | Action Required |
|------------|--------|-----------------|
| 80-85% | ⚠️ Warning | Investigate, plan cleanup |
| 85-90% | 🟡 Concerning | Act within 24 hours |
| 90-95% | 🟠 Critical | Act within 1 hour |
| 95-100% | 🔴 Emergency | Act immediately, containment first |

---

### Key principles

**Containment first when:**
- Directory grows within 60 seconds
- Disk usage > 90%
- Active writer identified

**Cleanup first when:**
- Growth is slow or stopped
- Historical accumulation
- Disk usage < 90%

**Always check journald:**
- Journal can be hidden contributor
- Use `journalctl --disk-usage`
- Vacuum if needed

---

## Quick Reference Commands

```bash
# Check all filesystems
df -h

# Find top space consumers in /var
sudo du -xh --max-depth=1 /var 2>/dev/null | sort -h | tail -n 15

# Check journal size
sudo journalctl --disk-usage

# Growth test (1 minute)
sudo du -sh /var/log && sleep 60 && sudo du -sh /var/log

# Truncate log safely
sudo truncate -s 0 /path/to/file.log

# Vacuum journal (7 days)
sudo journalctl --vacuum-time=7d

# Find deleted-but-open files
sudo lsof +L1

# Clean package cache (Ubuntu)
sudo apt-get clean

# Force log rotation
sudo logrotate -f /etc/logrotate.conf

# Find large files
sudo find /var -type f -size +100M 2>/dev/null -exec ls -lh {} \; | sort -k 5 -h
```

---

## Common Scenarios

### Scenario 1: Rapidly growing application log

**Symptoms:**
- Single log file growing rapidly
- Service known to be noisy

**Actions:**
1. Truncate the log: `sudo truncate -s 0 /var/log/app/app.log`
2. Review app logging level (reduce verbosity if possible)
3. Ensure log rotation is configured

---

### Scenario 2: Journal consuming multiple GB

**Symptoms:**
- `journalctl --disk-usage` shows > 1GB
- Recent system had high activity or errors

**Actions:**
1. Vacuum by time: `sudo journalctl --vacuum-time=7d`
2. Or vacuum by size: `sudo journalctl --vacuum-size=500M`
3. Configure persistent limit in `/etc/systemd/journald.conf`:
   ```
   SystemMaxUse=500M
   ```

---

### Scenario 3: Unknown writer, disk filling quickly

**Symptoms:**
- Disk usage increasing rapidly
- Source unknown

**Actions:**
1. Stop non-critical services temporarily
2. Use `sudo lsof | grep -E 'REG.*w'` to find writers
3. Use `sudo iotop -o` to watch real-time I/O
4. Identify and contain the writer
