# Runbook: Can't SSH to a Linux Server

## Goal

Restore SSH access when an SSH connection fails. Identify whether the issue is:
- Wrong target (DNS/hostname/IP)
- Network path problem (routing/firewall)
- Port not open (security group/NACL/firewall)
- SSH service down (sshd not running / not listening)

## Scope / Assumptions

- **Client machine:** My Mac (running `ssh`)
- **Server machine:** Ubuntu VM or Linux host (running OpenSSH server)
- **Standard SSH port:** 22 (unless configured otherwise)

---

## Symptoms (what I see on the client)

### A) `Connection refused`

Example:
```
ssh: connect to host <ip> port 22: Connection refused
```

**Meaning:** I reached the host, but nothing is listening on port 22 (or a firewall actively rejected it). This usually means the host is reachable, but the SSH service/port is closed.


### B) `Connection timed out`

Example:
```
ssh: connect to host <ip> port 22: Operation timed out
```

**Meaning:** Traffic is not getting through (routing issue, firewall drop, security group/NACL, host down).

### C) `Could not resolve hostname`

Example:
```
ssh: Could not resolve hostname <name>: Name or service not known
```

**Meaning:** DNS/name resolution failed or hostname is wrong.

### D) Authentication errors

Examples:
- `Permission denied (publickey,password).`
- Keeps asking for password

**Meaning:** SSH is reachable, but credentials/keys/user are wrong.

---

## Quick Triage (client side)

### 1. Confirm I am using the right target and user

```bash
ssh <user>@<ip-or-hostname>
```

If using a hostname, test DNS:

```bash
dig <hostname> +short
```

### 2. Check basic reachability (IP layer)

```bash
ping -c 3 <ip>
```

**Notes:**
- Ping can be blocked, so failure here is not always definitive
- If ping works, host is likely reachable

### 3. Check if port 22 is reachable from my Mac

```bash
nc -vz <ip> 22
```

**Interpretation:**
- `succeeded` / `open` → port reachable
- `refused` → host reachable, port closed/not listening
- `timed out` → path/firewall is blocking or host down

**Optional (more detail):**

```bash
ssh -vvv <user>@<ip>
```

---

## Server-Side Checks (requires console access to the server)

If I still have access via VM console (UTM) or cloud console, check:

### 4. Confirm SSH service status

```bash
sudo systemctl status ssh --no-pager
```

**Expected healthy:**
- `Active: active (running)`

If inactive, start it:

```bash
sudo systemctl start ssh
```

Enable at boot (optional, but usually desired):

```bash
sudo systemctl enable ssh
```

### 5. Confirm port 22 is actually listening

```bash
ss -tulpn | grep :22 || echo "Port 22 not listening"
```

**Expected healthy:**
- `LISTEN` on `0.0.0.0:22` and/or `[::]:22`

**If not listening:**
- SSH service may be down
- SSH may be bound to a specific interface
- Firewall may be blocking (see next step)

### 6. Check host firewall rules (Ubuntu)

If UFW is used:

```bash
sudo ufw status verbose
```

Allow SSH if needed:

```bash
sudo ufw allow 22/tcp
```

### 7. Check IP address and default route

```bash
ip a
ip route
```

**Confirm:**
- Server has the expected IP
- Default route exists (for outbound needs)
- Interface is UP

### 8. Check recent SSH logs

```bash
sudo journalctl -u ssh --no-pager | tail -n 50
```

**Look for:**
- Bind failures
- Config errors
- Authentication errors

---

## Fix Actions (choose based on the failure)

### If DNS/hostname issue

- Use the correct IP
- Fix DNS resolver settings on the client/server if needed
- Confirm with: `dig <hostname> +short`

### If port 22 not listening (common in labs)

On server:

```bash
sudo systemctl start ssh
ss -tulpn | grep :22
```

### If firewall/security rules blocking

- On Ubuntu: allow 22/tcp (UFW)
- On cloud: check Security Groups/NACLs
- Re-test port from client: `nc -vz <ip> 22`

### If credentials issue

- Verify username
- Reset password (lab only) or fix SSH keys
- Check `/etc/ssh/sshd_config` for auth settings (advanced)

---

## Verification (confirm recovery)

From my Mac:

```bash
ssh <user>@<ip>
```

Then confirm on server:

```bash
whoami
hostname
```

---

## Notes

- Existing SSH sessions can stay open even after stopping sshd. Stopping SSH mainly blocks new connections.
- `enabled` means "start at boot". It does not mean "running now". Use `Active:` and `ss` output to confirm running/listening.
