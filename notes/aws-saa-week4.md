# AWS SAA Study Notes

## Feb 23, 2026 (Week 4)

### CloudTrail Investigation: SSH Access Broke After Change

**Incident scenario:**
- SSH access to EC2 instance suddenly stopped working
- Symptoms indicate recent configuration change
- Need to identify what changed and when

---

### Symptom Identification

**Initial test:**
```bash
nc -vz -w 3 <EC2_PUBLIC_IP> 22
```
**Result:** Connection timeout

**Symptom classification:**
- Timeout (not connection refused or auth failure)
- Indicates network path blocked
- Most likely: Security Group or NACL change

---

### Investigation: CloudTrail Event History

**Step 1: Open CloudTrail Event history**
- Region: us-west-1 (match resource region)
- Time range: Last few hours

**Step 2: Filter to Security Group changes**
- Event name: `RevokeSecurityGroupIngress`
- Resource: `sg-<SECURITY_GROUP_ID>`

---

### Evidence: The Breaking Change

**Event found:**
- **Event time:** 2026-02-21T15:19:58Z
- **Event name:** `RevokeSecurityGroupIngress`
- **Principal:** `arn:aws:iam::<ACCOUNT_ID>:user/stephen-admin`
- **Username:** `stephen-admin`
- **Source IP:** `<MY_PUBLIC_IP>`
- **Resource:** `sg-<SECURITY_GROUP_ID>`

**What was removed:**
```json
{
  "requestParameters": {
    "groupId": "sg-<SECURITY_GROUP_ID>",
    "ipPermissions": {
      "items": [
        {
          "ipProtocol": "tcp",
          "fromPort": 22,
          "toPort": 22,
          "ipRanges": {
            "items": [
              {
                "cidrIp": "<MY_PUBLIC_IP>/32"
              }
            ]
          }
        }
      ]
    }
  }
}
```

**Root cause identified:**
- SSH inbound rule (TCP 22) was revoked
- Source IP `<MY_PUBLIC_IP>/32` no longer allowed
- Change made by `stephen-admin` from console

---

### Resolution: Re-add Security Group Rule

**Action taken:**
- EC2 Console → Security Groups → Select `sg-<SECURITY_GROUP_ID>`
- Add inbound rule: SSH (TCP 22), Source: My IP (`<MY_PUBLIC_IP>/32`)

**Evidence of fix:**
- **Event time:** 2026-02-21T15:26:55Z
- **Event name:** `AuthorizeSecurityGroupIngress`
- **Principal:** `stephen-admin`
- **Action:** Added TCP 22 for `<MY_PUBLIC_IP>/32` on `sg-<SECURITY_GROUP_ID>`

---

### Verification

**Test 1: Network connectivity**
```bash
nc -vz -w 3 <EC2_PUBLIC_IP> 22
```
**Result:** Connection succeeded

**Test 2: SSH connection**
```bash
ssh -i <key.pem> ec2-user@<EC2_PUBLIC_IP>
```
**Result:** Successfully connected

**Outcome:** SSH access restored.

---

### Troubleshooting Workflow Summary

**1) Classify symptom:**
- Timeout → Network path issue (SG, NACL, routing)
- Connection refused → Service/OS issue
- Auth failure → Credentials issue

**2) Use CloudTrail to find recent changes:**
- Filter by relevant event names (`RevokeSecurityGroupIngress`, `AuthorizeSecurityGroupIngress`)
- Filter by resource (security group ID, instance ID, etc.)
- Review events near time issue started

**3) Identify what changed:**
- Review `requestParameters` in event JSON
- Determine exact change (port, protocol, CIDR)
- Confirm change timing aligns with issue

**4) Implement fix:**
- Revert change or apply correct configuration
- Document change in CloudTrail

**5) Verify resolution:**
- Test connectivity with `nc`
- Test actual service (SSH, HTTP, etc.)
- Confirm CloudTrail shows fix event

---

### Key CloudTrail Events for Network Issues

| Issue Type | Event Names to Check | What to Look For |
|-----------|---------------------|------------------|
| **SSH timeout** | `RevokeSecurityGroupIngress`, `ModifyNetworkInterfaceAttribute` | Removed TCP 22 rule, public IP removed |
| **HTTP timeout** | `RevokeSecurityGroupIngress`, `DeleteRoute` | Removed TCP 80/443 rule, IGW route deleted |
| **Instance unreachable** | `StopInstances`, `TerminateInstances`, `ModifySubnetAttribute` | Instance stopped, subnet changed |
| **S3 AccessDenied** | `PutBucketPolicy`, `PutPublicAccessBlock` | Bucket policy changed, BPA enabled |
| **IAM permission error** | `DetachUserPolicy`, `PutUserPolicy` | Policy detached, deny added |

---

### Investigation Tips

**Speed up CloudTrail investigation:**

**Use specific filters:**
- Event name (exact API action)
- Resource name (SG ID, bucket name, etc.)
- Username (if known)
- Time range (narrow to incident window)

