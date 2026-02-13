# Service Down Drill 09 (systemd Basics)

## Goal

Practice handling a "site/service is down" ticket by proving:

1. Is the service running?
2. Is anything listening on the expected port?
3. Can a client reach it?
4. Is the failure a refusal (service down) or a timeout (network path issue)?

---

## Environment

- **Server:** Ubuntu VM
- **Access:** SSH
- **Lab folder:** `~/drills/service-down-09`
- **Timestamp:** Noted with `date` during session

---

## Baseline (Prove Service Is Up)

### Start test HTTP service

**Started a simple HTTP service (lab "service"):**
```bash
python3 -m http.server 8080
```

**Output:**
```text
Serving HTTP on 0.0.0.0 port 8080 (http://0.0.0.0:8080/) ...
```

---

### Client-side proof it works

```bash
curl -I http://127.0.0.1:8080
```

**Expected output:**
```text
HTTP/1.0 200 OK
Server: SimpleHTTP/0.6 Python/3.X.X
Date: ...
```

✅ **Service is responding**

---

### Confirm port is listening

```bash
ss -tulpn | grep 8080
```

**Expected output:**
```text
tcp   LISTEN 0      5            0.0.0.0:8080      0.0.0.0:*    users:(("python3",pid=XXXXX,fd=3))
```

✅ **Port 8080 is listening**

---

### Baseline verification note

I also checked a known "real" service (`ssh`) during baseline to confirm the VM's core service was healthy.

```bash
systemctl status ssh --no-pager
ss -tulpn | grep :22
```

✅ **SSH service confirmed healthy**

---

## Break Phase (Safe + Reversible)

### Stop the HTTP server

**Simulate "service down":**

In the server terminal, pressed `Ctrl+C` to stop the Python HTTP server.

---

### Confirm "service down" symptom from client side

```bash
curl -I --max-time 3 http://127.0.0.1:8080
```

**Result:**
```text
curl: (7) Failed to connect to 127.0.0.1 port 8080 after 0 ms: Couldn't connect to server
```

❌ **Service not reachable**

---

### Confirm nothing is listening on the port

```bash
ss -tulpn | grep 8080 || echo "8080 not listening"
```

**Output:**
```text
8080 not listening
```

❌ **No process listening on port 8080**

---

## Observe (Gather Evidence + Classify Failure)

### 1) "Service not listening" behavior

**Characteristic:** Local request fails immediately when nothing is listening.

```bash
curl -I --max-time 3 http://127.0.0.1:8080
```

**Result:**
```text
curl: (7) Failed to connect to 127.0.0.1 port 8080 after 0 ms: Couldn't connect to server
```

⚡ **Fast failure** (not a long wait)

---

### 2) "Timeout" behavior (network/path issue example)

**Test:** I tested a non-routable/unreachable IP to force a timeout

```bash
curl -I --max-time 3 http://10.255.255.1:8080
```

**Result:**
```text
curl: (28) Connection timed out after 3000 milliseconds
```

⏱️ **Timeout** (waited full 3 seconds before failing)

---

### 3) Confirm process is not running

```bash
ps aux | grep -E "python3 -m http\.server|http\.server" | grep -v grep || echo "http.server process not running"
```

**Output:**
```text
http.server process not running
```

❌ **Process confirmed not running**

---

### Decision takeaway

| Symptom | Likely Cause |
|---------|--------------|
| ⚡ **Fast failure** ("Connection refused") | Nothing listening (service down, wrong port, crashed) |
| ⏱️ **Timeout** ("Connection timed out") | Network path issue (routing, firewall, security group, NACL) |

---

## Fix Phase (Restore Service + Verify)

### Restart the service

```bash
python3 -m http.server 8080
```

**Output:**
```text
Serving HTTP on 0.0.0.0 port 8080 (http://0.0.0.0:8080/) ...
```

✅ **Service restarted**

---

### Verify port is listening again

```bash
ss -tulpn | grep 8080
```

**Expected output:**
```text
tcp   LISTEN 0      5            0.0.0.0:8080      0.0.0.0:*    users:(("python3",pid=XXXXX,fd=3))
```

