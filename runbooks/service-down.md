# Runbook: Service Down (systemd)

## Goal

Restore service availability when a host-based service is down or unreachable. Prove:

1. Is the service running?
2. Is it listening on the expected port?
3. Can a client reach it?
4. Is this a **refusal** (service/port issue) or a **timeout** (network path issue)?

---

## Symptoms (What Users Report)

- "Site is down"
- HTTP **502/503** (often from a load balancer or reverse proxy)
- `curl: (7) Failed to connect ... Couldn't connect to server` (fast failure)
- "Connection refused"
- "Connection timed out" (hangs then fails)
- `systemctl status <service>` shows **inactive**, **failed**, or restart loop

---

## Inputs to Collect (Before Changes)

Gather this information before making any changes:

- **Service name:** (example: `nginx`, `apache2`, `ssh`, `myapp`)
- **Expected port:** (example: 80/443/8080)
- **Hostname/IP** of the server
- **Who is impacted:** One user vs all users
- **When it started:** Timestamp or "last X minutes"

---

## CloudWatch-First Triage (Before Touching the Host)

**Why check CloudWatch first:**
- Provides historical context (when issue started, what changed)
- Shows trends vs single data points
- Helps classify issue type before SSH access
- Avoids making changes without understanding the problem

**Critical:** Verify you're in the correct AWS region before investigating.

---

### 1) Check CloudWatch Alarms

**CloudWatch Console → Alarms → All alarms**

**Look for:**
- Alarms in **ALARM** state (red)
- Alarms in **INSUFFICIENT_DATA** state (gray)
- Recent state changes in alarm history

**For each alarm, verify:**
- **Metric + Dimension:** Confirm correct resource (e.g., InstanceId for EC2)
- **Threshold:** What value triggers alarm
- **Period:** Time window for evaluation
- **Datapoints to alarm:** How many breaches needed to trigger
- **Missing data treatment:** How alarm handles gaps in data
- **Alarm history:** When state changed, previous states

---

**Alarm state interpretation map:**

| Alarm State | Metric | Next Steps |
|------------|--------|------------|
| **ALARM** | `CPUUtilization` | High load/saturation path → See on-host CPU investigation |
| **ALARM** | `StatusCheckFailed` | Instance health issue → Check EC2 status checks |
| **ALARM** | Custom metric | Service-specific issue → Check application logs/metrics |
| **INSUFFICIENT_DATA** | Any metric | Metrics not arriving → Verify instance state, region, dimension |

---

### 2) Review Metrics (Trend vs Spike)

**CloudWatch Console → Metrics → All metrics → AWS/EC2 → Per-Instance Metrics**

**Key metrics to check:**
- `CPUUtilization`
- `NetworkIn` / `NetworkOut`
- `StatusCheckFailed`
- `StatusCheckFailed_Instance`
- `StatusCheckFailed_System`

**Time range selection:**
- Use range that covers incident window
- Typical ranges: 1h, 3h, 12h, 24h
- Look for when abnormal behavior started

---

**Metric pattern interpretation:**

| Pattern | Likely Cause | Investigation Path |
|---------|--------------|-------------------|
| **High CPU + High Network** | Traffic spike or busy service | Check access logs, scale if needed |
| **High CPU + Low Network** | Runaway process or internal job | SSH to host, identify top CPU processes |
| **Flatline / Gaps** | Instance stopped, wrong region, metric delay | Verify instance running, check region/dimension |
| **Gradual climb** | Memory leak or growing workload | Check memory, investigate application |
| **Sudden drop to zero** | Instance stopped/terminated | Check EC2 console for instance state |

---

### 3) Check CloudWatch Logs (If Configured)

**CloudWatch Console → Logs → Log groups**

**If log groups exist:**
- Filter by time range around incident
- Search for ERROR, WARN, FATAL patterns
- Look for application stack traces
- Check for configuration errors

**If no log groups:**
- EC2 instances do NOT send logs by default
- Requires CloudWatch agent installation or service integration
- Fall back to on-host log investigation (`journalctl`, `/var/log`)

---

