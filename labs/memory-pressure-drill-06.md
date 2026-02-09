# Memory Pressure Drill 06

## Goal

Recognize "low memory" symptoms fast and run first checks quickly (memory, swap, top processes, kernel signals). Stop the load and confirm recovery.

---

## Environment

- **Server:** Ubuntu VM on UTM with NAT networking
- **Access:** SSH from host machine
- **Hostname:** `stephen-lab`
- **Date/time:** Mon Feb 9, 2026 

---

## Baseline (Normal Memory State)

### `free -h`

```bash
free -h
```

**Output:**
```text
              total        used        free      shared  buff/cache   available
Mem:           3.8Gi       285Mi       3.5Gi       1.0Mi       227Mi       3.5Gi
Swap:          3.8Gi          0B       3.8Gi
```

✅ **Plenty of free memory, no swap in use**

---

### `uptime`

```bash
uptime
```

**Output:**
```text
up a few minutes, load average: 0.01, 0.06, 0.02
```

✅ **System idle, normal load average**

---

### Top snapshot

```bash
top -b -n 1 | head -n 30
```

**Memory overview from `top`:**
```text
MiB Mem :   3902.0 total,   3533.2 free,    287.1 used,    227.4 buff/cache
MiB Swap:   3901.0 total,   3901.0 free,      0.0 used.   3615.4 avail Mem
```

✅ **No heavy memory consumers at baseline**

---

### Process memory usage

```bash
ps aux --sort=-%mem | head -n 15
```

**Top memory users (examples):**
- `multipathd` ~0.6% MEM
- `unattended-upgrades` (python process) ~0.5% MEM
- `journald`, `udisksd`, system services were low

✅ **All processes using minimal memory**

---

### Swap status

```bash
swapon --show
```

**Output:**
```text
NAME      TYPE SIZE USED PRIO
/swap.img file 3.8G   0B   -2
```

✅ **Swap present but unused**

---

## Break Phase (Induce Memory Pressure)

### Method used

Python memory allocation loop that allocates memory in chunks, then "holds" it until Ctrl+C.

### Command used

```bash
python3 - <<'PY'
import time
chunk = 50 * 1024 * 1024       # 50MB
target = 2000 * 1024 * 1024    # 2.0GB (safer than 2.5GB)
data = []
allocated = 0

try:
    while allocated < target:
        data.append(bytearray(chunk))
        allocated += chunk
        print(f"allocated={allocated//1024//1024}MB", flush=True)
        time.sleep(0.2)
except MemoryError:
    print("MemoryError hit", flush=True)

print("Holding allocation. Press Ctrl+C to stop and free memory.", flush=True)
try:
    while True:
        time.sleep(1)
except KeyboardInterrupt:
    pass
PY
```

### What I observed as memory climbed

**Terminal output:**
```text
allocated=100MB
allocated=150MB
allocated=200MB
allocated=250MB
...
allocated=1950MB
allocated=2000MB
Holding allocation. Press Ctrl+C to stop and free memory.
```

📈 **`python3` became the top memory consumer**

---

## Observe (During Memory Pressure)

### System responsiveness

⚠️ **Minor slowdown was possible, but the VM stayed responsive enough to run checks.**

✅ **No full lockup. Swap did not start climbing.**

---

### `free -h` during load

```bash
free -h
```

**Output:**
```text
              total        used        free      shared  buff/cache   available
Mem:           3.8Gi       2.3Gi       1.5Gi       1.0Mi       232Mi       1.5Gi
Swap:          3.8Gi          0B       3.8Gi
```

⚠️ **Memory usage jumped to 2.3Gi (was 285Mi at baseline)**  
⚠️ **Available memory dropped to 1.5Gi (was 3.5Gi at baseline)**  
✅ **Swap still at 0B used**

---

### `uptime` during load (load average)

```bash
uptime
```

**Load average observed in `top`:**
```text
load average: 0.26, 0.06, 0.02
```

💡 **Load average stayed low** - this makes sense because the test was memory pressure, not heavy CPU usage.

---

### Check for OOM killer activity

#### Kernel OOM / warnings check

**Attempted `dmesg` (non-root):**
```bash
dmesg -T | tail -n 50
```

**Result:**
```text
dmesg: read kernel buffer failed: Operation not permitted
```

❌ **`dmesg` was blocked as non-root**

**Fix path:**

If sudo is allowed, use:
```bash
sudo dmesg -T | tail -n 50
```

