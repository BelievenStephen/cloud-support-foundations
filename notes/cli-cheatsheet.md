# CLI Troubleshooting Cheatsheet

Use the **flow** first. Then run the matching commands.

---

## Quick triage flow

1. **Do I have network + a default route?**
2. **Is DNS working?**
3. **Is the service listening on the port?**
4. **Can I reach the service (HTTP)?**
5. **What do logs say?**

---

## Baseline checks

### Identify the host and time

```bash
date
hostname
```

### IP + interface state

```bash
ip a
```

**Look for:**

- Correct interface is `UP`
- An IP address is assigned (ex: `192.168.x.x`)

### Routing (gateway)

```bash
ip route
```

**Look for:**

- A `default via <gateway> dev <iface>` line exists

### Basic internet reachability (no DNS)

```bash
ping -c 3 1.1.1.1
```

**Interpretation:**

- ✅ Works = network path likely OK
- ❌ Fails = routing/gateway/firewall/host network issue (fix this before DNS)

---

## DNS checks (name resolution)

### Quick DNS lookup

```bash
dig example.com +short
```

**Interpretation:**

- IPs returned = DNS working
- Timeouts / "no servers could be reached" = DNS problem

### Resolver configuration and status (Ubuntu)

```bash
ls -l /etc/resolv.conf
cat /etc/resolv.conf
resolvectl status
```

**Look for:**

- `/etc/resolv.conf` points to systemd-resolved (often `nameserver 127.0.0.53`)
- `resolvectl status` shows DNS servers and the active interface

---

## Web reachability (HTTP/HTTPS)

### Check HTTPS response headers

```bash
curl -I https://example.com
```

**Interpretation:**

- HTTP status line returned (200/301/302, etc.) = DNS + network + HTTPS path working
- "Could not resolve host" = DNS issue
- Timeout = routing/firewall/path issue

---

## SSH / Port listening checks (on the server)

### Is SSH service running?

```bash
sudo systemctl status ssh --no-pager
```

**Look for:**

- `Active: active (running)`

### Is port 22 listening?

```bash
ss -tulpn | grep :22 || echo "Port 22 not listening"
```

**Interpretation:**

- `LISTEN` line = sshd is listening
- No output + "Port 22 not listening" = service down or not bound

### SSH service logs (last 50 lines)

```bash
sudo journalctl -u ssh --no-pager | tail -n 50
```

**Look for:**

- Startup failures
- Bind errors
- Auth failures

---

## Common error patterns (fast mapping)

### DNS-style failures

```
curl: (6) Could not resolve host: ...
ping: ... Name or service not known
```

**What to run:**

```bash
dig example.com +short
resolvectl status
cat /etc/resolv.conf
```

### Routing / gateway failures ("network down")

```
ping 1.1.1.1 fails
HTTP requests time out even if DNS works
```

**What to run:**

```bash
ip a
ip route
ping -c 3 1.1.1.1
```

### "Connection refused" (service/port closed)

Error happens immediately

**What to run (on server):**

```bash
sudo systemctl status ssh --no-pager
ss -tulpn | grep :22
```

### "Operation timed out" (traffic dropped/blocked)

Client waits then fails

**What to check:**

- routing (default gateway)
- firewall/security rules (later)

**Start with:**

```bash
ip route
ping -c 3 1.1.1.1
sudo ufw status verbose
sudo iptables -L INPUT -n --line-numbers
```

---

## Minimal "daily reps" set (10 commands)

Run these after you SSH into the VM:

```bash
ip a
ip route
ping -c 3 1.1.1.1
dig example.com +short
resolvectl status
curl -I https://example.com
sudo systemctl status ssh --no-pager
ss -tulpn | grep :22
sudo journalctl -u ssh --no-pager | tail -n 50
cat /etc/resolv.conf
```
