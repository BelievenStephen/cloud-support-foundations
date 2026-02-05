# Runbook: DNS Issues (Name Resolution Failures)

## Goal

Restore name resolution when DNS lookups fail and apps show "could not resolve host".

This runbook helps tell the difference between:
- DNS is broken
- Network is down
- The destination site/service is down

---

## Scope / Assumptions

- Troubleshooting from a Linux host (VM or server)
- Shell access available (SSH or console)
- DNS may be handled by `systemd-resolved` (common on Ubuntu)

---

## Symptoms (What I See)

### A) DNS lookup fails

**Examples:**
- `dig` times out or says `no servers could be reached`
- `nslookup` fails

### B) App errors that point to DNS

**Examples:**
- `curl: (6) Could not resolve host: example.com`
- `ping: example.com: Name or service not known`
- `apt update` fails with "Temporary failure resolving …"

---

## Quick Triage (Do These in Order)

### 1) Confirm basic network works (no DNS needed)

```bash
ip a
ip route
ping -c 3 1.1.1.1
```

**Interpretation:**
- ❌ If `ping 1.1.1.1` fails, this is likely a **network/routing problem**, not DNS. Fix networking first.
- ✅ If ping succeeds, network layer is working → proceed to DNS checks

---

### 2) Check DNS resolution directly

```bash
dig example.com +short
```

**Interpretation:**
- ✅ IPs returned → DNS working for this test
- ❌ Timeouts / `no servers could be reached` → DNS path is broken

---

### 3) Confirm whether the problem is DNS or the destination site

```bash
curl -I https://example.com
```

**Interpretation:**
- If DNS fails, `curl` will say it cannot resolve the host
- If DNS works but the site is down, you may still get HTTP errors or timeouts later

---

## Server-Side Checks (Ubuntu)

### 4) Inspect resolver configuration

```bash
ls -l /etc/resolv.conf
cat /etc/resolv.conf
```

**What I'm looking for:**
- `nameserver 127.0.0.53` often means `systemd-resolved` stub resolver is in use

---

### 5) Check `systemd-resolved` status (if available)

```bash
resolvectl status
systemctl status systemd-resolved --no-pager
```

**Interpretation:**
- If `systemd-resolved` is not running, DNS may fail even if networking is fine

---

### 6) Identify upstream DNS servers

From `resolvectl status`, note:
- **DNS Servers**
- **Current DNS Server**
- **Link/interface used** (ex: `enp0s1`)

---

## Fix Actions (Choose Based on What's Wrong)

### Case 1: `systemd-resolved` is down

```bash
sudo systemctl restart systemd-resolved
sudo systemctl status systemd-resolved --no-pager
```

**Re-test:**
```bash
dig example.com +short
```

---

### Case 2: `/etc/resolv.conf` is wrong or overwritten

If you have a known-good backup:

```bash
sudo cp /etc/resolv.conf.bak /etc/resolv.conf
```

**Re-test:**
```bash
cat /etc/resolv.conf
dig example.com +short
```

---

### Case 3: Upstream DNS server is unreachable or incorrect

- If using DHCP, try renewing the lease (depends on setup)
- If static DNS is configured, correct the DNS server values

**Re-test:**
```bash
dig example.com +short
```

---

### Case 4: DNS works, but the app still fails

Try multiple domains to rule out a single-site issue:

```bash
dig example.com +short
dig google.com +short
```

**Interpretation:**  
If DNS works for other domains, the issue may be specific to that domain (authoritative DNS, propagation, or the domain itself).

---

## Verification (Confirm Recovery)

**Run:**
```bash
dig example.com +short
curl -I https://example.com
```

**Expected:**
- ✅ `dig` returns one or more IPs
- ✅ `curl -I` returns an HTTP status line (200/301/302 etc.)

---

## Common Failure Patterns (Fast Mapping)

| Error Message | Root Cause | What to Check |
|---------------|------------|---------------|
| `Could not resolve host` | DNS resolution problem | `dig`, `resolvectl status`, `/etc/resolv.conf` |
| `Connection refused` | Host reachable, service/port not listening | `systemctl status`, `ss -tulpn` |
| `Operation timed out` | Traffic dropped/blocked or routing issue | Firewall rules, security groups, routing |

---

## Notes

- 💡 DNS failure can look like "internet is down" because most apps start with a DNS lookup
- 🎯 Always test an IP directly (like `ping 1.1.1.1`) to separate network vs DNS quickly
- 🔍 When DNS works for some domains but not others, the problem is likely with the authoritative DNS or domain-specific configuration

---

## Quick Reference Commands

```bash
# Test network without DNS
ping -c 3 1.1.1.1

# Test DNS resolution
dig example.com +short

# Check DNS configuration
cat /etc/resolv.conf
resolvectl status

# Check systemd-resolved
systemctl status systemd-resolved --no-pager

# Restart DNS resolver
sudo systemctl restart systemd-resolved

# Test with curl
curl -I https://example.com
```
