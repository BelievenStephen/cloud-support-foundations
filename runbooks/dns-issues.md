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

---

---

## DNS Failure Classification

Understanding the type of DNS failure helps narrow down the root cause quickly.

### Three Main Failure Types

**1) NXDOMAIN (Name Does Not Exist)**
```bash
dig nonexistent.example.com +short
# Returns: (empty or NXDOMAIN status)
```

**Meaning:**
- The name/record does not exist
- Wrong zone or domain
- Delegation issue (NS records not configured)

**Common causes:**
- Typo in domain name
- Record not created yet
- Wrong hosted zone queried

---

**2) SERVFAIL (Server Failure)**
```bash
dig problematic.example.com +short
# Returns: (empty with SERVFAIL status)
```

**Meaning:**
- DNS server encountered an error processing the query
- DNSSEC validation failure
- Resolver or authoritative server misconfiguration

**Common causes:**
- DNSSEC signature mismatch
- Broken delegation chain
- Authoritative server unreachable or misconfigured

---

**3) Timeout (No Response)**
```bash
dig example.com +short
# Hangs or shows: ;; connection timed out; no servers could be reached
```

**Meaning:**
- DNS server unreachable
- Network blocking DNS traffic
- Firewall blocking UDP/TCP port 53

**Common causes:**
- Resolver IP wrong or server down
- Security group/firewall blocking port 53
- Network path broken to DNS server

---

## Authoritative DNS and Delegation Checks

When DNS resolution fails, verify the domain's authoritative configuration.

### Check Current DNS Answer
```bash
# Quick check (IP only)
dig example.com +short

# Full answer with TTL
dig example.com A +noall +answer
```

**Example output:**
```
example.com.    300    IN    A    <IP_ADDRESS>
```

**What to note:**
- **300** = TTL in seconds (how long resolvers cache this answer)
- **A** = Record type (IPv4 address)
- **IP_ADDRESS** = Current resolved value

---

### Check Name Server Delegation
```bash
dig example.com NS +noall +answer
```

**Example output:**
```
example.com.    172800    IN    NS    ns-123.awsdns-12.com.
example.com.    172800    IN    NS    ns-456.awsdns-45.org.
```

**What this tells you:**
- Lists authoritative name servers for the domain
- Verifies delegation is configured
- If empty, domain delegation may be broken

**TTL meaning:**
- **172800** seconds (48 hours) = how long resolvers cache the NS records
- High TTL on NS records is normal and recommended

---

## Compare Public Resolvers

Isolate whether the issue is with a specific resolver or the authoritative DNS.

### Test Multiple Resolvers
```bash
# Cloudflare DNS
dig example.com @1.1.1.1 +short

# Google DNS
dig example.com @8.8.8.8 +short

# Your local resolver (default)
dig example.com +short
```

### Interpretation

| Scenario | Meaning | Next Steps |
|----------|---------|------------|
| **All resolvers return same answer** | Authoritative DNS is consistent | If wrong, check hosted zone record |
| **Resolvers return different answers** | Propagation in progress or cache differences | Wait for TTL expiry, check authoritative source |
| **Local resolver fails, public works** | Local resolver issue | Check `/etc/resolv.conf`, `systemd-resolved` status |
| **All resolvers fail** | Authoritative DNS problem | Check hosted zone, NS records, delegation |

---

## Route 53-Specific Checks (If Using Route 53)

### 1) Verify Hosted Zone

**AWS Console → Route 53 → Hosted zones**

**Check:**
- Correct domain name
- Zone type (Public or Private)
- NS records match domain registrar delegation

---

### 2) Verify Record Configuration

**Hosted zones → Select zone → Records**

**Common issues:**

| Problem | Cause | Fix |
|---------|-------|-----|
| **NXDOMAIN** | Record doesn't exist | Create A/AAAA/CNAME record |
| **Wrong IP** | Record value incorrect | Update record value |
| **CNAME at apex fails** | CNAME not allowed at zone apex | Use ALIAS record instead |

