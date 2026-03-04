# Runbook: EC2 - Can't SSH

## Purpose

Diagnose and resolve SSH connectivity issues to EC2 instances using systematic network path verification.

---

## Symptoms

**Common reports:**
- SSH hangs then times out
- Connection refused
- `nc -vz -w 3 <public-ip> 22` fails

---

## Fast Checks (In Order)

### 1) AWS Console Triage (Complete Network Path)

**Use this systematic order to avoid guessing. Most SSH failures are network path or Security Group source IP issues.**

---

#### Instance State and Target IP

**EC2 Console → Instances → Select instance**

**Capture these details:**
- Instance state: Must be `running`
- Public IPv4 address (or Elastic IP assigned)
- Subnet ID and VPC ID
- Security Group ID(s)
- Key pair name (assigned at launch)
- Availability Zone

**Critical:** If no public IP exists, you cannot SSH directly from internet

---

#### Subnet Public Reachability (Route to IGW)

**VPC Console → Subnets → Select subnet → Route table tab → Click route table ID**

**Verify routes:**
```
Destination          Target
172.31.0.0/16    →  local (VPC CIDR)
0.0.0.0/0        →  igw-...
```

**Required for SSH from internet:**
- Route `0.0.0.0/0 → igw-...` must exist
- This proves subnet has internet routing

**If route missing or points elsewhere:** SSH will timeout

---

#### Security Group Inbound (Most Common Root Cause)

**EC2 Console → Security Groups → Select attached SG → Inbound rules tab**

**Required rule:**
```
Type: SSH
Protocol: TCP
Port: 22
Source: <YOUR_CURRENT_IP>/32
```

**Use "My IP" option:** Console automatically fills your current public IP

**Common failure pattern:**
```
Scenario: SSH worked yesterday, fails today
Cause: Your ISP/VPN changed your public IP
Result: Connection timeout
Fix: Update Security Group rule to current IP
```

**Verify source IP matches:**
```bash
# Check your current public IP
curl ifconfig.me

# Compare with Security Group rule source
```

---

#### Network ACL (Only If Custom/Restrictive)

**VPC Console → Network ACLs → Find NACL associated with subnet**

**Default NACL:** Allows all traffic (rarely the issue)

**If NACL is restrictive, verify both directions (stateless):**

**Inbound rules:**
- Allow TCP 22 from your client IP (or `0.0.0.0/0` for labs)

**Outbound rules:**
- Allow TCP 1024-65535 to your client IP (or `0.0.0.0/0`)
- Ephemeral ports required for return traffic

**Rule evaluation:**
- Lower rule numbers evaluated first
- Explicit deny overrides later allows
- Ensure rule order doesn't deny before allow

---

#### Status Checks (Separate Network vs Host Issues)

**EC2 Console → Instances → Status checks tab**

**Two check types:**

| Check Type | What It Tests | Failed Indicates |
|-----------|---------------|-----------------|
| **System status** | AWS infrastructure (hypervisor, network, power) | AWS host/hardware issue |
| **Instance status** | Guest OS (network config, software) | OS-level issue |

**Interpretation:**

**System status failed:**
- AWS infrastructure problem
- Resolution: Stop and start instance (moves to new host)
- Or contact AWS Support in production

**Instance status failed:**
- OS or application issue
- Resolution: Use SSM Session Manager or EC2 Instance Connect to investigate
- Check: sshd service, disk space, OS firewall

**Both passed:**
- Network path and OS likely healthy
- Issue is typically: SG rules, NACL, SSH key, or username

---

#### Instance-Side Causes (After Network Path Confirmed)

**Check these only after confirming port 22 is reachable:**

**Wrong username:**

| AMI Type | Correct Username |
|----------|-----------------|
| Amazon Linux 2/2023 | `ec2-user` |
| Ubuntu | `ubuntu` |
| Red Hat | `ec2-user` |
| Debian | `admin` |
| CentOS | `centos` |

**Wrong SSH key:**
- Must match key pair assigned at instance launch
- Verify: EC2 Console → Instance → Details → Key pair name

**Key file permissions:**
```bash
# Check permissions
ls -l key.pem

# Fix if needed (must be 400 or 600)
chmod 400 key.pem
```

