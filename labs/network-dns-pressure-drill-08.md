# Network/DNS Pressure Drill 08 (DNS vs Routing Triage)

## Goal

Practice a fast "can't reach site" triage. Prove whether the issue is DNS resolution or routing/connectivity.

---

## Environment

- **Host:** Ubuntu VM (`stephen-lab`) on UTM (NAT)
- **Date/time:** Thu Feb 12, 2026 (UTC)
- **Interface:** `enp0s1`
- **Local gateway/route:** `default via <gateway-ip> dev enp0s1`
- **Resolver at start:** DNS server `<gateway-ip>` (and link-local IPv6)

---

## Baseline (Prove Name and IP Paths Work)

### Commands I ran and what they showed

**Check DNS resolver configuration:**
```bash
resolvectl status
```

**Observation:**  
✅ Confirmed systemd-resolved in use and DNS server configured for `enp0s1`

---

**Test DNS resolution:**
```bash
dig example.com +short
```

**Output:**
```text
104.18.27.120
104.18.26.120
```

✅ **DNS resolution working**

---

**Verify local resolver:**
```bash
resolvectl query example.com
```

**Observation:**  
✅ Returned A/AAAA answers (local resolver working)

---

**Check routing table:**
```bash
ip route
```

**Output:**
```text
default via <gateway-ip> dev enp0s1 proto dhcp src <vm-ip> metric 100
<subnet>/24 dev enp0s1 proto kernel scope link src <vm-ip> metric 100
```

✅ **Default route configured correctly**

---

**Test basic connectivity (no DNS needed):**
```bash
ping -c 2 1.1.1.1
```

**Output:**
```text
PING 1.1.1.1 (1.1.1.1) 56(84) bytes of data.
64 bytes from 1.1.1.1: icmp_seq=1 ttl=53 time=X ms
64 bytes from 1.1.1.1: icmp_seq=2 ttl=53 time=X ms

--- 1.1.1.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss
```

✅ **Basic connectivity/routing working**

---

**Test name-based HTTPS:**
```bash
curl -I https://example.com
```

**Output:**
```text
HTTP/2 200
date: Thu, 12 Feb 2026 XX:XX:XX GMT
content-type: text/html
server: cloudflare
```

✅ **Name-based HTTPS working**

---

### IP vs name test

**First attempt (incorrect - demonstrates common mistake):**
```bash
EX_IP=$(dig example.com +short | head -n 1)
curl -I "https://$EX_IP"
```

**Result:**  
❌ Failed with TLS handshake error (expected - HTTPS to IP often breaks SNI/cert validation)

---

**Correct test (bypassing DNS while preserving hostname):**
```bash
EX_IP=$(dig example.com +short | head -n 1)
curl -I --resolve "example.com:443:$EX_IP" https://example.com
```

**Output:**
```text
HTTP/2 200
```

✅ **Proved IP path works while keeping hostname/SNI correct**

> 💡 **Key lesson:** Use `--resolve` to test connectivity to an IP while maintaining proper TLS/SNI behavior.

---

## Break Phase (DNS-Only Failure, Reversible)

### Simulate DNS failure

I simulated a DNS failure by overriding `/etc/resolv.conf` to point to a bad DNS server.

**Check current state:**
```bash
ls -l /etc/resolv.conf
cat /etc/resolv.conf
```

---

**Back up current configuration:**
```bash
sudo cp /etc/resolv.conf /etc/resolv.conf.bak
```

---

**Point resolver to bad server:**
```bash
printf "nameserver 203.0.113.1\noptions timeout:1 attempts:1\n" | sudo tee /etc/resolv.conf
```