**Read events efficiently:**
1. Check `eventName` first (what happened)
2. Check `errorCode` (success or failure)
3. Read `userIdentity.userName` (who did it)
4. Review `requestParameters` (what changed)
5. Check `eventTime` (when it happened)

**Look for patterns:**
- Multiple changes in short time
- Same principal making many changes
- Changes to multiple related resources

---

### Common Mistakes and Prevention

**Mistake:** Accidentally removed wrong security group rule

**Prevention strategies:**
1. **Use resource tags:** Tag resources with purpose/owner
2. **Implement naming conventions:** Clear SG names (e.g., `prod-web-sg`, `dev-bastion-sg`)
3. **Use change management:** Require approval for production changes
4. **Enable CloudTrail alerts:** SNS/Lambda notifications for critical events
5. **Use infrastructure as code:** Terraform/CloudFormation with version control

**Quick recovery:**
- CloudTrail preserves exact change details
- Can reconstruct previous state from `requestParameters`
- Reference previous `AuthorizeSecurityGroupIngress` events to see what was there before

---

### Key Takeaways

**CloudTrail for incident investigation:**
- Essential tool for "what changed" questions
- 90-day event history built-in and free
- Filter by event name and resource for fast investigation

**Network timeout troubleshooting:**
1. Use `nc` to classify issue (timeout vs refused vs auth)
2. Check CloudTrail for recent SG/NACL/route changes
3. Verify public IP, route table, IGW unchanged
4. Fix configuration and verify with `nc` + actual service

**Event investigation pattern:**
- Filter Event history → Open relevant event → Read requestParameters → Identify exact change → Apply fix → Verify

**Prevention > reaction:**
- Use clear naming and tagging
- Implement change management processes
- Set up CloudTrail alerts for critical changes
- Use IaC to track and version infrastructure changes

---

## Feb 23, 2026 (continued)

### Application Load Balancer (ALB) Basics

**Lab focus:**
- Understanding ALB architecture and components
- Target group configuration
- 5xx error troubleshooting approach

---

### Load Balancer Types Overview

**Three types available in EC2:**

| Type | OSI Layer | Use Case | Key Features |
|------|-----------|----------|--------------|
| **Application Load Balancer (ALB)** | Layer 7 (HTTP/HTTPS) | Web applications, microservices | Host/path routing, content-based routing, WebSocket support |
| **Network Load Balancer (NLB)** | Layer 4 (TCP/UDP/TLS) | High performance, non-HTTP protocols | Static IPs, millions of requests/sec, low latency |
| **Gateway Load Balancer (GWLB)** | Layer 3 (IP) | Network security appliances | Routes traffic through third-party security appliances |

**When to choose each:**
- **ALB:** Web apps, APIs, microservices (HTTP/HTTPS routing)
- **NLB:** Gaming, IoT, static IPs required, TCP/UDP applications
- **GWLB:** Security appliances (firewalls, IDS/IPS, DDoS protection)

---

### ALB Architecture Components

**Three core objects:**

**1) Listener (front door):**
- Protocol + port (e.g., HTTP:80, HTTPS:443)
- Rules for routing traffic
- Default action if no rules match

**2) Target group (destination):**
- Collection of targets (instances, IPs, Lambda)
- Target port and protocol
- Health check configuration

**3) Targets:**
- Instances that receive traffic
- Must pass health checks to be eligible
- Can be in multiple availability zones

**Traffic flow:**
```
Client → ALB Listener → Rule evaluation → Target Group → Healthy Target
```

---

### Target Group Configuration

**Target types available:**

| Target Type | Description | Use Case |
|------------|-------------|----------|
| **Instances** | EC2 instance IDs | Standard web apps |
| **IP addresses** | Private IPs (including on-premises) | Hybrid environments, containers |
| **Lambda function** | Serverless function ARN | Serverless applications |
| **Application Load Balancer** | Another ALB as target | NLB → ALB chaining |

**Target group settings:**
- Target port and protocol
- Health check path and interval
- Health check success codes
- Healthy/unhealthy threshold counts
- Deregistration delay

---

### ALB Listener Rules

**ALB uses Layer 7 routing based on:**
- **Host header:** Route by domain (`api.example.com` vs `www.example.com`)
- **Path pattern:** Route by URL path (`/api/*` vs `/images/*`)
- **HTTP headers:** Route based on custom headers
- **HTTP method:** Route by GET, POST, etc.
- **Query string:** Route based on query parameters
- **Source IP:** Route based on client IP

**Rule evaluation:**
- Rules evaluated in priority order (lowest number first)
- First matching rule wins
- Default action if no rules match

---

### Health Check Configuration

**Health check determines target eligibility:**

**Key settings:**
- **Protocol:** HTTP, HTTPS, TCP
- **Port:** Port to check (can differ from target port)
- **Path:** HTTP/HTTPS health check endpoint (e.g., `/health`, `/`)
- **Success codes:** Expected HTTP response codes (e.g., `200`, `200-299`)
- **Interval:** Time between checks (5-300 seconds)
- **Timeout:** Time to wait for response (2-120 seconds)
- **Healthy threshold:** Consecutive successes needed (2-10)
- **Unhealthy threshold:** Consecutive failures needed (2-10)