**SSH daemon or host issues:**
- Use SSM Session Manager or EC2 Instance Connect to check:
```bash
# Check sshd service
sudo systemctl status sshd

# Check port listening
sudo ss -tulpn | grep :22

# Check disk space
df -h
```

**Disk full:**
- Can break logins and services
- Check when you have console access

---

### Symptom Classification and Root Cause Mapping

**Use this table to quickly identify likely issue:**

| Symptom | Meaning | Most Likely Root Cause | Where to Look First |
|---------|---------|----------------------|-------------------|
| **Timeout / hangs** | No network path to port 22 | SG source IP wrong/stale, missing IGW route, NACL deny | Security Group inbound rules |
| **Connection refused** | Host reachable but port closed | sshd not running, wrong port, host firewall | Service status on instance |
| **Permission denied (publickey)** | Network OK, auth failed | Wrong key, wrong username, key permissions | SSH key and username |
| **Host key verification failed** | Client distrusts host identity | Instance rebuilt or IP reused | Remove old key from `~/.ssh/known_hosts` |

---

### Quick Diagnostic Commands

**Test port reachability:**
```bash
nc -vz -w 3 <public-ip> 22
```
- Success: Port is open
- Timeout: Network path blocked
- Refused: Port closed or service down

**Verbose SSH for detailed debugging:**
```bash
ssh -vvv -i <key.pem> <username>@<public-ip>
```
- Shows each connection step
- Reveals where failure occurs

**Check your current public IP:**
```bash
curl ifconfig.me
```
- Verify matches Security Group rule source

---

### Verification Checklist

**Before declaring issue resolved, confirm:**

✅ **SSH connection succeeds:**
```bash
ssh -i <key.pem> <username>@<public-ip>
# Should: Connect and show shell prompt
```

✅ **Security Group properly restricted:**
- Source limited to your IP `/32` (for labs)
- Or appropriate CIDR for production

✅ **Route table unchanged:**
- Still has `0.0.0.0/0 → igw-...`

✅ **Document changes:**
- What was changed
- Why it was changed
- Restore to least-privilege access

✅ **Test persists:**
- Disconnect and reconnect
- Verify stable connection

---

## Fix Steps

### Fix 1: Add Security Group inbound rule

**AWS Console:**
1. Navigate to EC2 → Security Groups
2. Select security group attached to instance
3. Inbound rules → Edit inbound rules
4. Add rule:
   - Type: SSH
   - Protocol: TCP
   - Port: 22
   - Source: "My IP" (auto-fills your current public IP `/32`)
5. Save rules

---

### Fix 2: Update NACL (if restrictive)

**Check NACL first:**
- Default NACL allows all traffic (no action needed)
- Custom restrictive NACL requires explicit rules

**Add NACL rules if needed:**

**Inbound:**
- Rule: Allow TCP 22 from source (your IP or `0.0.0.0/0`)

**Outbound:**
- Rule: Allow TCP 1024-65535 to destination (your IP or `0.0.0.0/0`)

---

### Fix 3: Instance-level checks (if connection succeeds but SSH fails)

**Verify sshd service:**
```bash
# Connect via Systems Manager Session Manager or EC2 Instance Connect
sudo systemctl status sshd
```

**Common instance issues:**
- sshd not running
- OS firewall blocking port 22
- Wrong SSH key
- Wrong username for AMI:
  - Amazon Linux 2/2023: `ec2-user`
  - Ubuntu: `ubuntu`
  - Red Hat: `ec2-user`
  - Debian: `admin`

---

## Verify (Must Pass All)

### 1) Port reachability test
```bash
nc -vz -w 3 <public-ip> 22
```

**Success output:**
```text
Connection to <public-ip> 22 port [tcp/ssh] succeeded!
```

---

### 2) SSH connection test
```bash
ssh -i <key.pem> ec2-user@<public-ip>
```

**Before first attempt:**
- Verify key permissions: `chmod 400 key.pem`
- Use correct username for AMI

**Success criteria:**
- ✅ Connection established
- ✅ Authentication succeeds
- ✅ Shell prompt appears

---

## Troubleshooting Decision Flow

