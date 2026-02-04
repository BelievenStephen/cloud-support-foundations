# DNS Failure Drill 02

## Goal

Simulate a DNS failure on the Ubuntu VM, confirm the symptoms using basic tools, then restore DNS and verify recovery.

This drill teaches me how to tell the difference between:
- "The internet is down"
- "DNS is broken"
- "The website is down"

---

## Environment

- **Client:** macOS Terminal (SSH into VM)
- **Server:** Ubuntu Server VM in UTM
- **Network:** UTM NAT
- **VM hostname:** `<vm-hostname>`
- **VM IP:** `<vm-ip>`

---

## Baseline (confirm normal behavior)

### 1. Confirm SSH is listening (not part of DNS, but confirms access)

```bash
ss -tulpn | grep :22
```

**Expected:**
- Shows port 22 is listening (SSH is up)

### 2. Confirm DNS + HTTPS work

```bash
dig example.com +short
curl -I https://example.com
```

**Expected:**
- `dig` returns one or more IP addresses
- `curl -I` returns an HTTP status line (example: `HTTP/2 200`)

---

## Break (cause DNS failure on purpose)

### 3. Inspect current resolver config

```bash
ls -l /etc/resolv.conf
cat /etc/resolv.conf
```

**What I saw:**
- `/etc/resolv.conf` is managed by `systemd-resolved`
- It contained a resolver like: `nameserver 127.0.0.53`

**Meaning:**
- The VM is using a local stub resolver that forwards DNS requests upstream

### 4. Back up resolv.conf

```bash
sudo cp /etc/resolv.conf /etc/resolv.conf.bak
```

### 5. Replace resolv.conf with a bad DNS server

```bash
echo "nameserver 203.0.113.1" | sudo tee /etc/resolv.conf
```

**Why this breaks DNS:**
- `203.0.113.0/24` is a documentation/test range and is not a real reachable DNS server
- Now the VM tries to send DNS queries to a server that will not respond

---

## Observe the failure (confirm DNS is the issue)

### 6. Re-run the same tests

```bash
dig example.com
curl -I https://example.com
```

**What I saw (expected failure pattern):**
- `dig` showed timeouts and eventually `no servers could be reached`
- `curl` failed with: `Could not resolve host: example.com`

**What that means:**
- The VM still has network connectivity, but cannot translate domain names into IP addresses
- `curl` never reaches HTTPS because DNS fails first

---

## Fix (restore DNS)

### 7. Restore the original resolv.conf

```bash
sudo cp /etc/resolv.conf.bak /etc/resolv.conf
```

**Optional:** Confirm the file looks normal again

```bash
cat /etc/resolv.conf
```

### 8. Verify recovery

```bash
dig example.com +short
curl -I https://example.com
```

**Expected:**
- `dig` returns IPs again
- `curl -I` returns an HTTP status line again

---

## Key Takeaways

### DNS failure symptoms
- `dig`: timeouts, `no servers could be reached`
- `curl`: `Could not resolve host`

### This is different from other common failures
- **Connection refused** usually means the host is reachable but the port/service is not listening
- **Connection timed out** usually means routing/firewall/network path problems
- **Could not resolve host** points to DNS resolution problems

### Good troubleshooting habit
- Confirm baseline first
- Change one thing
- Observe the failure
- Restore and verify
