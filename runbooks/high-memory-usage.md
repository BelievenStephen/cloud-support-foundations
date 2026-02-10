# Runbook: High Memory Usage

## Summary

Use this when the VM or server feels slow and you suspect memory pressure (RAM exhaustion and/or swap thrashing). Goal is to confirm quickly, stop the biggest offender, and verify recovery.

---

## Symptoms

- SSH is slow or laggy (keystrokes delayed)
- Commands take much longer than normal
- Load average climbs even when CPU does not look maxed
- Swap usage grows quickly (if swap exists)
- Processes get killed unexpectedly
- Logs may show OOM activity ("Out of memory", "Killed process …")

---

## Fast checks (do in this order)

### 1) Current memory and swap summary

```bash
free -h
```

**What I'm looking for:**

- Low available memory
- Swap used increasing (if swap exists)

### 2) Load average and general responsiveness

```bash
uptime
```

**What I'm looking for:**

- Load average is much higher than baseline for this box

### 3) Top memory consumers (non-interactive)

```bash
ps aux --sort=-%mem | head -n 15
```

**What I'm looking for:**

- One or two processes using most of RAM

### 4) Interactive view (optional but helpful)

```bash
top
```

- Press `M` to sort by memory
- Note the top offenders
- Press `q` to quit

### 5) Swap status (may be empty)

```bash
swapon --show
```

If it prints nothing, swap is not configured. That is fine.

---

## Confirm OOM activity (permission-aware)

### Option A (preferred if allowed)

```bash
sudo dmesg -T | tail -n 50
```

### Option B (if `dmesg` is blocked or sudo isn't available)

```bash
journalctl -k --since "30 min ago" --no-pager | tail -n 80
```

**What I'm looking for:**

- `Out of memory`
- `Killed process <pid> (<name>)`
- `oom-killer`

---

## Fix (stop the offender)

### 1) Identify the process PID

Start with:

```bash
ps aux --sort=-%mem | head -n 10
```

If I need to target a known command pattern:

```bash
pgrep -af <pattern>
```

### 2) Try graceful stop first

```bash
kill <PID>
```

Wait a few seconds and re-check:

```bash
ps -p <PID>
```

### 3) Force kill if it does not stop

```bash
kill -9 <PID>
```

---

## Verify recovery

Run:

```bash
free -h
uptime
ps aux --sort=-%mem | head -n 10
```

**Expected:**

- Available memory returns to a normal range
- Load average trends down over the next minute or two
- The offending process is gone

### Optional: check OOM messages again

```bash
sudo dmesg -T | tail -n 50 || journalctl -k --since "30 min ago" --no-pager | tail -n 80
```

---

## Prevention (light)

- Reduce or disable non-essential services on small VMs
- Avoid running multiple heavy tasks at once on low-RAM instances
- Add swap later (if appropriate for the workload)
- In AWS later: add CloudWatch alarms for memory (via agent) and instance health signals
