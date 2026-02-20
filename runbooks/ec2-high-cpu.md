# Runbook: EC2 High CPU Usage

## Goal
Diagnose and resolve high CPU utilization on an EC2 instance.

## When to Use This Runbook
- CloudWatch **CPUUtilization** alarm transitions from OK → ALARM
- Instance or application feels slow and you suspect CPU saturation
- Users report increased latency or timeouts

## Symptoms
- CloudWatch alarm: CPUUtilization above threshold (e.g., >70%)
- Application slow responses, timeouts, increased request latency
- SSH lag or commands take longer to execute
- High load average (visible in `uptime`)
- On burstable instances (t3/t4g): CPU credit balance drops rapidly during sustained load

---

## Quick Triage (2-5 Minutes)

### 1) Capture Context
- Instance ID
- Instance name
- Region
- Alarm name
- Timestamp when alarm triggered

### 2) Check CloudWatch Metrics
**EC2 Console → Instance → Monitoring tab** or **CloudWatch Console → Alarms**

**Primary metric:**
- CPUUtilization graph
- Note: Statistic type (Average vs Maximum)
- Note: Period (1-minute vs 5-minute intervals)
- Note: Datapoints required to trigger alarm

**Related metrics to check:**
- **NetworkIn/NetworkOut:** Traffic spike correlating with CPU spike?
- **StatusCheckFailed:** Instance impaired?
- **CPUCreditBalance/CPUCreditUsage:** (T-series only) Credits depleting?

### 3) Connect to Instance
```bash
# Check system load
uptime

# View real-time process activity
top
```

**In `top`:**
- Press `P` to sort by CPU usage
- Press `1` to show per-core CPU breakdown
- Note load average and CPU time breakdown (us/sy/id/wa/st)

---

## Troubleshooting Flow (In Order)

### 1) Identify Top CPU Consumer

**List processes by CPU usage:**
```bash
ps -eo pid,ppid,user,pcpu,pmem,etime,cmd --sort=-pcpu | head -15
```

**View per-thread CPU usage (if needed):**
```bash
top -H
```

**Check service status (if top process is a service):**
```bash
systemctl status <service-name> --no-pager
```

---

### 2) Determine CPU Type Breakdown

**In `top`, examine CPU time categories:**

| Category | Name | Meaning | Common Causes |
|----------|------|---------|---------------|
| **us** | User | Application code execution | Busy loops, compute-heavy tasks, inefficient code |
| **sy** | System | Kernel operations | Heavy filesystem/network I/O, system calls |
| **wa** | I/O Wait | Waiting for disk/network | Disk bottleneck, slow storage, network congestion |
| **st** | Steal | CPU stolen by hypervisor | Host contention (rare on AWS) |
| **id** | Idle | CPU available | (Lower is busier) |

**Key insight:** High `wa` (I/O wait) looks like high CPU but is actually a storage/network bottleneck, not a CPU problem.

---

### 3) Check Service Logs Around the Spike

**If you know the service name:**
```bash
journalctl -u <service-name> --since "15 min ago" --no-pager
```

**If service is unknown:**
```bash
journalctl --since "15 min ago" --no-pager | tail -200
```

**Look for:**
- Error messages
- Warnings
- Increased request volume
- Long-running operations

---

### 4) Check for Recent Changes

**Recent changes often cause sudden CPU spikes:**
- New deployment
- Configuration change
- Scheduled job (cron/systemd timer)
- Package updates

**Check scheduled tasks:**
```bash
systemctl list-timers --all | head -50

# Check cron jobs
crontab -l
sudo cat /etc/crontab
ls -l /etc/cron.d/
```

**Check recent package installs/updates (Amazon Linux/RHEL-based):**
```bash
rpm -qa --last | head -20
```

**Check recent package installs/updates (Ubuntu/Debian-based):**
```bash
grep " install " /var/log/dpkg.log | tail -20
```

---

### 5) Check CPU Credit Behavior (T-Series Instances Only)

**If using t3, t3a, t4g instances:**
- CloudWatch → CPUCreditBalance metric
- If balance is trending down + CPU sustained high → instance is depleting credits
- Once credits exhausted, CPU throttles to baseline performance

**Baseline performance rates:**

| Instance Type | Baseline CPU % | vCPUs |
|--------------|----------------|-------|
| t3.micro | 10% | 2 |
| t3.small | 20% | 2 |
| t3.medium | 20% | 2 |
| t3.large | 30% | 2 |

**Solution if credits depleting:**
- Scale up to larger t-series instance (more credits)
- Switch to non-burstable instance type (m6i, c6i, etc.)

---

## Resolution Options (Choose Least Risky First)

### Option A: Restart Runaway Service/Process

**If a specific service is consuming excessive CPU:**

**1) Restart the service (preferred):**
```bash
sudo systemctl restart <service-name>

# Verify service restarted successfully
sudo systemctl status <service-name> --no-pager
```

**2) Stop a specific process (if service restart not possible):**

**Graceful termination (try first):**
```bash
sudo kill -TERM <PID>

# Wait 10 seconds, verify process stopped
ps -p <PID>
```

**Forceful termination (if graceful fails):**
```bash
sudo kill -KILL <PID>
```

**Warning:** Killing processes can:
- Drop in-flight requests
- Corrupt data being written
- Cause service disruptions

---

### Option B: Reduce Load

**Temporary load reduction strategies:**

**Rate limit upstream traffic:**
- If you control the source, reduce request rate
- Implement backpressure or queuing

**Disable batch jobs temporarily:**
```bash
# Disable a systemd timer
sudo systemctl stop <timer-name>.timer
sudo systemctl disable <timer-name>.timer

# Verify timer is inactive
systemctl list-timers --all
```