### 4) Map CloudWatch Results to Next Steps

**Based on CloudWatch findings, choose investigation path:**

---

**A) No data / INSUFFICIENT_DATA**

**Checks:**
1. EC2 Console → Verify instance is running
2. CloudWatch alarm → Verify correct InstanceId dimension
3. CloudWatch alarm → Verify correct region
4. Check time range and missing data treatment

**Common causes:**
- Instance stopped or terminated
- Wrong dimension (old instance ID)
- Viewing wrong region
- Recent launch (metrics not yet available)

---

**B) CPU High (Above Threshold)**

**Immediate checks:**
- Review metric graph for sustained vs brief spike
- Check NetworkIn/Out for correlation
- Note exact time of spike

**Next steps:**
1. SSH to instance
2. Run `top` to identify processes
3. Run `ps aux --sort=-%cpu | head -20`
4. Consider scaling or load reduction if sustained

**Do NOT restart service yet** - identify root cause first

---

**C) Status Check Failed**

**EC2 Console → Instances → Status checks tab**

**Identify which check failed:**

| Check Type | Meaning | Resolution |
|-----------|---------|------------|
| **System check** | AWS infrastructure issue | Stop and start instance (migrates to new host) |
| **Instance check** | Guest OS or config issue | Investigate on host (logs, disk, memory) |
| **Both** | Multiple issues | Start with system check resolution |

**Status check troubleshooting:**
- System: Hardware, network, power (AWS responsibility)
- Instance: OS kernel, filesystem, networking config (your responsibility)

---

**D) Normal Metrics But Service Down**

**If CloudWatch shows healthy but service is down:**
- CloudWatch metrics are instance-level, not service-level
- Issue is likely application/service specific
- Proceed to "Fast Checks" section for service-level investigation

---

### 5) Verify Recovery (CloudWatch + Endpoint Test)

**After implementing fix, confirm recovery in order:**

---

**1) CloudWatch alarm returns to OK:**
```
CloudWatch → Alarms → Select alarm → Check current state
```
**Expected:** State changes from ALARM to OK

**Review alarm history:**
- Confirms state transition time
- Provides recovery timestamp

---

**2) Metrics show normal trend:**
```
CloudWatch → Metrics → Review metric graph
```
**Expected:** Metric returns to baseline or expected range

---

**3) Endpoint responds successfully:**
```bash
curl -sv --max-time 5 http://<server-ip-or-hostname>:<port> -o /dev/null
```
**Expected:** HTTP response headers (200/301/etc)

---

**4) Health check endpoint (if available):**
```bash
curl -sf http://<server-ip-or-hostname>:<port>/health && echo OK
```
**Expected:** Successful response with health status

---

**5) Service logs show no errors:**
```bash
sudo journalctl -u <service> --since "10 min ago" --no-pager | tail -n 50
```
**Expected:** No errors, warnings, or restart loops

---

## CloudWatch Investigation Tips

**Region verification:**
- CloudWatch is region-scoped
- Instance in us-west-1 won't show metrics in us-east-1 console
- Verify region selector in console matches instance region

**Metric delay awareness:**
- Standard monitoring: 5-minute intervals
- Detailed monitoring: 1-minute intervals
- Allow 5-10 minutes for new datapoints to appear

**Alarm history usage:**
- Shows when alarm changed state
- Provides timeline of incident
- Useful for correlating with changes/deployments

**Missing data vs zero:**
- Missing data = no datapoints (instance stopped, metric not publishing)
- Zero value = datapoint received with value of 0
- Check "Missing data treatment" setting in alarm

---

## Fast Checks (Do in Order)

### 1) Confirm the service state (systemd)

```bash
sudo systemctl status <service> --no-pager
sudo systemctl is-enabled <service>
```

**Interpretation:**

| Status | Meaning |
|--------|---------|
| `active (running)` | Service is up from systemd's view |
| `inactive (dead)` | Service is down or stopped |
| `failed` | Service crashed or failed to start |
| "restarting" / crash loop | Likely config/runtime error |

---

### 2) Confirm something is listening on the expected port