**Start here:**
```
Can't SSH to instance
    ↓
Test with nc
    ↓
├─ Timeout → Network path issue
│   ├─ Check: Instance has public IP
│   ├─ Check: Subnet route table (0.0.0.0/0 → IGW)
│   ├─ Check: IGW attached to VPC
│   ├─ Check: Security Group inbound (TCP 22 from my IP)
│   └─ Check: NACL (if restrictive)
│
├─ Connection refused → Service/port issue
│   ├─ Check: sshd running (systemctl status sshd)
│   ├─ Check: Port 22 listening (ss -tulpn | grep :22)
│   └─ Check: OS firewall
│
└─ Connects but auth fails → Credentials issue
    ├─ Check: Correct username for AMI
    ├─ Check: Using correct SSH key
    └─ Check: Key file permissions (400 or 600)
```

---

## Hands-On Lab Example

### Lab setup (Feb 14, 2026)

**Environment:**
- Region: us-west-1
- VPC: `vpc-0622cb5ac599f4c36`
- Subnet: `subnet-0ed1fa4a40bc62a26` (us-west-1c)
- Internet Gateway: `igw-0d2b4afba531a9b3b`
- Instance: Amazon Linux 2023
- Public IP: `<REDACTED>`

---

### Verification checklist completed

✅ **1. Instance has public IP**
- Confirmed public IPv4 assigned

✅ **2. Subnet routing**
- Route table has `0.0.0.0/0 → igw-0d2b4afba531a9b3b`

✅ **3. Internet Gateway**
- IGW attached to VPC

✅ **4. Security Group**
- Inbound rule exists:
  - Type: SSH (TCP 22)
  - Source: My public IP `/32`
- Rule ID: `sgr-0317186940658c253`

✅ **5. Network ACL**
- Using default NACL
- Rule 100: ALLOW all traffic inbound/outbound
- No restrictions blocking SSH

✅ **6. Instance service check**
- `systemctl status sshd` → Active (running)
- `ss -tulpn | grep :22` → LISTEN on port 22

---

### Testing and verification

**Port reachability test:**
```bash
nc -vz -w 3 <public-ip> 22
```
✅ Connection succeeded

**SSH connection test:**
```bash
ssh -i my-key.pem ec2-user@<public-ip>
```
✅ Successfully connected

---

## Common Pitfalls

### "My IP" changes frequently

**Problem:**
- Dynamic IP from ISP changes
- Security Group rule becomes invalid
- SSH times out

**Solution:**
- Update Security Group source IP
- Or use broader CIDR (less secure)
- Or use VPN with static IP

---

### Wrong username for AMI

**Problem:**
- Using `root` or `admin` on Amazon Linux
- Authentication fails

**Solution:**

| AMI Type | Default Username |
|----------|------------------|
| Amazon Linux 2/2023 | `ec2-user` |
| Ubuntu | `ubuntu` |
| Red Hat | `ec2-user` |
| Debian | `admin` |
| CentOS | `centos` |

---

### SSH key permissions too open

**Problem:**
- Key file has 644 or 777 permissions
- SSH refuses to use key

**Solution:**
```bash
chmod 400 key.pem
```

---

### NACL blocking return traffic

**Problem:**
- Custom NACL allows inbound TCP 22
- Forgot outbound ephemeral ports
- Connection times out

**Solution:**
- Add outbound rule: TCP 1024-65535 to `0.0.0.0/0`
- Remember: NACL is stateless (both directions required)

---

## Cost Note

**This troubleshooting has no cost impact:**
- Security Group changes: Free
- NACL changes: Free
- Viewing instance details: Free

**Cost considerations:**
- Running instances incur hourly charges
- Stop instance when not needed (retains EBS, stops compute charges)
- Terminate instance when completely finished (deletes everything)

---

## Quick Reference Commands
```bash
# Test port reachability
nc -vz -w 3 <public-ip> 22

# SSH with specific key
ssh -i <key.pem> <username>@<public-ip>

# SSH with verbose output (debugging)
ssh -v -i <key.pem> <username>@<public-ip>

# Check key permissions
ls -l key.pem

# Fix key permissions
chmod 400 key.pem

# Get your current public IP
curl ifconfig.me
```
