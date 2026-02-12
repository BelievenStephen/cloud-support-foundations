# Runbook: Can't Reach Site (DNS vs Routing Triage)

## Goal

Quickly determine whether a "site won't load" issue is caused by DNS resolution or routing/connectivity, then restore service.

---

## Symptoms (What the User Reports)

- Website won't load in browser
- `curl`/apps time out
- Name lookups fail or are intermittent
- Errors like:
  - `Could not resolve host`
  - `Temporary failure in name resolution`
  - `no servers could be reached`
  - Connection timeouts

---

## Fast Triage Decision Tree (Run in Order)

### 1) Is basic IP connectivity up?

**Check default route:**
```bash
ip route
```

**Ping a known public IP (bypasses DNS):**
```bash
ping -c 2 1.1.1.1
```

**Interpretation:**

- ❌ **Ping fails:** Likely routing/connectivity/firewall/upstream outage
  - → Go to "Routing/Connectivity Path" below
- ✅ **Ping works:** Connectivity is OK
  - → Continue to DNS checks

---

### 2) Does DNS resolve?

**Check configured DNS + interface view:**
```bash
resolvectl status
```

**Query via system resolver:**
```bash
resolvectl query example.com
```

**Query via `dig`:**
```bash
dig example.com +short
```

**Interpretation:**

- ❌ **`resolvectl query`/`dig` fail:** DNS path problem
  - → Continue to bypass test (Step 3)
- ✅ **DNS returns IPs:** DNS likely OK
  - → If site still fails, suspect TLS/proxy/app issue

---

### 3) Bypass local resolver (local vs upstream DNS)

**Query a known public resolver directly:**
```bash
dig @1.1.1.1 example.com +short
```

**Interpretation:**

- ❌ **Fails locally, ✅ works with `@1.1.1.1`:** Local resolver/config issue
  - Issue: systemd-resolved, `/etc/resolv.conf`, or interface DNS
  - → Go to "Fix Steps" below
  
- ❌ **Fails both ways:** Upstream DNS issue or network egress issue to DNS servers (less common)
  - → Treat like connectivity issue or try another resolver:
    ```bash
    dig @8.8.8.8 example.com +short
    ```

---

### 4) Name vs IP HTTP proof (avoid TLS confusion)

**Preferred "clean" test (keeps hostname/SNI correct):**
```bash
EX_IP=$(dig @1.1.1.1 example.com +short | head -n 1)
curl -I --resolve "example.com:443:$EX_IP" https://example.com
```

**Interpretation:**

- ✅ **`--resolve` works, ❌ normal `curl` fails:** DNS issue (not routing)
- ❌ **Both fail:** App/TLS/endpoint issue or routing issue

> ⚠️ **Note:** `curl -I https://<IP>` may fail due to TLS SNI/cert mismatch. Use `--resolve` to keep the test valid.

---

## Fix Steps (DNS-Path Focused)

### A) Fix systemd-resolved (common on Ubuntu)

**Restart resolver:**
```bash
sudo systemctl restart systemd-resolved
sudo resolvectl flush-caches
```

**Verify resolver status:**
```bash
resolvectl status
```

**Expected result:** DNS servers listed for active interface

---

### B) Fix `/etc/resolv.conf` (if it was edited or broken)

**Check where it points:**
```bash
ls -l /etc/resolv.conf
cat /etc/resolv.conf
```

**If you have a backup from a change, restore it:**
```bash
sudo mv /etc/resolv.conf.bak /etc/resolv.conf
sudo systemctl restart systemd-resolved
sudo resolvectl flush-caches
```

**Verify:**
```bash
cat /etc/resolv.conf
dig example.com +short
```

---

### C) Fix interface DNS (if DNS server is wrong)

**Identify interface and set DNS (example uses `enp0s1`):**
```bash
resolvectl dns enp0s1 1.1.1.1 8.8.8.8
resolvectl domain enp0s1 "~."
sudo systemctl restart systemd-resolved
sudo resolvectl flush-caches
```

> 💡 **Note:** If NetworkManager is managing DNS, you may need to adjust it there instead. The key is: restore a valid DNS server for the active interface.

**Verify:**
```bash
resolvectl status
dig example.com +short
```

---

## Routing/Connectivity Path (If Step 1 Fails)

If `ping -c 2 1.1.1.1` fails:

### 1) Confirm interface is up and has an IP

```bash
ip a
```

**Look for:**
- Interface state: `UP`
- IP address assigned

---

