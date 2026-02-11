# Disk Pressure Drill 07

## Goal

Recognize disk pressure symptoms and run fast checks to confirm what is filling the disk, then fix it and verify recovery.

---

## Environment

- **Server:** Ubuntu VM (SSH)
- **Network:** UTM NAT networking
- **Date:** Wed Feb 11, 2026

---

## Baseline (Normal Disk State)

### `df -h` (and `df -h /`)

**`/` filesystem baseline:**
```bash
df -h /
```

**Output:**
```text
Filesystem                         Size  Used Avail Use% Mounted on
/dev/mapper/ubuntu--vg-ubuntu--lv   30G  6.6G   22G  24% /
```

✅ **Plenty of space available**

---

### `lsblk`

```bash
lsblk
```

**Key observations:**
- **Disk:** `vda` (64G total)
- **Root filesystem:** Mounted on `/` via LVM (`/dev/mapper/ubuntu--vg-ubuntu--lv`) sized ~30.5G
- **Separate partitions:** `/boot` and `/boot/efi` present

---

### `du -sh .` and `du -sh *`

**In `~/drills/disk-pressure-07` before creating the large file:**

```bash
du -sh .
```

**Observation:**  
✅ Folder was very small (only `drill.log` present)

---

### `df -i` (inode usage)

```bash
df -i /
```

**Observation:**  
✅ Inodes were not close to full. Root filesystem inode usage was low (single-digit %)

---

## Break Phase (Induce Disk Pressure)

### Method used

Created a large file using `fallocate`.

### Commands used

```bash
fallocate -l 3G bigfile.bin
ls -lh bigfile.bin
```

**Output:**
```text
-rw-rw-r-- 1 user user 3.0G Feb 11 XX:XX bigfile.bin
```

---

### Disk consumed / % used reached

**After creating the 3G file:**

```bash
df -h /
```

**Output:**
```text
Filesystem                         Size  Used Avail Use% Mounted on
/dev/mapper/ubuntu--vg-ubuntu--lv   30G  9.6G   19G  34% /
```

📊 **Disk usage increased from 24% to 34%** (added 3.0G)

---

## Observe (During Disk Pressure)

### `df -h` during "load"

```bash
df -h /
```

**Output:**
```text
Filesystem                         Size  Used Avail Use% Mounted on
/dev/mapper/ubuntu--vg-ubuntu--lv   30G  9.6G   19G  34% /
```

⚠️ **Disk usage at 34%** - not critical, but noticeable increase

---

### Write failure testing

**No failures occurred.** Small writes were successful and fast.

**Small write test:**
```bash
time bash -c 'for i in {1..2000}; do echo x >> smallwrites.log; done'
ls -lh smallwrites.log
```

**Results:**
- ✅ Completed quickly (about 0.05s)
- ✅ `smallwrites.log` ended up ~4.0K

> 💡 **Key observation:** System remained fully functional. At 34% disk usage, there's still plenty of space for normal operations.

---

### Kernel warnings check

```bash
dmesg | tail -n 50
sudo journalctl -k | tail -n 50
```

**Observation:**  
✅ **No kernel warnings were observed during this run**

---

## Fix Phase (Recovery)

### What I deleted

```bash
rm -f bigfile.bin smallwrites.log
sync
```

**Actions:**
- Removed the 3G test file (`bigfile.bin`)
- Removed the write test log (`smallwrites.log`)
- Ran `sync` to ensure changes are written to disk

---

### `df -h` after recovery

```bash
df -h /
```

**Output:**
```text
Filesystem                         Size  Used Avail Use% Mounted on
/dev/mapper/ubuntu--vg-ubuntu--lv   30G  6.6G   22G  24% /
```

✅ **Disk usage returned to baseline (24%)**

---

### Verify with `du`

```bash
du -sh .
```

**Output:**
```text
24K     .
```

✅ **Folder returned to small size, leaving only `drill.log`**

---

## Takeaways

### What I Learned

1. 🎯 **Fastest diagnostic checks:**
   ```bash
   df -h              # Overall filesystem usage
   du -sh .           # Space used by current directory
   du -sh *           # Space used by each item in directory
   lsblk              # What disks/partitions exist and where they mount
   ```

2. 🔴 **Common symptom when truly full:**
   - "No space left on device" error message

3. 🧹 **Typical fixes:**
   - Delete or rotate large logs
   - Clear temp files (`/tmp`, `/var/tmp`)
   - Remove large unused files
   - Expand the disk or filesystem when appropriate

4. 💡 **Important distinction:**
   - This drill simulated controlled disk pressure (34% usage)
   - Real disk-full situations (95-100%) cause immediate failures
   - See Drill 05 for full disk failure symptoms

---

## Troubleshooting Pattern

### Quick diagnostic workflow

```bash
# Step 1: Check overall disk usage
df -h

# Step 2: Identify which filesystem is full
df -h | grep -E '(Filesystem|9[0-9]%|100%)'

# Step 3: Find what's consuming space
du -sh /* 2>/dev/null | sort -h | tail -n 10

# Step 4: Drill down into large directories
cd /var
du -sh * 2>/dev/null | sort -h | tail -n 10
```

---

### Common space hogs to check

| Location | What to Look For | Common Fix |
|----------|------------------|------------|
| `/var/log/*` | Old or large log files | Rotate logs, delete old logs |
| `/tmp/*` and `/var/tmp/*` | Temporary files | Clear old temp files |
| `/var/lib/apt/lists/*` | APT cache (Debian/Ubuntu) | `sudo apt clean` |
| `/home/*/` | User data | Ask users to clean up |
| `/var/lib/docker/*` | Docker images/containers | `docker system prune` |

---

### Comparison: Disk Pressure vs Disk Full

| State | % Used | Symptoms | Urgency |
|-------|--------|----------|---------|
| **Normal** | <70% | No issues | ✅ None |
| **Pressure** | 70-85% | System works normally, should investigate | ⚠️ Monitor |
| **Warning** | 85-95% | May see slowdowns, should act soon | 🟡 Act soon |
| **Critical** | 95-100% | "No space left on device" errors | 🔴 Immediate |

---

## Quick Reference Commands

```bash
# Check all filesystem usage
df -h

# Check specific filesystem
df -h /

# Find largest directories (top 10)
du -sh /* 2>/dev/null | sort -h | tail -n 10

# Find largest files in a directory
find /var -type f -size +100M 2>/dev/null -exec ls -lh {} \; | sort -k 5 -h

# Check inode usage (sometimes the issue)
df -i

# View disk and partition layout
lsblk

# Clean package cache (Debian/Ubuntu)
sudo apt clean

# Clean package cache (Red Hat/CentOS)
sudo dnf clean all

# Find and delete old log files (example)
sudo find /var/log -type f -name "*.log.gz" -mtime +30 -delete

# Free up journal logs (keep last 3 days)
sudo journalctl --vacuum-time=3d
```

---

## Notes

- ⚠️ **Prevention in production:**
  - Set up disk usage alerts (80-85% threshold)
  - Implement log rotation policies
  - Monitor growth trends over time
  - Plan capacity increases before reaching critical levels

- 🔍 **Finding hidden space usage:**
  ```bash
  # Files that have been deleted but still held open
  sudo lsof | grep deleted
  
  # Hidden files in home directory
  du -sh ~/.??* | sort -h
  ```

- 🛠️ **Emergency response when disk is full:**
  1. Free up space immediately (delete logs, clear temp files)
  2. Identify root cause (what filled the disk?)
  3. Implement longer-term fix (log rotation, monitoring, capacity planning)
  4. Document the incident for future reference