**Health check states:**
- **Healthy:** Passing health checks, receiving traffic
- **Unhealthy:** Failing health checks, not receiving traffic
- **Initial:** Initial registration period before first check
- **Draining:** Deregistering, completing in-flight requests

---

### Support Mapping: ALB Returning 5xx Errors

**Common 5xx errors and meanings:**

| Error Code | Common Cause | Quick Check |
|-----------|--------------|-------------|
| **502 Bad Gateway** | Target returned invalid response | Target app error, connection failure |
| **503 Service Unavailable** | No healthy targets available | Check target health in target group |
| **504 Gateway Timeout** | Target didn't respond in time | Target slow/hung, timeout settings |

**Note:** These are starting points, not absolute rules. Always verify root cause.

---

### Troubleshooting Flow: ALB 5xx Errors

**Triage order (most common to least common):**

**Step 1: Check target group health**
- EC2 Console → Target Groups → Select target group → Targets tab
- Check: How many healthy targets?
- **0 healthy targets** → Expect 503 errors
- **Some healthy, some unhealthy** → Capacity issue or partial outage
- **All healthy** → Look deeper at target behavior

---

**Step 2: Verify listener rule routing**
- Load Balancers → Listeners tab → View/edit rules
- Confirm: Traffic routes to correct target group
- Test: Use specific host/path that should match
- Common issue: Rule condition doesn't match request

---

**Step 3: Check target health check configuration**
- Target group → Health checks tab
- Verify:
  - Protocol/port match target's listening configuration
  - Path returns success (test with `curl` on target)
  - Success codes match actual response
  - Timeout and interval are reasonable

---

**Step 4: Verify target is listening and responding**

**From the target instance:**
```bash
# Check if service is listening on health check port
ss -tulpn | grep :<health-check-port>

# Test health check locally
curl -I http://localhost:<port><health-check-path>

# Expected: HTTP 200 or configured success code
```

---

**Step 5: Check target security group**
- EC2 → Instance → Security tab → Security groups
- Verify inbound allows:
  - ALB security group on target port (e.g., 80, 8080)
  - ALB security group on health check port (if different)

**Critical requirement:** Target SG must allow inbound from ALB SG, not just `0.0.0.0/0`.

---

### Support Mapping: Targets Showing Unhealthy

**Systematic health check debugging:**

**1) Verify health check configuration matches target:**
```bash
# On target instance, check what's actually listening
ss -tulpn

# Expected: Process listening on health check port
```

**Compare with target group health check settings:**
- Port matches
- Protocol matches (HTTP vs HTTPS)
- Path exists and is accessible

---

**2) Test health check endpoint locally:**
```bash
# Test exactly what the ALB will test
curl -I http://localhost:<health-check-port><health-check-path>

# Expected response
HTTP/1.1 200 OK
```

**If this fails:**
- Application not running
- Health check endpoint misconfigured
- Application error

---

**3) Check health check success codes:**
- Target group → Health checks → Success codes
- Common mistake: Success codes = `200` but app returns `200-399`
- Fix: Update success codes to match actual responses

---

**4) Verify security group allows health checks:**
```
Target Security Group Inbound Rules:
- Type: Custom TCP
- Port: <health-check-port>
- Source: <ALB security group ID>
```

**Common mistake:** Allowing only target port, forgetting health check port (if different).

---

**5) Check target logs for health check requests:**
```bash
# View recent application logs
sudo journalctl -u <service-name> --since "5 min ago" | grep -i health

# OR check web server access logs
sudo tail -f /var/log/nginx/access.log
sudo tail -f /var/log/httpd/access_log
```

**Look for:** Frequent requests to health check path from ALB IPs.

---

### ALB Security Group Requirements

**Two security groups required:**

**1) ALB Security Group:**
- Inbound: Allow HTTP/HTTPS from clients (e.g., `0.0.0.0/0`)
- Outbound: Allow to target security group on target ports

**2) Target Security Group:**
- Inbound: Allow from ALB security group on target port + health check port
- Outbound: Allow responses (usually allow all)

**Example configuration:**

**ALB SG (`sg-alb-123`):**
```
Inbound:
- Type: HTTP, Port: 80, Source: 0.0.0.0/0
- Type: HTTPS, Port: 443, Source: 0.0.0.0/0

Outbound:
- Type: All traffic, Destination: 0.0.0.0/0
```

**Target SG (`sg-target-456`):**
```
Inbound:
- Type: Custom TCP, Port: 8080, Source: sg-alb-123
- Type: Custom TCP, Port: 8080, Source: sg-alb-123 (health check, if same port)

Outbound:
- Type: All traffic, Destination: 0.0.0.0/0
```

---

### Troubleshooting Layers

**Keep layers separate for faster diagnosis:**

**Layer 1: ALB Configuration**
- Listeners configured (protocol + port)
- Rules route to correct target group
- Target group exists and has targets registered

