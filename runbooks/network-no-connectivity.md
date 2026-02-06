# Runbook: Network No Connectivity (Routing / Gateway Failure)

## Goal

Restore basic network connectivity when the host cannot reach anything off-subnet.  
This runbook focuses on routing/default gateway issues (not DNS or firewall rules).

---

## Symptoms (What I see)

- `ping 1.1.1.1` fails (even by IP)
- `curl -I https://example.com` fails to connect
- Errors may include:
  - `Destination Host Unreachable`
  - `Network is unreachable`
  - `No route to host`
  - Long timeouts to any external IP

---

## Quick Triage (Do in order)

### 1) Confirm the interface is up and has an IP

```bash
ip a
```

**Expected:**

- Interface (ex: `enp0s1`) is `UP`
- Has an IPv4 address (ex: `192.168.x.x/24`)

If no IP or interface is DOWN, fix link/DHCP first (see Fix Actions: Case 1).

---

### 2) Check routes (default gateway)

```bash
ip route
```

**What I'm looking for:**

- A default route like:
  - `default via <gateway-ip> dev <iface>`

If there is no default route, or it points to a wrong gateway, off-subnet traffic will fail.

---

### 3) Test connectivity by IP (bypasses DNS)

```bash
ping -c 3 1.1.1.1
```

**Interpretation:**

- ✅ Works: basic routing is probably fine. Move to DNS checks (see "Not this runbook" below).
- ❌ Fails: this is a network/routing problem. Continue.

---

### 4) Check reachability of the gateway (local next hop)

Replace with your gateway from `ip route` (often `.1` on the subnet):

```bash
ping -c 3 <gateway-ip>
```

**Interpretation:**

- ❌ Gateway ping fails: likely L2/L3 local issue (interface, subnet, ARP, hypervisor network).
- ✅ Gateway ping works: issue may be upstream routing/NAT, but local routing is still the first check.

---

### 5) Optional: identify why packets won't route

```bash
ip route get 1.1.1.1
```

**Expected:**

- Shows the chosen gateway/dev for that destination.

---

## Fix Actions (Choose the case)

### Case 1: Interface down or missing IP

Bring interface up and refresh DHCP (Ubuntu typically uses networkd/NetworkManager depending on setup).

Check link:

```bash
ip link show <iface>
```

Try DHCP renew (common with systemd-networkd):

```bash
sudo dhclient -r <iface>
sudo dhclient <iface>
```

Re-check:

```bash
ip a
ip route
ping -c 3 1.1.1.1
```

---

### Case 2: Default route missing

If you know the correct gateway:

```bash
sudo ip route add default via <gateway-ip> dev <iface>
```

Re-check:

```bash
ip route
ping -c 3 1.1.1.1
```

---

### Case 3: Default route points to the wrong gateway (common drill scenario)

View routes:

```bash
ip route
```

Remove the bad default route (example):

```bash
sudo ip route del default via <bad-gateway-ip> dev <iface>
```

If needed, add the correct one back:

```bash
sudo ip route add default via <good-gateway-ip> dev <iface>
```

Re-check:

```bash
ip route
ping -c 3 1.1.1.1
curl -I https://example.com
```

---

### Case 4: DNS looks "fine" but nothing connects

DNS may be returning cached results. Routing still has to work for real traffic.

Confirm with:

```bash
ping -c 3 1.1.1.1
curl -I https://example.com
```

If these fail, treat it as connectivity/routing first.

---

## Verification (Confirm recovery)

Run:

```bash
ip route
ping -c 3 1.1.1.1
curl -I https://example.com
dig example.com +short
```

**Expected:**

- Default route exists via the real gateway
- IP ping succeeds
- `curl -I` returns an HTTP status line
- `dig` returns IPs

---

## Not this runbook (fast mapping)

- If IP ping works but names fail → use `runbooks/dns-issues.md`
- If `Connection refused` or `Operation timed out` to a specific port → firewall/service issue (see firewall drill/runbook)

---

## Command quick list

```bash
ip a
ip route
ip route get 1.1.1.1
ping -c 3 <gateway-ip>
ping -c 3 1.1.1.1
curl -I https://example.com
dig example.com +short
sudo ip route del default via <bad-gateway-ip> dev <iface>
sudo ip route add default via <good-gateway-ip> dev <iface>
```