```bash
sudo ss -tulpn | grep -E ':(<port>)\b' || echo "port not listening"
```

**Interpretation:**

- ❌ **Port not listening** → Service is not bound to it
  - Possible causes: service down, misconfigured, wrong port, failed startup

---

### 3) Check recent logs for the service

```bash
sudo journalctl -u <service> --since "30 min ago" --no-pager | tail -n 120
```

**Look for:**

- Start/stop events
- Config parse errors
- Permission errors (files/ports)
- "Address already in use"
- Missing env vars or dependencies
- Out of memory (OOM) kills

---

### 4) Client-side test (local and remote)

**Local test from the host:**

```bash
curl -sv --max-time 3 http://127.0.0.1:<port> -o /dev/null
```

**If HTTPS:**

```bash
curl -skv --max-time 3 https://127.0.0.1:<port> -o /dev/null
```

**Interpretation:**

| Result | Likely Cause |
|--------|--------------|
| ⚡ **Fast failure** ("Failed to connect", "Connection refused") | Nothing is listening (service down/wrong port) |
| ⏱️ **Timeout** | Network path/filtering issue, OR service hung and not accepting connections |

---

## Quick Decision Tree

### A) `systemctl` shows failed/inactive OR port not listening

**Diagnosis:** Likely a **service problem**

- Service down, crashed, or config issue
- → Go to "Fix Steps (Service Path)" below

---

### B) Service is active AND port is listening, but curl times out

**Diagnosis:** Likely a **network path** or filtering problem

- Could be: host firewall, security groups, NACLs, routing
- → Check firewall rules, security groups, routing

---

### C) Service is active and listening, but returns 502/503

**Diagnosis:** Often an **upstream dependency** issue

- Backend down, bad upstream config, health checks failing
- → Check backend services, upstream connections, health check endpoints

---

## Fix Steps (Service Path)

### 1) Restart and re-check

```bash
sudo systemctl restart <service>
sudo systemctl status <service> --no-pager
```

**If restart fails, immediately view logs:**

```bash
sudo journalctl -u <service> --since "10 min ago" --no-pager | tail -n 200
```

---

### 2) Validate configuration (if applicable)

**Common examples:**

**Nginx:**
```bash
sudo nginx -t
```

**Apache:**
```bash
sudo apachectl configtest
```

**systemd unit:**
```bash
sudo systemctl cat <service>
```

**If config is invalid:**

1. Fix the configuration file
2. Restart the service:
   ```bash
   sudo systemctl restart <service>
   ```

---

### 3) Check if the port is already in use

```bash
sudo ss -tulpn | grep -E ':(<port>)\b'
sudo lsof -i :<port> 2>/dev/null || true
```

**If another process owns the port:**

- Option 1: Stop the conflicting process
- Option 2: Change service to use different port

---

### 4) Confirm dependencies (common causes)

**Disk full:**
```bash
df -h
```

**Permissions/ownership errors:**
- Check in `journalctl` output
- Verify service user can access required files/directories

**Missing env vars/secrets:**
- Shows in `journalctl`
- Verify environment file exists and is readable

**Out of memory / OOM kills:**
```bash
dmesg | tail -n 50
```

---

## Verify (Must Pass All)

### 1) Service healthy

```bash
sudo systemctl status <service> --no-pager
```

**Expected:** `active (running)`

---

### 2) Port is listening

```bash
sudo ss -tulpn | grep -E ':(<port>)\b'
```

**Expected:** `LISTEN` on the expected port

---

### 3) Local curl works

```bash
curl -sv --max-time 3 http://127.0.0.1:<port> -o /dev/null
```

**Expected:** HTTP response headers (200/301/401/etc depending on service)

---

### 4) External reachability (if applicable)

**From a client host:**
```bash
curl -sv --max-time 5 http://<server-ip-or-hostname>:<port> -o /dev/null
```

**Expected:** Successful connection and HTTP response

---

## Common Service-Specific Commands

### Nginx

```bash
# Check configuration
sudo nginx -t

# View error log
sudo tail -n 50 /var/log/nginx/error.log

# Restart service
sudo systemctl restart nginx

# Check listening ports
sudo ss -tulpn | grep nginx
```

