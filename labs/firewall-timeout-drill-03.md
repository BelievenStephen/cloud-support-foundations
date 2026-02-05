# Firewall Timeout Drill 03 (Refused vs Timed Out)

## Goal

Learn the difference between SSH failures that return **Connection refused** vs **Operation timed out**, and practice confirming the cause from both the client (Mac) and server (VM).

This drill teaches how to tell the difference between:
- Host reachable but **port/service not listening** (refused)
- Traffic being **dropped/blocked** (timed out)

---

## Environment

- **Client:** macOS Terminal
- **Server:** Ubuntu Server VM in UTM
- **Network:** UTM NAT
- **VM IP:** `<vm-ip>` 
- **SSH port:** 22

---

## Baseline (Prove SSH Works)

### On the VM (existing SSH session or UTM console)

```bash
sudo systemctl status ssh --no-pager
ss -tulpn | grep :22
```

**Expected:**
- `Active: active (running)`
- A `LISTEN` line for `:22` (example: `0.0.0.0:22` and/or `[::]:22`)

### On the Mac (new terminal)

```bash
ssh stephen@<vm-ip>
```

**Expected:**
- ✅ Prompts for password and connects

**Exit the test session:**
```bash
exit
```

---

## Refused Test (Host Reachable, Port Not Listening)

**What I'm simulating:** The host is reachable, but nothing is listening on port 22 because SSH is stopped.

### On the VM

**Stop SSH and socket activation:**
```bash
sudo systemctl stop ssh
sudo systemctl stop ssh.socket
```

**Confirm port 22 is not listening:**
```bash
ss -tulpn | grep :22 || echo "Port 22 not listening"
```

**Expected:**
- `Port 22 not listening`
- No `LISTEN` output for `:22`

### On the Mac (new terminal)

```bash
ssh stephen@<vm-ip>
```

**Expected:**
```
ssh: connect to host <vm-ip> port 22: Connection refused
```

> **Key observation:**  
> The error is **immediate** because the host responds right away that nothing is listening on that port.

---

## Timed Out Test (Traffic Dropped by Firewall)

**What I'm simulating:** SSH is running and listening, but firewall rules drop traffic. The client waits and then times out.

### On the VM

**Start SSH again (port should be listening):**
```bash
sudo systemctl start ssh
sudo systemctl start ssh.socket
ss -tulpn | grep :22
```

**Expected:**
- `LISTEN` on `:22`

**Check UFW:**
```bash
sudo ufw status verbose
```

**If UFW is inactive, add a temporary iptables DROP rule:**
```bash
sudo iptables -I INPUT -p tcp --dport 22 -j DROP
```

> **What this does:** Inserts a rule at the top of the INPUT chain that drops all TCP traffic destined for port 22.

> ⚠️ **WARNING:** This DROP rule blocks new inbound SSH traffic to port 22. Any current SSH session may keep working briefly or may hang/disconnect depending on traffic and keepalives. Be ready to switch to the UTM console to remove the rule.

> 💡 **Recovery path:** You'll need to use the **UTM console** to access the VM and remove the iptables rule. Do not run this command over SSH unless you're prepared to lose access and switch to the console.

### On the Mac (new terminal)

**Try to establish a new SSH connection:**
```bash
ssh stephen@<vm-ip>
```

**Expected:**
After a wait (~60-75 seconds), it fails with:
```
ssh: connect to host <vm-ip> port 22: Operation timed out
```

> **Key observation:**  
> SSH is listening, but packets are **dropped** by the firewall, so the client keeps trying until it times out. Unlike "Connection refused" (which is immediate), this takes time because the client doesn't receive any response.

---

## Fix Steps (Restore Access)

### On the VM

**Remove the iptables rule that was inserted at the top:**
```bash
sudo iptables -D INPUT 1
```

**Confirm the DROP rule is gone:**
```bash
sudo iptables -L INPUT --line-numbers -n
```

**Expected:**
- No `DROP` rule for `tcp dpt:22`

**Confirm status of SSH:**
```bash
sudo systemctl status ssh --no-pager
ss -tulpn | grep :22
```

### On the Mac

```bash
ssh stephen@<vm-ip>
```

**Expected:**
- ✅ Connects again (password prompt to connect)

---

## Takeaways

### Connection refused happened when:
- ✅ The host was reachable
- ❌ Port 22 was not listening (SSH service stopped)
- ⚡ The error was **immediate**

### Operation timed out happened when:
- ❓ The host may be reachable
- ✅ SSH was listening, but traffic was **dropped** (firewall rule)
- ⏱️ The client waited, then failed

---

## Practical Troubleshooting Pattern

### If I see `Connection refused`, I check:
- Is the service running?
  ```bash
  systemctl status ssh
  ```
- Is the port listening?
  ```bash
  ss -tulpn | grep :22
  ```

### If I see `Operation timed out`, I check:
- Firewall rules / security rules
  - UFW: `sudo ufw status verbose`
  - iptables: `sudo iptables -L INPUT -n`
  - Cloud: Security Groups / NACLs
- Network path issues
  - Routing
  - Host down
  - Upstream blocks

---

## Quick Reference

| Error | Meaning | Common Causes | Where to Look |
|-------|---------|---------------|---------------|
| **Connection refused** | Host responds: "nothing listening" | Service stopped, wrong port | `systemctl status`, `ss -tulpn` |
| **Operation timed out** | No response (packets dropped) | Firewall blocking, network issue | `iptables`, UFW, Security Groups, routing |
