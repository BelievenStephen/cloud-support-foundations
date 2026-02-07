# Disk Full Failure Drill 05

## Goal

Learn what a "disk full" situation looks like on Linux, how it breaks normal commands and tools, and how to recover safely.

---

## Environment

- **Server:** Ubuntu Server VM in UTM
- **Network:** UTM NAT
- **Access:** SSH from my Mac
- **Hostname:** `stephen-lab`
- **Timestamp:** `Sat Feb 7 06:10:06 PM UTC 2026`
- **Disk layout:** Root filesystem is `/dev/mapper/ubuntu--vg-ubuntu--lv` mounted on `/` (30G total)

---

## Baseline

### `df -h` (before filling)

```bash
df -h
```

**Output (root filesystem):**
```text
Filesystem                         Size  Used Avail Use% Mounted on
/dev/mapper/ubuntu--vg-ubuntu--lv   30G  6.5G   22G  23% /
```

✅ Root filesystem `/` had plenty of space.

---

### Check home directory usage

```bash
du -sh ~/* 2>/dev/null | sort -h | tail -n 15
```

**Observation:**  
Nothing looked unusually large. 

---

### Check SSH service logs

```bash
journalctl -u ssh --no-pager | tail -n 20
```

**Observation:**  
Mostly normal entries like SSH starting, listening on port 22, and accepted password logins.

---

## Break Phase (Fill the Disk)

### Location used

I used `/var/tmp/labfill` (safe and easy to clean)

### Setup commands

```bash
sudo mkdir -p /var/tmp/labfill
sudo chown "$USER":"$USER" /var/tmp/labfill
cd /var/tmp/labfill
```

**Check available space and tools:**
```bash
df -h .
df -B1 . | tail -n 1
which fallocate
```

**Result:**  
✅ `fallocate` exists at `/usr/bin/fallocate`

---

### Fill files created (in steps)

**Step 1: Create 20G file**
```bash
fallocate -l 20G fillfile.bin
df -h /
```

**Step 2: Add 1G file**
```bash
fallocate -l 1G fillfile2.bin
df -h /
```

---

**Step 3: Push closer to full (700M)**
```bash
fallocate -l 700M fillfile3.bin
df -h /
```

---

**Step 4: Push to 100% full (200M)**
```bash
fallocate -l 200M fillfile4.bin
```

**Result:**  
❌ `fallocate: fallocate failed: No space left on device`

```bash
df -h /
```

**Output:**
```text
Filesystem                         Size  Used Avail Use% Mounted on
/dev/mapper/ubuntu--vg-ubuntu--lv   30G   30G  115M 100% /
```

🔴 **Disk is now full!**

---

## Observe (What Breaks)

### Confirm disk is full

```bash
df -h /
```

**Output:**
```text
Filesystem                         Size  Used Avail Use% Mounted on
/dev/mapper/ubuntu--vg-ubuntu--lv   30G   30G  115M 100% /
```

---

### Commands that failed

#### Test 1: Basic write operation

```bash
echo "test" > /var/tmp/labfill/write-test.txt
```

**Error:**
```text
bash: echo: write error: No space left on device
```

❌ **Even simple writes fail**

---

#### Test 2: Try to allocate more space

```bash
fallocate -l 200M fillfile4.bin
```

**Error:**
```text
fallocate: fallocate failed: No space left on device
```

---

### Package management behavior

```bash
sudo apt update
```

**Errors observed:**
```text
Error writing to file - write (28: No space left on device)
[Multiple errors about failing to write repo index data under apt lists]
```

❌ **Package management breaks because it needs to write files under `/var/lib/...`**

---

### Check system logs

```bash
sudo journalctl -u ssh --no-pager | tail -n 20
```

**Observation:**  
SSH logs were mostly normal in my output at that moment (start/listen/accepted password). The big "signal" in this drill was the write failures and apt errors, not necessarily SSH-specific issues.

---

## Fix Phase (Recover Space)

### Delete fill files

**From `/var/tmp/labfill`:**
```bash
ls -lh
rm -f fillfile*.bin write-test.txt
sync
df -h /
```

**Output after cleanup:**
```text
Filesystem                         Size  Used Avail Use% Mounted on
/dev/mapper/ubuntu--vg-ubuntu--lv   30G  6.5G   22G  23% /
```

✅ **Root filesystem `/` returned to normal space levels!**

---

### Verify recovery with writes

```bash
echo "ok" > /var/tmp/labfill/write-test.txt
cat /var/tmp/labfill/write-test.txt
```

**Output:**
```text
ok
```

✅ **Write succeeded!**

---

### Verify package management works

```bash
sudo apt update
```

✅ **`apt update` succeeded normally again.**

---

### Final space check

```bash
df -h /
```

**Output:**
```text
Filesystem                         Size  Used Avail Use% Mounted on
/dev/mapper/ubuntu--vg-ubuntu--lv   30G  6.5G   22G  23% /
```

✅ **System fully recovered**

---

## Takeaways

### What I Learned

1. **A full disk breaks basic Linux behavior quickly**
   - Even simple writes (`echo > file`) fail with "No space left on device"
   
2. **Package management breaks**
   - `apt update` fails because it needs to write files under `/var/lib/...`
   
3. **The fastest diagnostic checks:**
   - `df -h` to confirm space usage across filesystems
   - `du -sh <path>/* | sort -h | tail` to find what is taking space
   
4. **Safe simulation approach:**
   - Using `/var/tmp/labfill` for test files makes cleanup easy
   - Deleting the fill files immediately recovers space

---

## Troubleshooting Pattern

### Symptoms of a full disk

| Symptom | What It Means |
|---------|---------------|
| `No space left on device` | Filesystem is at 100% capacity |
| `apt update` fails with write errors | Package cache can't be written |
| Application logs stop updating | Apps can't write new log entries |
| Services fail to start | Services need temp files or logs |

### Quick diagnostic commands

```bash
# Check all filesystems
df -h

# Find largest directories
du -sh /* 2>/dev/null | sort -h | tail -n 10

# Find largest files in a directory
find /var -type f -size +100M 2>/dev/null -exec ls -lh {} \; | sort -k 5 -h

# Check specific application logs or caches
du -sh /var/log/*
du -sh /var/lib/apt/lists/*
du -sh /tmp/*
```

### Common space hogs to check

- `/var/log/*` - Old or rotated logs
- `/var/lib/apt/lists/*` - Package cache
- `/tmp/*` and `/var/tmp/*` - Temporary files
- `/home/*/` - User data
- Docker/container images (if applicable)

---

## Notes

- **Always use test locations** like `/var/tmp/labfill` for drills - never fill production filesystems
- **Reserve space:** Many production systems keep 10-20% free space as a buffer
- **Monitor proactively:** Set up alerts when filesystems reach 80-85% capacity
- **Regular cleanup:** Implement log rotation and cleanup policies before space becomes critical