✅ **Port 8080 listening**

---

### Verify client can reach it

```bash
curl -I --max-time 3 http://127.0.0.1:8080
```

**Expected output:**
```text
HTTP/1.0 200 OK
Server: SimpleHTTP/0.6 Python/3.X.X
```

✅ **Service responding**

---

### Verify process exists

```bash
ps aux | grep -E "python3 -m http\.server|http\.server" | grep -v grep
```

**Expected output:**
```text
user    XXXXX  ... python3 -m http.server 8080
```

✅ **Process running**

---

## Key Takeaways

### Always prove three things in order:

1. **Reachability** (client test: `curl`)
2. **Listening** (server test: `ss -tulpn`)
3. **Running process** (`ps` or `systemctl` for real services)

### Failure classification:

- **Connection refused / failed quickly** → Host is reachable but service not listening
- **Timed out** → Request not getting through network path (routing or filtering)

### Lab approach:

This drill used a Python HTTP server to safely simulate a service and validate the "service down" workflow end-to-end without affecting production services.

---

## Troubleshooting Pattern

### Quick diagnostic workflow

```bash
# Step 1: Test from client perspective
curl -I --max-time 3 http://<host>:<port>

# Step 2: Check if port is listening (on server)
ss -tulpn | grep :<port>

# Step 3: Check if process is running
ps aux | grep <service-name>

# For systemd services:
systemctl status <service-name> --no-pager
```

---

### Comparison: Refused vs Timeout

| Test | Connection Refused | Connection Timed Out |
|------|-------------------|---------------------|
| **curl result** | Fast failure (< 1 second) | Slow failure (hits timeout limit) |
| **Error code** | curl: (7) | curl: (28) |
| **Likely cause** | Service not listening | Network path blocked |
| **Next check** | `ss -tulpn`, `systemctl status` | Firewall, security groups, routing |

---

## Real-World Service Examples

### For systemd-managed services:

```bash
# Check if service is running
systemctl status nginx --no-pager
systemctl status apache2 --no-pager

# Check if service is enabled (starts at boot)
systemctl is-enabled nginx

# Check listening ports
ss -tulpn | grep nginx
ss -tulpn | grep apache2

# View service logs
journalctl -u nginx -n 50 --no-pager
journalctl -u apache2 -n 50 --no-pager
```

---

### Common service ports reference:

| Service | Default Port | Check Command |
|---------|--------------|---------------|
| HTTP | 80 | `ss -tulpn \| grep :80` |
| HTTPS | 443 | `ss -tulpn \| grep :443` |
| SSH | 22 | `ss -tulpn \| grep :22` |
| MySQL | 3306 | `ss -tulpn \| grep :3306` |
| PostgreSQL | 5432 | `ss -tulpn \| grep :5432` |
| Redis | 6379 | `ss -tulpn \| grep :6379` |

---

## Advanced Debugging Commands

```bash
# Check all listening TCP ports
ss -tulpn | grep LISTEN

# Check specific service with full details
systemctl status <service> --no-pager --full

# View recent service errors
journalctl -u <service> -p err --since "1 hour ago"

# Check if service binary exists
which <service-binary>

# Check service configuration syntax (nginx example)
nginx -t

# Check service configuration syntax (apache example)
apache2ctl configtest

# View service dependencies
systemctl list-dependencies <service>

# Check if firewall is blocking
sudo iptables -L -n | grep <port>
sudo ufw status | grep <port>
```

---

## Notes

- 🎯 **The "three-check" pattern:** Reachability → Listening → Running
- ⚡ **Fast failure** = service issue (usually fixable with service restart)
- ⏱️ **Slow timeout** = network issue (check firewall, routing, security groups)
- 🔍 **For production services:** Always check logs (`journalctl -u <service>`) for root cause
- 📝 **Service won't start?** Check:
  - Configuration file syntax
  - Port conflicts (another process using the port)
  - Permissions (can the service user access required files?)
  - Dependencies (are required services running?)
- 💡 **Pro tip:** Use `--max-time` with curl to avoid waiting forever on timeouts