**Layer 2: Target Group Health**
- Health check settings correct (protocol, port, path, success codes)
- Security groups allow health checks
- Sufficient healthy targets for traffic load

**Layer 3: Target Application**
- Service running and listening on correct port
- Application responds correctly to requests
- No application errors in logs
- Sufficient resources (CPU, memory, disk)

**Troubleshooting approach:**
1. Start at Layer 1 (ALB config)
2. Move to Layer 2 (health checks)
3. Only dig into Layer 3 (app) if layers 1 and 2 are correct

---

### Common Misconfiguration Patterns

| Symptom | Common Cause | Fix |
|---------|--------------|-----|
| **All targets unhealthy** | Security group blocking health checks | Add ALB SG to target SG inbound |
| **503 Service Unavailable** | 0 healthy targets | Fix health checks, verify targets running |
| **502 Bad Gateway** | Target returning errors | Check target application logs |
| **504 Gateway Timeout** | Target too slow | Investigate target performance, increase timeout |
| **Requests not routing** | Listener rule doesn't match | Check host/path conditions in rules |
| **Health checks failing** | Wrong path or success codes | Update health check configuration |

---

### Key Takeaways

**ALB architecture:**
- Listener (front door) → Rules → Target Group → Targets
- Layer 7 (HTTP/HTTPS) enables content-based routing
- Targets must pass health checks to receive traffic

**5xx triage priority:**
1. Check target group health (healthy host count)
2. Verify listener rules route correctly
3. Investigate target application and logs

**Health check debugging:**
- Test locally on target first (`curl localhost:port/path`)
- Verify security groups allow ALB → target traffic
- Match health check config to actual target behavior

**Security group pattern:**
- ALB SG: Allow inbound from internet
- Target SG: Allow inbound from ALB SG (not `0.0.0.0/0`)
- Both must allow health check port + target port

**Troubleshooting layers:**
- Layer 1: ALB config (listeners, rules)
- Layer 2: Target group health (health checks, SG)
- Layer 3: Target application (service, logs, resources)
- Debug top-down to avoid wasting time at wrong layer

---

## Feb 24, 2026

### EC2 EBS Volumes + Snapshots Basics

**Lab environment:**
- Instance: `i-<INSTANCE_ID>` (`lab-unreachable`)
- Region: us-west-1
- Availability Zone: us-west-1c

---

### EBS Volume Review

**Instance storage configuration:**
- **Root device:** `/dev/xvda`
- **Attached volumes:** 1 (root volume only)
- **Root volume ID:** `vol-<VOLUME_ID>`

**Volume details:**
- **Type:** gp3 (General Purpose SSD)
- **Size:** 8 GiB
- **State:** in-use
- **Encryption:** No
- **IOPS:** 3000
- **Throughput:** 125 MB/s

**Key observation:** EBS volumes must be in the same Availability Zone as the instance to attach.

---

### EBS Volume Types

**General Purpose SSD:**

| Type | Use Case | Performance | Cost |
|------|----------|-------------|------|
| **gp3** | Default choice for most workloads | Configurable IOPS and throughput independent of size | Lower cost than gp2 |
| **gp2** | Legacy general purpose | IOPS scales with volume size (3 IOPS/GiB) | Higher cost than gp3 for same performance |

**Provisioned IOPS SSD:**

| Type | Use Case | Performance | Cost |
|------|----------|-------------|------|
| **io2** | Mission-critical, high-performance databases | Up to 64,000 IOPS, 99.999% durability | Premium |
| **io1** | Legacy provisioned IOPS | Up to 64,000 IOPS, 99.9% durability | Premium |

**When to choose each:**
- **gp3:** Default for most workloads (boot volumes, dev/test, low-latency apps)
- **io2/io1:** Databases requiring consistent high IOPS (Oracle, SQL Server, MongoDB)

**Key advantage of gp3:** IOPS and throughput are configurable separately from volume size.

---

### Root Volume vs Data Volume

**Root volume:**
- Contains OS and boot files
- Device name typically `/dev/xvda` or `/dev/sda1`
- Created automatically when instance launches
- Deletion behavior configurable (default: delete on termination)

**Data volume:**
- Additional storage attached after instance creation
- Used for application data, databases, logs
- Device names like `/dev/xvdf`, `/dev/xvdg`, etc.
- Requires formatting and mounting in OS
- Typically preserved on instance termination

---

### Snapshots Overview

**What snapshots are:**
- Point-in-time backups of EBS volumes
- Incremental (only changed blocks stored after first snapshot)
- Stored in S3 (managed by AWS, not visible in your S3 console)
- Can be copied across regions
- Can be used to create new volumes or AMIs

**Snapshot vs AMI:**
- **Snapshot:** Backup of a single EBS volume
- **AMI:** Template to launch instances (includes snapshot of root volume + configuration)

**Use cases:**
- Backup before risky changes
- Disaster recovery
- Create AMIs for replication
- Copy data to different AZ/region

---

### EBS Cost Considerations