### 2) Confirm default route exists

```bash
ip route
```

**Look for:**
- `default via <gateway-ip> dev <interface>`

---

### 3) Check basic reachability to gateway

```bash
ping -c 2 <gateway-ip>
```

**Example:**
```bash
ping -c 2 192.168.64.1
```

---

### 4) Check firewall (if applicable)

```bash
sudo ufw status
```

**Or check iptables:**
```bash
sudo iptables -L -n
```

---

### Diagnosis:

| Symptom | Likely Cause |
|---------|--------------|
| Gateway ping fails | Local network/VM NAT issue |
| Gateway works, internet ping fails | Upstream/egress/firewall issue |

---

## Verify (Success Criteria)

**DNS returns IPs:**
```bash
dig example.com +short
resolvectl query example.com
```

**Expected:**
```text
104.18.27.120
104.18.26.120
```

---

**HTTP headers succeed:**
```bash
curl -I https://example.com
```

**Expected:**
```text
HTTP/2 200
```

---

**Optional proof with `--resolve`:**
```bash
EX_IP=$(dig example.com +short | head -n 1)
curl -I --resolve "example.com:443:$EX_IP" https://example.com
```

**Expected:**
```text
HTTP/2 200
```

✅ **All tests passing = issue resolved**

---

## Quick Reference Decision Matrix

| Test | Result | Diagnosis | Next Step |
|------|--------|-----------|-----------|
| `ping 1.1.1.1` | ❌ Fails | Routing/connectivity issue | Check ip route, gateway, firewall |
| `ping 1.1.1.1` | ✅ Works | Network OK | Continue to DNS tests |
| `dig example.com +short` | ❌ Fails | DNS issue | Test with `dig @1.1.1.1` |
| `dig @1.1.1.1 example.com +short` | ✅ Works | Local DNS config issue | Fix resolv.conf or systemd-resolved |
| `dig @1.1.1.1 example.com +short` | ❌ Fails | Upstream DNS or egress issue | Check firewall, try alternate resolver |
| `curl --resolve` works, normal curl fails | ✅/❌ | DNS resolution issue | Fix DNS configuration |

---

## Common Scenarios and Solutions

### Scenario 1: systemd-resolved is down

**Symptoms:**
- `resolvectl query` returns error
- `systemctl status systemd-resolved` shows inactive

**Fix:**
```bash
sudo systemctl start systemd-resolved
sudo systemctl enable systemd-resolved
sudo resolvectl flush-caches
```

---

### Scenario 2: Wrong DNS server in /etc/resolv.conf

**Symptoms:**
- `cat /etc/resolv.conf` shows bad nameserver
- Local DNS queries fail, but `dig @1.1.1.1` works

**Fix:**
```bash
# Restore proper DNS servers
printf "nameserver 1.1.1.1\nnameserver 8.8.8.8\n" | sudo tee /etc/resolv.conf
sudo systemctl restart systemd-resolved
```

---

### Scenario 3: Interface has no DNS servers configured

**Symptoms:**
- `resolvectl status` shows no DNS servers for interface
- DNS queries fail locally

**Fix:**
```bash
# Replace enp0s1 with your interface name
resolvectl dns enp0s1 1.1.1.1 8.8.8.8
sudo systemctl restart systemd-resolved
```

---

### Scenario 4: Default route is missing

**Symptoms:**
- `ping 1.1.1.1` fails
- `ip route` shows no default route

**Fix:**
```bash
# Replace with your actual gateway IP and interface
sudo ip route add default via 192.168.64.1 dev enp0s1
```

> ⚠️ **Note:** This fix is temporary and will be lost on reboot. For permanent fix, update network configuration files.

---

## Notes

- 🎯 **Use `ping` to a public IP to separate "DNS" from "network down"**
- ⚠️ **`curl https://<IP>` is not a clean test for HTTPS due to SNI/certs**
  - Prefer `curl --resolve` for accurate testing
- 📊 **If DNS is intermittent:**
  - Capture timestamps
  - Compare `resolvectl status` output over time
  - Check resolver reachability with `ping <dns-server-ip>`
- 🔍 **For persistent issues:**
  - Check `/var/log/syslog` or `journalctl -u systemd-resolved`
  - Monitor DNS queries: `sudo tcpdump -i any port 53`
- 💡 **Cloud-specific considerations:**
  - Check security group rules for port 53 (DNS)
  - Verify VPC DNS settings (if applicable)
  - Check if custom DNS servers are reachable from subnet
