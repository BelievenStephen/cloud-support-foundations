# Disk Pressure Drill 10 (Disk Full / Low Space Basics)

## Goal

Practice handling a "disk space low / disk full" ticket by:
- Proving current disk usage
- Finding the biggest directories/files
- Confirming what is consuming space
- Fixing safely and verifying recovery

---

## Environment

- **Server:** Ubuntu VM (UTM, NAT networking)
- **Hostname:** `stephen-lab`
- **Date/time:** Sat Feb 14, 2026

---

## Baseline (Before Break)

### Capture baseline disk usage

**Check overall disk usage:**
```bash
df -h /
```

**Output:**
```text
Filesystem                         Size  Used Avail Use% Mounted on
/dev/mapper/ubuntu--vg-ubuntu--lv   30G  7.5G   22G  25% /
```

✅ **Root filesystem had plenty of free space** (~25% used, ~22G available)

---

### Find biggest directories in home

```bash
du -xh --max-depth=1 ~ 2>/dev/null | sort -h | tail -n 15
```

**Observation:**  
✅ Home was very small (only a few KB / small folders)

---

### Find biggest directories under `/var`

```bash
sudo du -xh --max-depth=1 /var 2>/dev/null | sort -h | tail -n 15
```

**Observation:**  
📊 `/var` was the main consumer on the system (hundreds of MB)

**Common large directories:**
- `/var/log`
- `/var/lib`
- `/var/cache`

---

### Check journald disk usage

```bash
sudo journalctl --disk-usage
```

**Output:**
```text
Archived and active journals take up 184.0M in the file system.
```

📝 **Journals using ~184MB**

---

## Break Phase (Safe + Reversible)

### Create bounded "pressure" file

I created a 700M file inside my drill folder to simulate a disk-usage spike:

```bash
cd ~/drills/disk-pressure-10
fallocate -l 700M pressure.bin
ls -lh pressure.bin
```

**Output:**
```text
-rw-rw-r-- 1 user user 700M Feb 14 XX:XX pressure.bin
```

✅ **700M pressure file created**

---

### Re-check disk usage

```bash
df -h /
```

**Output:**
```text
Filesystem                         Size  Used Avail Use% Mounted on
/dev/mapper/ubuntu--vg-ubuntu--lv   30G  8.2G   21G  28% /
```

📈 **Root usage increased from ~25% to ~28%** (as expected)

---

## Observe (Find What Is Consuming Space)

### Confirm drill folder is now the hotspot

```bash
du -xh --max-depth=1 ~/drills/disk-pressure-10 2>/dev/null | sort -h | tail -n 15
```

**Output:**
```text
701M    /home/user/drills/disk-pressure-10
```

✅ **Drill folder shows ~701M**, which lines up with the pressure file

---

### Verify `/var` usage patterns

**Re-check `/var` directories:**
```bash
sudo du -xh --max-depth=1 /var 2>/dev/null | sort -h | tail -n 15
```

**Observation:**  
📊 `/var/log`, `/var/lib`, etc. were significant compared to most other directories

> 💡 **Common real-world hotspots:** Application logs, package caches, docker images, database files

---

### Simulate slow log growth (safe demonstration)

**Create/grow a fake application log:**
```bash
for i in {1..1000}; do 
  echo "$(date) INFO: Application event $i" >> ~/drills/disk-pressure-10/fake-app.log
  sleep 0.01
done
```

**Check the log file:**
```bash
ls -lh ~/drills/disk-pressure-10/fake-app.log
wc -l ~/drills/disk-pressure-10/fake-app.log
tail -n 3 ~/drills/disk-pressure-10/fake-app.log
```

**Observation:**  
📝 This demonstrated how logs can grow over time, even if slowly in this lab

---

## Fix Phase (Recover Space Safely)

### Remove the pressure file

```bash
rm -f ~/drills/disk-pressure-10/pressure.bin
```

---

### Verify disk space recovered

```bash
df -h /
```

**Output:**
```text
Filesystem                         Size  Used Avail Use% Mounted on
/dev/mapper/ubuntu--vg-ubuntu--lv   30G  7.5G   22G  25% /
```

✅ **Root filesystem returned to ~25% used (~22G available)**

---

### Cleanup drill artifacts

**Verify drill folder size dropped:**
```bash
ls -lh ~/drills/disk-pressure-10
du -sh ~/drills/disk-pressure-10
```

**Expected:**  
✅ Folder size back to KB range

---

**Remove simulated app log:**
```bash
rm -f ~/drills/disk-pressure-10/fake-app.log
```