---

### 3) Check Record Type (ALIAS vs CNAME)

**At zone apex (e.g., `example.com`):**
- ❌ **CNAME:** Not allowed at apex
- ✅ **ALIAS:** Route 53-specific, works at apex
- ✅ **A/AAAA:** Standard IP address records

**For subdomains (e.g., `www.example.com`):**
- ✅ **CNAME:** Points to another DNS name
- ✅ **ALIAS:** Points to AWS resource
- ✅ **A/AAAA:** Direct IP address

---

### 4) Consider TTL Strategy

**Current TTL:**
- Check record TTL in hosted zone
- Low TTL (60-300s) = faster propagation, more queries
- High TTL (3600s+) = slower propagation, fewer queries

**Best practice:**
- Lower TTL before planned DNS changes
- Raise TTL after changes stabilize

---

### 5) Private Hosted Zone Awareness

**If using private hosted zone:**

**Requirements:**
- Query must originate from associated VPC
- VPC DNS resolution enabled
- VPC DNS hostnames enabled

**Verification:**
```bash
# From EC2 instance in VPC
dig internal.example.com +short

# From outside VPC (will fail)
dig internal.example.com @8.8.8.8 +short
# Returns: (empty, NXDOMAIN)
```

**Console check:**
- Route 53 → Hosted zones → Select private zone
- VPCs tab → Verify VPC association

---

## DNS Cache Flushing (Optional)

Sometimes local cache can cause stale DNS results even after fixes.

### Linux (systemd-resolved)
```bash
# Flush systemd-resolved cache
sudo resolvectl flush-caches

# Verify cache cleared
resolvectl statistics
```

---

### macOS
```bash
# Flush DNS cache (macOS)
sudo dscacheutil -flushcache
sudo killall -HUP mDNSResponder

# Verify by re-testing resolution
dig example.com +short
```

**Note:** No confirmation message is displayed. Test resolution to verify.

---

### When to Flush Cache

**Flush cache if:**
- DNS was recently fixed but still showing old answer
- Testing DNS changes locally
- Switching between networks/VPNs

**Don't flush if:**
- Testing propagation (you want to see cached behavior)
- Diagnosing resolver issues (cache behavior is diagnostic info)

---

## Extended Troubleshooting Reference

### DNS Query Workflow
```
Application
    ↓
Local resolver cache (systemd-resolved, dscacheutil)
    ↓
Recursive resolver (ISP, 1.1.1.1, 8.8.8.8)
    ↓
Authoritative name servers (Route 53, etc.)
    ↓
DNS record value
```

**Failure can occur at any layer.**

---

### Diagnostic Command Summary
```bash
# Classify failure type
dig example.com A +noall +answer

# Check delegation
dig example.com NS +noall +answer

# Test specific resolver
dig example.com @1.1.1.1 +short
dig example.com @8.8.8.8 +short

# Check TTL and full answer
dig example.com A +noall +answer

# Flush local cache (Linux)
sudo resolvectl flush-caches

# Flush local cache (macOS)
sudo dscacheutil -flushcache && sudo killall -HUP mDNSResponder
```

---

## Key Takeaways

**DNS failure types:**
- NXDOMAIN = name/record missing
- SERVFAIL = server error (DNSSEC, misconfig)
- Timeout = network/firewall blocking

**Delegation verification:**
- Check NS records with `dig domain NS`
- Verify delegation matches registrar settings

**TTL awareness:**
- Controls cache duration
- High TTL = slower change propagation
- Check with `dig domain A +noall +answer`

**Resolver comparison:**
- Test multiple resolvers (1.1.1.1, 8.8.8.8)
- Isolates local resolver vs authoritative issues

**Route 53 specifics:**
- ALIAS at apex (not CNAME)
- Private zones only work in associated VPCs
- Verify record type and value in console