> ⚠️ **What this does:** Points DNS to `203.0.113.1` (a documentation-only IP that won't respond), breaking local DNS resolution while leaving routing intact.

---

## Observe (Decision Tree Evidence)

### DNS failure symptoms

**Test DNS resolution:**
```bash
dig example.com +short
```

**Result:**
```text
;; connection timed out; no servers could be reached
```

❌ **Local resolver path broken**

---

**Test with curl:**
```bash
curl -I https://example.com
```

**Error:**
```text
curl: (6) Could not resolve host: example.com
```

❌ **Name resolution failed**

---

### Proved routing/connectivity still OK

**Test basic connectivity:**
```bash
ping -c 2 1.1.1.1
```

**Output:**
```text
PING 1.1.1.1 (1.1.1.1) 56(84) bytes of data.
64 bytes from 1.1.1.1: icmp_seq=1 ttl=53 time=X ms
64 bytes from 1.1.1.1: icmp_seq=2 ttl=53 time=X ms

--- 1.1.1.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss
```

✅ **Network path is fine - not a routing outage**

---

**Bypass local resolver (query upstream DNS directly):**
```bash
dig @1.1.1.1 example.com +short
```

**Output:**
```text
104.18.27.120
104.18.26.120
```

✅ **Upstream DNS works - local DNS config is the issue**

---

**Prove HTTP works when DNS is bypassed:**
```bash
EX_IP=$(dig @1.1.1.1 example.com +short | head -n 1)
curl -I --resolve "example.com:443:$EX_IP" https://example.com
```

**Output:**
```text
HTTP/2 200
```

✅ **Routing OK - can reach destination when DNS is bypassed**

---

### Decision tree conclusion

| Symptom | Root Cause |
|---------|------------|
| ✅ Ping to IP works<br>❌ Name lookups fail | DNS/config/resolver issue |
| ❌ Ping to IP fails<br>❌ Name lookups fail | Routing, firewall, or upstream connectivity issue |

---

## Fix Phase (Revert and Verify)

### Restore DNS configuration

**Restore resolver config:**
```bash
sudo mv /etc/resolv.conf.bak /etc/resolv.conf
sudo systemctl start systemd-resolved
sudo resolvectl flush-caches
```

---

### Verification

**Test DNS resolution:**
```bash
dig example.com +short
```

**Output:**
```text
104.18.27.120
104.18.26.120
```

✅ **DNS restored**

---

**Verify local resolver:**
```bash
resolvectl query example.com
```

✅ **Local resolver restored**

---

**Test name-based HTTPS:**
```bash
curl -I https://example.com
```

**Output:**
```text
HTTP/2 200
```

✅ **Name-based access restored**

---

## Takeaways (DNS vs Routing Logic)

### Fast triage checks

**DNS path:**
```bash
dig example.com +short                # Test local DNS resolution
resolvectl query example.com          # Verify systemd-resolved
cat /etc/resolv.conf                  # Check DNS server config
```

**Connectivity path:**
```bash
ping -c 2 1.1.1.1                     # Test basic connectivity (no DNS)
ip route                              # Verify routing table
```

---

### Simple decision logic

| Scenario | Diagnosis |
|----------|-----------|
| 📛 **Name fails, IP works** | DNS issue (resolver config/service) |
| 📛 **Name fails, IP fails** | Routing/connectivity/firewall issue |

**To prove upstream DNS is fine:**
```bash
dig @1.1.1.1 example.com +short
```

**To prove HTTP works without local DNS:**
```bash
curl --resolve "host:443:IP" https://host
```

---

### Common real-world symptoms

| Error Message | Likely Cause |
|---------------|--------------|
| `Temporary failure in name resolution` | DNS resolver down or misconfigured |
| `Could not resolve host` | DNS resolution failing |
| `no servers could be reached` | DNS server unreachable |

---

## Troubleshooting Pattern

### Quick diagnostic workflow

```bash
# Step 1: Test if it's DNS or connectivity
ping -c 2 1.1.1.1                     # Works? → Network OK
dig example.com +short                # Fails? → DNS issue

# Step 2: Verify upstream DNS works
dig @1.1.1.1 example.com +short       # Works? → Local DNS config issue
                                       # Fails? → Upstream DNS/network issue

# Step 3: Check local DNS configuration
cat /etc/resolv.conf                  # Verify nameserver entries
resolvectl status                     # Check systemd-resolved status

# Step 4: Test connectivity bypassing DNS
EX_IP=$(dig @1.1.1.1 example.com +short | head -n 1)
curl -I --resolve "example.com:443:$EX_IP" https://example.com
```

---

## Comparison: DNS Issues vs Routing Issues

| Aspect | DNS Issue | Routing Issue |
|--------|-----------|---------------|
| **Ping by IP** | ✅ Works | ❌ Fails |
| **Ping by name** | ❌ Fails | ❌ Fails |
| **dig example.com** | ❌ Fails or times out | ❌ Times out (if DNS is remote) |
| **dig @1.1.1.1 example.com** | ✅ Works (if internet OK) | ❌ Fails |
| **curl with --resolve** | ✅ Works | ❌ Fails |
| **Fix location** | /etc/resolv.conf, systemd-resolved | ip route, firewall, gateway |

---

## Advanced Debugging Commands

```bash
# Check if systemd-resolved is running
systemctl status systemd-resolved

# View detailed DNS resolution info
resolvectl status

# Flush DNS cache
sudo resolvectl flush-caches

# Test specific DNS server
dig @8.8.8.8 example.com

# Check if DNS port 53 is reachable
nc -zv 1.1.1.1 53

# Trace DNS query path
dig +trace example.com

# Check for DNS-related firewall rules
sudo iptables -L -n | grep -E '(53|domain)'

# Monitor DNS queries in real-time (if available)
sudo tcpdump -i any port 53
```

---

## Notes

- 🎯 **The golden rule:** If you can ping an IP but can't resolve names, it's DNS
- 🔍 **Always test both paths:** Name resolution AND direct IP connectivity
- 💡 **Use `--resolve` flag:** Allows testing IP connectivity while maintaining proper TLS/SNI
- ⚠️ **Common mistake:** Testing HTTPS directly to an IP fails due to certificate/SNI issues, not routing
- 📝 **In production:** Check DNS resolver config, systemd-resolved status, and upstream DNS before escalating