---

### Apache2

```bash
# Check configuration
sudo apachectl configtest

# View error log
sudo tail -n 50 /var/log/apache2/error.log

# Restart service
sudo systemctl restart apache2

# Check listening ports
sudo ss -tulpn | grep apache2
```

---

### SSH

```bash
# Check status
sudo systemctl status ssh --no-pager

# View logs
sudo journalctl -u ssh --since "1 hour ago" --no-pager

# Check listening on port 22
sudo ss -tulpn | grep :22

# Test locally
ssh localhost
```

---

### Custom Application Services

```bash
# Check status
sudo systemctl status myapp --no-pager

# View recent logs
sudo journalctl -u myapp -n 100 --no-pager

# View all logs for today
sudo journalctl -u myapp --since today --no-pager

# Follow logs in real-time
sudo journalctl -u myapp -f
```

---

## Troubleshooting Flowchart

```
Start: Service reported down
    ↓
1. Check systemctl status
    ↓
    ├─ Active (running)? → Go to step 2
    └─ Failed/Inactive? → Check journalctl logs
                        → Fix config/dependencies
                        → Restart service
    ↓
2. Check ss -tulpn (port listening?)
    ↓
    ├─ Listening? → Go to step 3
    └─ Not listening? → Service not binding to port
                      → Check config, restart
    ↓
3. Test with curl locally
    ↓
    ├─ Works? → Check external access (firewall/SG)
    └─ Fails fast? → Service not accepting connections
                   → Check logs, config
    ↓
4. Test from client
    ↓
    ├─ Works? → Issue resolved
    ├─ Timeout? → Network path issue (firewall/routing)
    └─ Refused? → Recheck service/port
```

---

## Notes / Mapping to "Refused vs Timed Out"

### Connection Refused / Failed to Connect Quickly

**Usually indicates:**
- Service down
- Port not listening
- Wrong port configured
- Service crashed

**What to check:**
- `systemctl status <service>`
- `ss -tulpn | grep :<port>`
- `journalctl -u <service>`

---

### Connection Timed Out

**Usually indicates:**
- Routing/firewall/filtering issue
- OR service hung and not accepting connections

**What to check:**
- Firewall rules (`iptables`, `ufw`)
- Security Groups (cloud)
- NACLs (cloud)
- Routing tables
- Service resource usage (CPU/memory)

---

### General Rule

Always pair:
- 🔍 **Client test** (`curl`) 
- 🎯 **Listening check** (`ss`)
- 📝 **Logs** (`journalctl`)

---

## Quick Reference Commands

```bash
# Service management
sudo systemctl status <service> --no-pager
sudo systemctl start <service>
sudo systemctl stop <service>
sudo systemctl restart <service>
sudo systemctl enable <service>
sudo systemctl disable <service>

# Check listening ports
sudo ss -tulpn | grep :<port>
sudo netstat -tulpn | grep :<port>  # older alternative
sudo lsof -i :<port>

# View logs
sudo journalctl -u <service> --no-pager
sudo journalctl -u <service> -n 100
sudo journalctl -u <service> --since "1 hour ago"
sudo journalctl -u <service> -f  # follow/tail

# Test connectivity
curl -sv --max-time 3 http://localhost:<port>
curl -skv --max-time 3 https://localhost:<port>  # HTTPS, skip cert check

# Check system resources
df -h                    # disk usage
free -h                  # memory usage
top                      # CPU/memory by process
dmesg | tail -n 50      # kernel messages (OOM kills)
```

---

## Production Checklist

Before declaring service restored:

- [ ] `systemctl status` shows `active (running)`
- [ ] Port is listening (`ss -tulpn`)
- [ ] Local `curl` test succeeds
- [ ] External client test succeeds
- [ ] No errors in recent logs (`journalctl`)
- [ ] Service is enabled for automatic start (`systemctl is-enabled`)
- [ ] Monitored for 5-10 minutes (no immediate crash)
- [ ] Documented incident and root cause