**Ongoing costs:**
- **Attached volumes:** Charged per GB-month whether instance is running or stopped
- **Detached volumes:** Still charged until deleted
- **Snapshots:** Charged per GB-month for stored data
- **Data transfer:** Charged when copying snapshots across regions

**Cost optimization:**
- Delete unused volumes (check "available" state volumes)
- Delete old snapshots no longer needed
- Use gp3 instead of gp2 (same performance, lower cost)
- Stop instances instead of leaving them running when not in use

**Important:** Stopping an instance does NOT remove volumes or stop volume charges.

---

### Support Mapping: Disk Usage High / Volume Full

**Symptoms:**
- Alert: "No space left on device"
- Application errors or crashes
- SSH lag or timeout
- Log write failures
- Database cannot write

---

**Investigation: On-instance checks**

**Step 1: Identify which filesystem is full**
```bash
df -h
```

**Example output:**
```
Filesystem      Size  Used Avail Use% Mounted on
/dev/xvda1      8.0G  7.8G  200M  98% /
```

**Key information:**
- Which filesystem is full (`/`, `/data`, etc.)
- How much space is used vs available
- Which device the filesystem is on

---

**Step 2: Find largest directories**
```bash
# Check top-level directories under root
sudo du -xhd1 / | sort -h

# Check specific directory
sudo du -xhd1 /var/log | sort -h
```

**Common space consumers:**
- `/var/log` - Application and system logs
- `/tmp` - Temporary files
- `/var/cache` - Package manager cache
- `/home` - User files
- Application-specific directories

---

**Step 3: Confirm volume mapping**
- EC2 Console → Instance → Storage tab
- Match device name to filesystem in `df -h`
- Determine if full filesystem is on root volume or data volume

---

**Resolution options (in order of risk):**

**Option A: Delete or rotate logs (safest)**
```bash
# View large log files
sudo ls -lhS /var/log/

# Truncate a log file (preserves file, empties content)
sudo truncate -s 0 /var/log/large-file.log

# Delete old rotated logs
sudo rm /var/log/*.gz
sudo rm /var/log/*.1 /var/log/*.2
```

---

**Option B: Remove temporary files**
```bash
# Clear /tmp (be cautious, may affect running processes)
sudo rm -rf /tmp/*

# Clear package manager cache (safe)
sudo yum clean all     # Amazon Linux, RHEL
sudo apt-get clean     # Ubuntu, Debian
```

---

**Option C: Increase volume size (requires reboot or careful resize)**

**Step 1: Modify volume size in console**
- EC2 Console → Volumes → Select volume → Actions → Modify volume
- Increase size (e.g., 8 GiB → 20 GiB)
- Wait for modification to complete (state: optimizing)

**Step 2: Extend partition (if needed)**
```bash
# Check current partitions
lsblk

# Extend partition (example for root partition)
sudo growpart /dev/xvda 1
```

**Step 3: Resize filesystem**

**For ext4:**
```bash
sudo resize2fs /dev/xvda1
```

**For xfs:**
```bash
sudo xfs_growfs /
```

**Step 4: Verify**
```bash
df -h
# Should show increased size
```

---

**Verification:**
- `df -h` shows free space available
- Application resumes normal operation
- SSH responsiveness improves
- No more "No space left on device" errors

---

### Support Mapping: Add Second EBS Volume to Instance

**Requirement:** Add storage without modifying root disk

**Example use case:** Separate application data from OS disk

---

**Preparation: Confirm instance Availability Zone**
- EC2 Console → Instance → Details → Availability Zone
- Note: us-west-1c (example)
- New volume MUST be created in same AZ

---

**Step 1: Create new EBS volume**
- EC2 Console → Volumes → Create volume
- **Volume type:** gp3
- **Size:** Based on storage needs (e.g., 50 GiB)
- **Availability Zone:** us-west-1c (MUST match instance)
- **Encryption:** Enable if required
- Create volume

---

**Step 2: Attach volume to instance**
- Volumes → Select new volume → Actions → Attach volume
- **Instance:** Select target instance
- **Device name:** `/dev/xvdf` (or next available)
- Attach

---

**Step 3: Format and mount on instance**

**Check new device:**
```bash
lsblk
```

**Example output:**
```
NAME    MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
xvda    202:0    0   8G  0 disk
└─xvda1 202:1    0   8G  0 part /
xvdf    202:80   0  50G  0 disk
```

**Format new volume (if brand new, not from snapshot):**
```bash
# Create ext4 filesystem
sudo mkfs -t ext4 /dev/xvdf

# OR create xfs filesystem
sudo mkfs -t xfs /dev/xvdf
```

**Warning:** Formatting erases all data. Skip if volume is from a snapshot.

---

**Create mount point and mount:**
```bash
# Create directory
sudo mkdir /data

# Mount volume
sudo mount /dev/xvdf /data

# Verify
df -h
```

---

**Step 4: Make mount persistent (survive reboot)**

**Get volume UUID:**
```bash
sudo blkid /dev/xvdf
```

**Example output:**
```
/dev/xvdf: UUID="abc123-def456-..." TYPE="ext4"
```