---

## Takeaways

### Quick diagnostic questions

| Question | Command | What It Tells You |
|----------|---------|-------------------|
| Which filesystem is full? | `df -h` | Overall disk usage by filesystem |
| What's consuming space? | `du -sh /*` | Top-level directory sizes |
| Which directory is the culprit? | `du -xh --max-depth=1 <path> \| sort -h` | Sorted directory sizes |
| How much are logs using? | `sudo journalctl --disk-usage` | Systemd journal space |

---

### Common disk pressure hotspots

**In production environments:**

1. **`/var/log`** - Application and system logs
2. **`/var/lib`** - Application data (databases, docker, etc.)
3. **`/var/cache`** - Package manager caches
4. **`/tmp`** - Temporary files that weren't cleaned
5. **`/home/user`** - User data, downloads, cached files
6. **Journal logs** - systemd-journald archives

---

### Safe recovery pattern

**Follow this order:**

1. **Identify the largest culprit**
   ```bash
   sudo du -xh --max-depth=1 / 2>/dev/null | sort -h | tail -n 15
   ```

2. **Drill down into large directories**
   ```bash
   sudo du -xh --max-depth=1 /var 2>/dev/null | sort -h | tail -n 15
   ```

3. **Remove/move files or rotate/vacuum logs**
   ```bash
   # For application logs
   sudo rm -f /var/log/app/*.log.old
   
   # For journal logs
   sudo journalctl --vacuum-time=7d
   
   # For package cache (Debian/Ubuntu)
   sudo apt clean
   ```

4. **Re-check `df -h` to confirm recovery**
   ```bash
   df -h /
   ```

---

## Production Disk Space Commands

### Quick space checks

```bash
# Overall disk usage
df -h

# Top-level directories
sudo du -sh /* 2>/dev/null | sort -h

# Specific directory deep dive
sudo du -xh --max-depth=2 /var 2>/dev/null | sort -h | tail -n 20

# Find large files (over 100M)
sudo find / -type f -size +100M 2>/dev/null -exec ls -lh {} \; | sort -k 5 -h

# Check inode usage (sometimes the issue)
df -i
```

---

### Log management

```bash
# Check journal disk usage
sudo journalctl --disk-usage

# Vacuum journals (keep last 7 days)
sudo journalctl --vacuum-time=7d

# Vacuum journals (keep max 500M)
sudo journalctl --vacuum-size=500M

# Check largest log files
sudo du -sh /var/log/* 2>/dev/null | sort -h | tail -n 15

# Rotate logs immediately (if logrotate configured)
sudo logrotate -f /etc/logrotate.conf
```

---

### Package cache cleanup

**Debian/Ubuntu:**
```bash
# Remove package cache
sudo apt clean

# Remove old kernels (keeps current + one previous)
sudo apt autoremove --purge
```

**Red Hat/CentOS:**
```bash
# Clean yum cache
sudo yum clean all

# Clean dnf cache
sudo dnf clean all
```

---

## Quick Reference

### Disk full triage checklist

- [ ] Check overall disk usage: `df -h`
- [ ] Identify full filesystem
- [ ] Find top-level space consumers: `sudo du -sh /*`
- [ ] Drill down into largest directory
- [ ] Check journal logs: `sudo journalctl --disk-usage`
- [ ] Check for large files: `find / -type f -size +100M`
- [ ] Identify safe cleanup candidates
- [ ] Clean up (logs, cache, old files)
- [ ] Verify recovery: `df -h`

---

### Warning signs in production

| Disk Usage | Status | Action |
|------------|--------|--------|
| < 70% | ✅ Normal | Monitor |
| 70-85% | ⚠️ Warning | Investigate and plan cleanup |
| 85-95% | 🟡 Critical | Clean up soon |
| > 95% | 🔴 Emergency | Clean up immediately |

---

## Notes

- 🎯 **`df -h` answers:** "Which filesystem is full and how bad is it?"
- 🔍 **`du` answers:** "What directories/files are consuming the space?"
- 📝 **Journald can be significant:** Always check `journalctl --disk-usage` during incidents
- ⚠️ **Don't blindly delete:** Understand what you're removing before deletion
- 💡 **Automate cleanup:** Set up log rotation, automated vacuuming, and monitoring
- 🔒 **Production safety:** Always have backups before bulk deletions

---

## Related Drills

- [Drill 05: Disk Full Failure](disk-full-drill-05.md)
- [Drill 07: Disk Pressure (Controlled)](disk-pressure-drill-07.md)
