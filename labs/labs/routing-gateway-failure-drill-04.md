# Routing / Gateway Failure Drill 04

## Goal

Learn how a routing/default gateway failure looks, and how it differs from DNS issues and firewall drops.

---

## Environment

- **Client:** macOS (SSH)
- **Server:** Ubuntu VM (UTM)
- **VM IP:** `192.168.64.1`
- **Interface:** `enp0s1`
- **Normal gateway:** `192.168.64.1`

---

## Baseline (Working)

### Commands

```bash
date
hostname
ip a
ip route
ping -c 3 1.1.1.1
curl -I https://example.com
dig example.com +short
```

### Expected

- ✅ Default route exists via `<gateway-ip>`
- ✅ Ping to `1.1.1.1` succeeds
- ✅ `curl -I` returns HTTP status (200/301/302 etc.)
- ✅ `dig` returns IPs

### Actual Output

**`ip route`:**
```text
default via <gateway-ip> dev enp0s1 proto dhcp src <vm-ip> metric 100 
192.168.64.0/24 dev enp0s1 proto kernel scope link src <vm-ip> metric 100 
<gateway-ip> dev enp0s1 proto dhcp scope link src <vm-ip> metric 100
```

**`ping -c 3 1.1.1.1`:**
```text
PING 1.1.1.1 (1.1.1.1) 56(84) bytes of data.
64 bytes from 1.1.1.1: icmp_seq=1 ttl=53 time=17.3 ms
64 bytes from 1.1.1.1: icmp_seq=2 ttl=53 time=14.1 ms
64 bytes from 1.1.1.1: icmp_seq=3 ttl=52 time=14.4 ms

--- 1.1.1.1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2006ms
rtt min/avg/max/mdev = 14.061/15.238/17.277/1.447 ms
```

**`curl -I https://example.com`:**
```text
HTTP/2 200 
date: Fri, 06 Feb 2026 17:39:44 GMT
content-type: text/html
cf-ray: 9c9c6dbb0d0e83d9-LAX
last-modified: Wed, 04 Feb 2026 05:22:19 GMT
allow: GET, HEAD
age: 3831
cf-cache-status: HIT
accept-ranges: bytes
server: cloudflare
```

**`dig example.com +short`:**
```text
104.18.27.120
104.18.26.120
```

---

## Break (Simulate Bad Default Gateway)

### What I'm Changing

Replace the default route with a **valid-but-wrong** gateway on the local subnet. This simulates a misconfiguration or gateway failure.

### Commands

```bash
sudo ip route replace default via 192.168.64.254 dev enp0s1
ip route
ping -c 3 1.1.1.1
curl -I https://example.com
dig example.com +short
```

### Expected

- ❌ `ip route` shows default via `<wrong-gateway-ip>`
- ❌ Ping to `1.1.1.1` fails (often `Destination Host Unreachable` or 100% loss)
- ❌ `curl` fails to connect (no route to internet)
- ⚠️ DNS may still return IPs (because DNS resolution can still happen locally/cached), but HTTPS still fails

### Actual Output

**`ip route`:**
```text
default via <wrong-gateway-ip> dev enp0s1 
default via <gateway-ip> dev enp0s1 proto dhcp src <vm-ip> metric 100 
192.168.64.0/24 dev enp0s1 proto kernel scope link src <vm-ip> metric 100 
<gateway-ip> dev enp0s1 proto dhcp scope link src <vm-ip> metric 100
```

> 💡 **Note:** Both default routes are visible. The route without a metric (wrong-gateway-ip) takes priority over the other route with metric 100.

**`ping -c 3 1.1.1.1`:**
```text
PING 1.1.1.1 (1.1.1.1) 56(84) bytes of data.
From <vm-ip> icmp_seq=1 Destination Host Unreachable
From <vm-ip> icmp_seq=2 Destination Host Unreachable
From <vm-ip> icmp_seq=3 Destination Host Unreachable

--- 1.1.1.1 ping statistics ---
3 packets transmitted, 0 received, +3 errors, 100% packet loss, time 2073ms
pipe 3
```

**`curl -I https://example.com`:**
```text
curl: (7) Failed to connect to example.com port 443 after 6191 ms: Couldn't connect to server
```

**`dig example.com +short`:**
```text
104.18.27.120
104.18.26.120
```

> 🔍 **Key observation:** DNS still worked (returned IPs), but I couldn't actually reach the destination because my gateway was wrong. This shows the difference between DNS resolution and actual routing.

---

## Fix (Restore Routing)

### Commands

```bash
sudo ip route del default via <wrong-gateway-ip> dev enp0s1
ip route
```

### Expected

- ✅ Bad default route is removed
- ✅ Default via `<gateway-ip>` remains (original DHCP route)

### Actual Output

**`ip route`:**
```text
default via <gateway-ip> dev enp0s1 proto dhcp src <vm-ip> metric 100 
192.168.64.0/24 dev enp0s1 proto kernel scope link src <vm-ip> metric 100 
<gateway-ip> dev enp0s1 proto dhcp scope link src <vm-ip> metric 100
```

---

## Verify Recovery

### Commands

```bash
ping -c 3 1.1.1.1
curl -I https://example.com
dig example.com +short
```

### Expected

✅ All succeed again.

### Actual Output

**Ping:**
```text
PING 1.1.1.1 (1.1.1.1) 56(84) bytes of data.
64 bytes from 1.1.1.1: icmp_seq=1 ttl=53 time=16.3 ms
64 bytes from 1.1.1.1: icmp_seq=2 ttl=53 time=13.3 ms
64 bytes from 1.1.1.1: icmp_seq=3 ttl=53 time=13.2 ms

--- 1.1.1.1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2010ms
rtt min/avg/max/mdev = 13.198/14.266/16.290/1.431 ms
```

**Curl:**
```text
HTTP/2 200 
date: Fri, 06 Feb 2026 17:54:08 GMT
content-type: text/html
cf-ray: 9c9c82d26ea40f27-LAX
last-modified: Wed, 04 Feb 2026 05:21:17 GMT
allow: GET, HEAD
age: 28
cf-cache-status: HIT
accept-ranges: bytes
server: cloudflare
```

**Dig:**
```text
104.18.26.120
104.18.27.120
```

---

## Takeaways

### What I Learned

- 🎯 **This failure breaks internet access even by IP**, unlike DNS-only failures
- 🎯 **This is not "firewall drop" behavior**. It fails because the host cannot route traffic off the subnet
- 🎯 **DNS can still work** while routing is broken (if using local/cached DNS or if the DNS server is on the local subnet)

### Quick Isolation Pattern

| Symptom | Likely Cause | What to Check |
|---------|-------------|---------------|
| `dig` fails but `ping 1.1.1.1` works | DNS issue | `resolvectl status`, `/etc/resolv.conf` |
| `ping 1.1.1.1` fails and default route is wrong/missing | Routing/gateway issue | `ip route`, gateway reachability |
| Service is listening but connections hang | Firewall drop | `iptables`, UFW, Security Groups |

### Troubleshooting Commands

```bash
# Check routing table
ip route

# Test connectivity without DNS
ping -c 3 1.1.1.1

# Test DNS resolution
dig example.com +short

# Test full HTTP path
curl -I https://example.com

# Check gateway reachability
ping -c 3 <gateway-ip>
```

---

## Notes

- 💡 When troubleshooting in the real world, always verify the default route first: `ip route | grep default`