**Add to /etc/fstab:**
```bash
sudo nano /etc/fstab
```

**Add line (use UUID, not device name):**
```
UUID=abc123-def456-...  /data  ext4  defaults,nofail  0  2
```

**Test fstab:**
```bash
sudo mount -a
```

**If no errors, configuration is correct.**

---

**Verification:**
- `df -h` shows new mount point with correct size
- `lsblk` shows volume mounted
- Test write: `sudo touch /data/test.txt`
- Reboot instance and verify mount persists

---

### Common EBS Operations Reference

| Task | Command/Action | Notes |
|------|---------------|-------|
| **Check disk usage** | `df -h` | Shows filesystem usage and mount points |
| **Find large directories** | `sudo du -xhd1 / \| sort -h` | Identifies space consumers |
| **List block devices** | `lsblk` | Shows all disks and partitions |
| **Check volume in console** | EC2 → Volumes | View state, size, IOPS, attachments |
| **Resize volume** | Modify volume → Increase size | Requires filesystem resize after |
| **Extend partition** | `sudo growpart /dev/xvda 1` | Expands partition to use new space |
| **Resize ext4** | `sudo resize2fs /dev/xvda1` | Grows ext4 filesystem |
| **Resize xfs** | `sudo xfs_growfs /` | Grows xfs filesystem |
| **Format new volume** | `sudo mkfs -t ext4 /dev/xvdf` | Creates filesystem (erases data) |
| **Get UUID** | `sudo blkid /dev/xvdf` | Gets UUID for fstab entry |

---

### Troubleshooting Tips

**Volume won't attach:**
- Verify volume and instance are in same AZ
- Check instance is running or stopped (not terminated)
- Verify device name not already in use

**Filesystem not growing after resize:**
- Check partition was extended (`lsblk`)
- Verify used correct resize command (ext4 vs xfs)
- Ensure running resize on mounted filesystem

**Mount fails after reboot:**
- Check fstab syntax (spaces vs tabs matter)
- Verify UUID is correct
- Use `nofail` option to prevent boot hang
- Test with `sudo mount -a` before rebooting

**"No space left" but df shows space:**
- Check inode usage: `df -i`
- Too many small files can exhaust inodes
- May need to delete many small files

---

### Key Takeaways

**EBS volume basics:**
- Must be in same AZ as instance
- Charges apply even when instance is stopped
- Root vs data volumes serve different purposes
- gp3 is default choice for most workloads

**Disk full troubleshooting:**
1. Use `df -h` to identify full filesystem
2. Use `du` to find space consumers
3. Delete logs/temp files first
4. Resize volume if needed
5. Always extend filesystem after volume resize

**Adding data volume:**
1. Confirm instance AZ
2. Create volume in same AZ
3. Attach to instance
4. Format (if new), mount, test
5. Add to fstab for persistence

**Cost awareness:**
- Volumes cost money whether attached or not
- Snapshots accumulate storage costs
- Delete unused resources regularly
- gp3 typically cheaper than gp2 for same performance

---

## Feb 25, 2026

### S3 Permissions + AccessDenied Triage

**Lab setup:**
- Created bucket: `stephen-s3-perms-0225`
- Region: us-west-1
- Purpose: Review S3 permissions architecture and troubleshooting

---

### S3 Access Control Layers

**Three main layers evaluated for access decisions:**

1. **IAM Identity Policy:**
   - Attached to users, groups, or roles
   - Grants permissions to the principal
   - Must explicitly allow the action

2. **S3 Bucket Policy:**
   - Attached to the S3 bucket
   - Can allow or deny specific principals
   - Can include conditions (IP, MFA, VPC endpoint, etc.)

3. **Block Public Access (BPA):**
   - Account-level and bucket-level settings
   - Overrides policies that would allow public access
   - Four settings (all should be ON for baseline security)

**Additional layer (if applicable):**
- **KMS Key Policy:** If bucket uses SSE-KMS encryption

---

### Lab Bucket Configuration Review

**Caller identity verification:**
```bash
aws sts get-caller-identity
```
- **Principal:** IAM user `stephen-admin`
- **Permissions:** AdministratorAccess (broad IAM policy)
- **Expected:** Bucket accessible unless explicit deny or BPA blocks

---

**Block Public Access settings:**
- Location: S3 Console → Bucket → Permissions → Block public access
- **All 4 settings:** ON (correct baseline)
- **Purpose:** Prevent unintended public exposure

**Four BPA settings:**
1. Block public access to buckets and objects granted through new access control lists (ACLs)
2. Block public access to buckets and objects granted through any access control lists (ACLs)
3. Block public access to buckets and objects granted through new public bucket or access point policies
4. Block public and cross-account access to buckets and objects through any public bucket or access point policies

---

**Bucket policy:**
- Location: S3 Console → Bucket → Permissions → Bucket policy
- **Current state:** No bucket policy defined
- **Implication:** No policy-based allow or deny affecting access

---