If sudo is not allowed (or dmesg is still restricted), use the journal instead:
```bash
journalctl -k --since "30 min ago" --no-pager \
  | egrep -i 'oom|out of memory|killed process|page allocation failure|hung task|oom-killer' \
  | tail -n 50
```

**Result in this run:**  
✅ **No OOM-killer or "killed process" lines found**

> 💡 **Key observation:** No OOM kill happened in this run. Memory pressure does not always trigger the OOM killer - it depends on how quickly memory is consumed and what else is running.

---

## Fix Phase (Recovery)

### How I stopped it

🛑 **Stopped the python memory hog with Ctrl+C in the terminal running it.**

---

### `free -h` after recovery

```bash
free -h
```

**Output:**
```text
              total        used        free      shared  buff/cache   available
Mem:           3.8Gi       317Mi       3.4Gi       1.0Mi       232Mi       3.5Gi
Swap:          3.8Gi          0B       3.8Gi
```

✅ **Memory returned to normal levels**  
✅ **Swap still unused (0B)**

---

### `uptime` after recovery

```bash
uptime
```

✅ **System returned to normal responsiveness**

**Verify with `vmstat`:**
```bash
vmstat 1 5
```

**Observations:**
- `si/so` (swap in/out) = 0
- `id` (idle CPU) = ~99-100%

✅ **No swap activity, CPU idle**

---

## Takeaways

### What I Learned

1. 📊 **Memory pressure symptoms:**
   - Available memory dropping significantly
   - Top process list changing (single process dominating %MEM)
   - Commands may become slower to respond
   
2. 💾 **Swap behavior:**
   - Swap may stay at 0 (like my run) or may start climbing
   - If swap climbs fast, performance usually degrades significantly
   
3. 🔴 **OOM killer is not guaranteed:**
   - You can have memory pressure without the OOM killer firing
   - OOM typically only fires when the system is truly desperate for memory
   
4. 🎯 **Quickest diagnostic checks:**
   ```bash
   free -h                           # Overall memory state
   ps aux --sort=-%mem | head        # Top memory consumers
   top                               # Interactive view (press 'M' to sort by memory)
   journalctl -k | grep -i oom       # Check for OOM killer (when dmesg restricted)
   ```

---

## Troubleshooting Pattern

### Symptoms of memory pressure

| Symptom | What It Means | Severity |
|---------|---------------|----------|
| Available memory < 20% of total | System getting tight on memory | ⚠️ Warning |
| Swap usage climbing | System paging to disk | 🔴 Critical |
| Commands becoming slow | Memory contention | ⚠️ Warning |
| OOM killer messages in logs | System killing processes | 🔴 Critical |
| Single process using >50% memory | Likely memory leak or runaway process | ⚠️ Warning |

---

### Quick diagnostic commands

```bash
# Check overall memory state
free -h

# Check top memory consumers
ps aux --sort=-%mem | head -n 15

# Interactive monitoring (sort by memory with 'M')
top

# Check swap activity
swapon --show
vmstat 1 5    # Watch si/so columns for swap in/out

# Check for OOM killer activity
sudo journalctl -k | grep -i oom
sudo journalctl -k | grep -i "killed process"

# If you have root access, check dmesg
sudo dmesg | tail -n 50
```

---

### Memory states explained

```text
              total        used        free      shared  buff/cache   available
Mem:           3.8Gi       2.3Gi       1.5Gi       1.0Mi       232Mi       1.5Gi
```

- **total:** Total physical RAM
- **used:** Memory currently in use by processes
- **free:** Unused memory
- **buff/cache:** Memory used for disk caching (can be reclaimed if needed)
- **available:** Memory available for new applications (includes reclaimable cache)

> 💡 **Important:** Look at "available" not "free" - the kernel uses free memory for caching, which is reclaimable.

---

## Notes

- ⚠️ **Prevention in production:**
  - Set up memory usage alerts (80-85% threshold)
  - Monitor for memory leaks in applications
  - Configure appropriate swap space
  - Use systemd resource limits for critical services

- 🔍 **Identifying memory leaks:**
  ```bash
  # Watch a specific process over time
  watch -n 5 'ps aux | grep <process-name>'
  
  # Monitor total system memory trends
  sar -r 1 10    # If sysstat is installed
  ```

- 🛠️ **Emergency response:**
  1. Identify the memory hog: `ps aux --sort=-%mem | head`
  2. Check if it's critical or can be killed
  3. Kill gracefully first: `kill <PID>`
  4. Force kill if needed: `kill -9 <PID>`
  5. Investigate why it happened (logs, application issues)
