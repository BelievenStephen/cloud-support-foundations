# SSH Break/Fix Drill 01

## Goal

Practice diagnosing and restoring SSH access by intentionally breaking the SSH service and verifying the symptom from the client side (Mac), then fixing it from the server side (VM console).

## Environment

- **Client:** macOS Terminal 
- **Server:** Ubuntu Server VM in UTM
- **Network:** UTM NAT
- **VM IP:** `<vm-ip>`
- **Host IP:** `<host-ip>`

> **Important idea:** SSH requires a server process on the VM to listen on TCP port 22.

---

## Key Concept

SSH has two parts:

1. **SSH client** on my Mac: the `ssh` command I run
2. **SSH server** on the VM: the `sshd` service that listens on port 22 and accepts logins

If the server is not listening on port 22, my Mac cannot connect.

---

## Before the Break (Baseline)

I confirmed SSH was working.

**From Mac:**
```bash
ssh stephen@<vm-ip>
```
✅ Connected successfully

**On VM:**
```bash
systemctl status ssh
```
✅ Showed the service was `active (running)`

```bash
ss -tulpn | grep :22
```
✅ Showed the VM was listening on port 22

**What this means:** The SSH server (`sshd`) was running and reachable.

---

## Break Phase (Make SSH Unavailable on Purpose)

### Step 1: Stop the SSH server

**On the VM I ran:**
```bash
sudo systemctl stop ssh
```

**What this does:**
- Stops the `ssh` service, which normally runs the `sshd` process
- This should remove the listener on port 22

---

### Step 2: I saw a message about ssh.socket

**I saw something like:**
```
Stopping ssh.service, but its triggering units are still active: ssh.socket
```

**What this means:**
- Ubuntu can use a feature called **socket activation**
- `ssh.socket` can be configured to manage the port and trigger the SSH service when someone connects
- Even if `ssh` is stopped, `ssh.socket` might still be active and could cause SSH to come back or keep port behavior confusing

---

### Step 3: Stop the socket too (to fully break it)

**On the VM I ran:**
```bash
sudo systemctl stop ssh.socket
```

**What this does:**
- Stops the socket unit so it will not manage or trigger SSH
- Prevents SSH from auto-starting when a connection attempt happens

---

### Step 4: Confirm the server is actually down

**On the VM I checked:**
```bash
sudo systemctl status ssh --no-pager
sudo systemctl status ssh.socket --no-pager
ss -tulpn | grep :22 || echo "Port 22 not listening"
```

**What the results mean:**
- `Active: inactive (dead)` for `ssh` means `sshd` is not running
- `Active: inactive (dead)` for `ssh.socket` means the socket is not active
- `Port 22 not listening` means nothing on the VM is accepting connections on port 22

> **Important note about "enabled":**  
> - `enabled` means "start at boot"
> - It does NOT mean "running right now"
> - The real "is it running" indicator is `Active:` and whether port 22 is listening

---

## Client-Side Test (Prove It Failed from the Mac)

### Step 5: Try to SSH from my Mac

**On my Mac I ran:**
```bash
ssh stephen@<vm-ip>
```

**I got:**
```
ssh: connect to host <vm-ip> port 22: Connection refused
```

**What "Connection refused" means:**
- My Mac reached the VM over the network
- The VM responded "no service is listening on that port"
- This matches my server-side check that port 22 was not listening

✅ **This is the expected failure mode for this drill.**

---

## Fix Phase (Restore SSH Access)

### Step 6: Start the socket and service again

**On the VM I ran:**
```bash
sudo systemctl start ssh.socket
sudo systemctl start ssh
```

**What this does:**
- Starts socket activation (if used) and/or starts `sshd`
- Brings back the listener on port 22
- Starting `ssh` brings `sshd` up immediately. Starting `ssh.socket` controls socket activation behavior if the system uses it.

---

### Step 7: Confirm port 22 is listening again

**On the VM I ran:**
```bash
ss -tulpn | grep :22
```

**I saw lines like:**
```
0.0.0.0:22
[::]:22
```

**What this means:**
- SSH is listening on port 22 for IPv4 (`0.0.0.0`) and IPv6 (`[::]`)
- The server should be reachable again

---

### Step 8: Try SSH from the Mac again

**On my Mac I ran:**
```bash
ssh stephen@<vm-ip>
```

✅ **This time it connected and prompted for a password.**

**What this proves:**
- The server is listening again
- The full path works: Mac client → network → VM server → login

---

## What I Learned (The Troubleshooting Pattern)

This drill is a real "cloud support" workflow:

### 1. Symptom from client
"SSH won't connect"

**Error message matters:**
- `Connection refused` usually means "port reachable but nothing listening"
- `Connection timed out` usually means "network/firewall dropping traffic"

### 2. Check service state on server
```bash
systemctl status ssh
systemctl status ssh.socket  # if socket activation exists
```

### 3. Check listening ports
```bash
ss -tulpn | grep :22
```

### 4. Fix
- Start services (`systemctl start ...`)
- Re-check port listening
- Re-test from client

---

## Notes and Small Gotchas

- If you stop `ssh` but not `ssh.socket`, SSH can behave unexpectedly depending on configuration
- **"enabled" is not the same as "active"**
- If I am connected via SSH and stop `sshd`, my existing session may stay open for a bit. Stopping the service mostly prevents **new** connections. That is why testing from a new Mac terminal window is best.