**Object Ownership:**
- Location: S3 Console → Bucket → Permissions → Object Ownership
- **Setting:** Bucket owner enforced
- **Implication:** ACLs disabled, bucket policy is main access control
- **Best practice:** Prevents ACL-based public access issues

---

**Default encryption:**
- Location: S3 Console → Bucket → Properties → Default encryption
- **Encryption type:** SSE-S3 (S3-managed keys)
- **Implication:** No KMS permissions required for this bucket

---

### S3 Action-to-ARN Mapping

**Critical distinction: Bucket operations vs Object operations**

| Action | Required Permission | Resource ARN |
|--------|-------------------|--------------|
| **List bucket** | `s3:ListBucket` | `arn:aws:s3:::bucket-name` (bucket ARN) |
| **Read object** | `s3:GetObject` | `arn:aws:s3:::bucket-name/*` (object ARN with wildcard) |
| **Write object** | `s3:PutObject` | `arn:aws:s3:::bucket-name/*` |
| **Delete object** | `s3:DeleteObject` | `arn:aws:s3:::bucket-name/*` |

**Common mistake:** Using bucket ARN when object ARN is needed (or vice versa)

---

### Support Mapping: S3 AccessDenied (403) Triage

**Systematic troubleshooting order:**

**Step 1: Confirm caller identity**
```bash
aws sts get-caller-identity
```
- Verify who is making the request
- Check if it's the expected user/role
- Note account ID

---

**Step 2: Identify exact action and resource**

**From error message, extract:**
- **Action:** What were you trying to do? (`ListBucket`, `GetObject`, `PutObject`)
- **Bucket:** Which bucket? (e.g., `my-bucket`)
- **Object key:** Which object? (if object operation)

---

**Step 3: Check IAM identity policy**

**For ListBucket (403 when listing):**
```json
{
  "Effect": "Allow",
  "Action": "s3:ListBucket",
  "Resource": "arn:aws:s3:::bucket-name"
}
```

**For GetObject (403 when reading):**
```json
{
  "Effect": "Allow",
  "Action": "s3:GetObject",
  "Resource": "arn:aws:s3:::bucket-name/*"
}
```

**Common issues:**
- Missing permission entirely
- Wrong resource ARN (bucket ARN instead of object ARN, or vice versa)
- Typo in bucket name

---

**Step 4: Check bucket policy for explicit deny or conditions**

**S3 Console → Bucket → Permissions → Bucket policy**

**Look for:**
- `"Effect": "Deny"` statements
- Condition blocks restricting access:
  - `IpAddress` or `NotIpAddress`
  - `aws:SecureTransport` (HTTPS required)
  - `aws:MultiFactorAuthPresent`
  - `aws:SourceVpce` (VPC endpoint restriction)

**Example restrictive policy:**
```json
{
  "Effect": "Deny",
  "Principal": "*",
  "Action": "s3:*",
  "Resource": [
    "arn:aws:s3:::bucket-name",
    "arn:aws:s3:::bucket-name/*"
  ],
  "Condition": {
    "NotIpAddress": {
      "aws:SourceIp": "<ALLOWED_IP>/32"
    }
  }
}
```

---

**Step 5: Check Block Public Access settings**

**Note:** BPA typically prevents public access, not authenticated principal access.

**Exception:** If BPA setting "Block public and cross-account access" is ON, cross-account access may be blocked even with bucket policy.

---

**Step 6: If SSE-KMS encryption, verify KMS permissions**

**Check encryption type:**
- S3 Console → Bucket → Properties → Default encryption
- If SSE-KMS (not SSE-S3):

**Required IAM permissions:**
```json
{
  "Effect": "Allow",
  "Action": [
    "kms:Decrypt",
    "kms:DescribeKey"
  ],
  "Resource": "arn:aws:kms:region:account:key/key-id"
}
```

**Required KMS key policy:**
- Key policy must allow the principal to use the key
- Check: KMS Console → Keys → Select key → Key policy

---

### Resolution Steps: AccessDenied

**If IAM policy missing/incorrect:**
1. Add required permission (s3:ListBucket or s3:GetObject)
2. Verify correct resource ARN (bucket vs object)
3. Test access again

**If bucket policy has explicit deny:**
1. Review deny conditions
2. Remove deny or adjust conditions to exclude this principal
3. Test access again

**If KMS permissions missing:**
1. Add kms:Decrypt to IAM policy
2. Verify KMS key policy allows principal
3. Test access again

---

### Verification

**Test list bucket:**
```bash
aws s3 ls s3://bucket-name/
```

**Test read object:**
```bash
aws s3 cp s3://bucket-name/object-key /tmp/test
```

**Check CloudTrail for policy changes:**
- CloudTrail Event history
- Filter by event names:
  - `PutBucketPolicy`
  - `PutPublicAccessBlock`
  - `PutBucketAcl`
  - `PutBucketOwnershipControls`

---

### Support Mapping: Bucket is Public When It Shouldn't Be

**Security concern: Unintended public exposure**

**Systematic investigation:**

**Step 1: Check Block Public Access settings**