**Scale out (add more instances):**
- If workload is legitimate, add capacity
- Use Auto Scaling Group to distribute load
- Add instances behind load balancer

---

### Option C: Scale Up Instance Type

**When to scale up:**
- CPU consistently high during normal operations
- Application requires more compute capacity
- T-series instance depleting credits regularly

**Steps:**
1. Stop instance
2. Change instance type (Actions → Instance settings → Change instance type)
3. Start instance
4. Verify application performance improves

**Considerations:**
- **T-series → Non-burstable:** If sustained CPU is normal workload
- **Same family, larger size:** m6i.large → m6i.xlarge (2x vCPUs)
- **Different family:** Consider compute-optimized (c6i) for CPU-intensive workloads

---

### Option D: Tune CloudWatch Alarm (After Root Cause Understood)

**Alarm design considerations:**

| Alarm Purpose | Statistic | Evaluation Period | Datapoints |
|--------------|-----------|-------------------|------------|
| Catch brief spikes | Maximum | 1 minute | 1 out of 1 |
| Sustained high CPU | Average | 5 minutes | 3 out of 3 |
| Balance sensitivity | Average | 5 minutes | 2 out of 3 |

**Example alarm configurations:**

**Sensitive (catches brief spikes):**
- Threshold: CPU > 70%
- Statistic: Maximum
- Period: 1 minute
- Evaluation: 1 datapoint

**Stable (ignores brief spikes):**
- Threshold: CPU > 70%
- Statistic: Average
- Period: 5 minutes
- Evaluation: 3 out of 5 datapoints

**Document alarm changes so thresholds remain meaningful over time.**

---

## Verification

### 1) CloudWatch Metrics
```bash
# Check from AWS CLI (optional)
aws cloudwatch get-metric-statistics \
  --namespace AWS/EC2 \
  --metric-name CPUUtilization \
  --dimensions Name=InstanceId,Value=<instance-id> \
  --start-time <timestamp> \
  --end-time <timestamp> \
  --period 300 \
  --statistics Average
```

**Expected results:**
- CPUUtilization drops below threshold
- Alarm state returns to **OK**
- Related metrics (Network, Status checks) appear normal

### 2) On the Instance
```bash
# Check current load
uptime

# Verify top processes are normal
top

# Check CPU usage trend
ps -eo pid,ppid,user,pcpu,pmem,etime,cmd --sort=-pcpu | head -10
```

**Expected results:**
- Load average trending down or stable at normal levels
- Top CPU-consuming process is expected/normal
- No runaway processes

### 3) Application Health
```bash
# Test application endpoint
curl -I http://localhost:<port>/health

# Check recent error logs
journalctl -u <service-name> --since "5 min ago" --no-pager | grep -i error
```

**Expected results:**
- Application responds normally
- Error rates return to baseline
- Response times are normal

---

## Information to Capture for Support Ticket

**If escalating to AWS Support or internal infrastructure team:**

**1) Alarm details:**
- Alarm name
- State transitions with timestamps (OK → ALARM → OK)
- Threshold and evaluation settings

**2) Instance details:**
- Instance ID
- Instance name/tag
- Region and Availability Zone
- Instance type

**3) CloudWatch data:**
- CPUUtilization graph screenshot
- CPUCreditBalance graph (if T-series)
- Time range covering the spike

**4) Instance diagnostics:**
```bash
# Capture system state
uptime
ps -eo pid,ppid,user,pcpu,pmem,etime,cmd --sort=-pcpu | head -15
journalctl --since "30 min ago" --no-pager | tail -200
```

**5) Recent changes:**
- Recent deployments
- Package updates
- Configuration changes
- New cron jobs or scheduled tasks
- Traffic pattern changes

---

## Lab Testing (Optional)

**Reproduce CPU spike for testing:**

**Install stress tool:**
```bash
# Amazon Linux 2023
sudo dnf install -y stress-ng

# Ubuntu
sudo apt-get install -y stress-ng
```

**Generate CPU load:**
```bash
# Stress all CPU cores for 5 minutes
stress-ng --cpu 0 --timeout 300s

# Stress specific number of cores
stress-ng --cpu 2 --timeout 300s
```

**Monitor in CloudWatch:**
- Watch CPUUtilization metric rise
- Observe alarm state change
- Test resolution procedures

**Cleanup after testing:**
```bash
# Stop stress-ng if still running
pkill stress-ng

# Remove stress-ng
sudo dnf remove -y stress-ng  # Amazon Linux
sudo apt-get remove -y stress-ng  # Ubuntu
```

---

## Common Root Cause Patterns

| Symptom | Common Cause | Resolution |
|---------|--------------|------------|
| Sudden spike, then drops | Batch job or scheduled task | Reschedule or optimize job |
| Gradual increase over time | Memory leak causing CPU to compensate | Restart service, investigate leak |
| Sustained high during business hours | Under-provisioned for workload | Scale up or scale out |
| High `wa` (I/O wait), not `us` | Disk bottleneck, not CPU | Check disk I/O, consider faster storage |
| T-series credit depletion | Workload exceeds baseline | Switch to non-burstable instance |

---

## Prevention and Best Practices

**Monitoring:**
- Set up alarms for both short spikes and sustained high CPU
- Monitor CPU credits on T-series instances
- Track application-level metrics (request rates, latency)

**Capacity planning:**
- Right-size instances based on actual workload patterns
- Use Auto Scaling for variable workload
- Consider Reserved Instances for predictable baseline capacity

**Application optimization:**
- Profile and optimize CPU-intensive code paths
- Implement caching to reduce compute requirements
- Use asynchronous processing for batch operations

**Operational practices:**
- Test deployments in staging before production
- Schedule batch jobs during off-peak hours
- Implement gradual rollouts to detect issues early