**Bucket-level:**
- S3 Console → Bucket → Permissions → Block public access (bucket settings)
- **Expected:** All 4 settings ON

**Account-level:**
- S3 Console → Block Public Access settings for this account
- **Expected:** All 4 settings ON

**If any are OFF:** Enable them immediately

---

**Step 2: Check bucket policy for public access**

**S3 Console → Bucket → Permissions → Bucket policy**

**Look for:**
```json
{
  "Effect": "Allow",
  "Principal": "*",
  "Action": "s3:GetObject",
  "Resource": "arn:aws:s3:::bucket-name/*"
}
```

**`Principal: "*"` = public access**

---

**Step 3: Check Object Ownership setting**

**S3 Console → Bucket → Permissions → Object Ownership**

**Recommended setting:** Bucket owner enforced
- Disables ACLs
- Prevents object-level public access via ACLs
- Bucket policy is sole access control mechanism

**If set to "Object writer":**
- Objects can have individual ACLs
- Object uploader can grant public access
- Review each object's ACL

---

**Step 4: Check for public object ACLs (if ownership not enforced)**

**From CLI:**
```bash
# List objects with public ACLs
aws s3api list-objects-v2 --bucket bucket-name \
  --query 'Contents[?StorageClass==`STANDARD`].[Key]' \
  --output text | while read key; do
    aws s3api get-object-acl --bucket bucket-name --key "$key"
  done
```

**Look for:** Grantee with URI `http://acs.amazonaws.com/groups/global/AllUsers`

---

### Resolution Steps: Remove Public Access

**Step 1: Enable Block Public Access**
- S3 Console → Bucket → Permissions → Block public access
- Edit → Check all 4 boxes → Save changes

**Step 2: Remove public bucket policy**
- Bucket → Permissions → Bucket policy
- Remove or modify statements with `"Principal": "*"`
- Save changes

**Step 3: Enforce Object Ownership**
- Bucket → Permissions → Object Ownership
- Edit → Select "Bucket owner enforced"
- Save changes

**Step 4: Verify public access removed**
- Check bucket details page
- "Public access" indicator should show "Objects can be public" (if BPA off) or "Bucket and objects not public"

**Test anonymous access:**
```bash
curl -I https://bucket-name.s3.region.amazonaws.com/object-key
# Should return 403 Forbidden
```

---

### CloudTrail Investigation for S3 Changes

**Key event names to filter:**

| Change Type | Event Name | What to Look For |
|------------|-----------|------------------|
| **Bucket policy changed** | `PutBucketPolicy`, `DeleteBucketPolicy` | New policy JSON, who changed it |
| **BPA changed** | `PutPublicAccessBlock` | Which settings changed, who changed |
| **Object ownership changed** | `PutBucketOwnershipControls` | New ownership setting |
| **ACL changed** | `PutBucketAcl`, `PutObjectAcl` | New ACL grants |

**Investigation steps:**
1. CloudTrail → Event history
2. Filter by event name
3. Filter by resource (bucket name)
4. Review `requestParameters` for changes
5. Note `userIdentity` (who made change)
6. Note `eventTime` (when it happened)

---

### Common S3 AccessDenied Patterns

| Symptom | Common Cause | Where to Check |
|---------|--------------|----------------|
| **403 listing bucket** | IAM missing s3:ListBucket on bucket ARN | IAM policy Resource statement |
| **403 reading object** | IAM missing s3:GetObject on object ARN | IAM policy Resource statement |
| **403 despite broad IAM** | Bucket policy explicit deny | Bucket policy Deny statements |
| **403 from specific IP** | Bucket policy IP condition | Bucket policy Condition blocks |
| **403 on KMS-encrypted object** | Missing kms:Decrypt permission | IAM policy + KMS key policy |
| **403 cross-account** | BPA blocking cross-account | BPA "Block public and cross-account" |

---

### Key Takeaways

**S3 access evaluation order:**
1. Caller identity confirmed
2. IAM policy allows action on correct resource ARN
3. Bucket policy doesn't explicitly deny
4. BPA doesn't block (mainly for public access scenarios)
5. KMS permissions if applicable

**Critical ARN distinction:**
- Bucket operations (ListBucket) → bucket ARN: `arn:aws:s3:::bucket-name`
- Object operations (GetObject) → object ARN: `arn:aws:s3:::bucket-name/*`

**403 triage workflow:**
1. Confirm caller identity
2. Verify IAM allows with correct resource ARN
3. Check bucket policy for explicit deny or conditions
4. Check BPA (if public access involved)
5. Check KMS (if SSE-KMS encryption)
6. Use CloudTrail to find recent changes

**Public bucket prevention:**
- Enable BPA (all 4 settings ON)
- Use "Bucket owner enforced" object ownership
- Review bucket policies for `Principal: "*"`
- Use CloudTrail to track permission changes

**Best practices:**
- Default to BPA enabled
- Use bucket owner enforced ownership
- Minimize use of bucket policies (prefer IAM policies)
- Use SSE-S3 unless KMS required (simpler permissions)
- Regularly audit for public buckets

---
