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

### 1) Confirm public path exists

**Verify instance has public connectivity:**

**Check public IP:**
- Instance has public IPv4 address (or Elastic IP)
- Find in EC2 console: Instance Details → Public IPv4 address

**Check subnet routing:**
- Navigate to subnet's route table
- Verify route: `0.0.0.0/0 → igw-...`
- This proves internet routing exists

**Check Internet Gateway:**
- VPC has Internet Gateway attached
- IGW must be attached to same VPC as instance

---

### 2) Security Group (stateful)

**Instance-level firewall - check first for most SSH issues**

**Required inbound rules:**
- Protocol: TCP
- Port: 22
- Source: Your current public IP `/32`

**Common mistakes:**
- Wrong source IP (your public IP changed)
- Editing wrong security group (not attached to instance)
- Typo in CIDR block

**Outbound rules:**
- Usually allow all by default
- If restricted: return traffic automatic due to stateful behavior
- No special outbound rule needed for SSH

**How to find your public IP:**
- AWS Console has "My IP" option when adding rules
- Or use: `curl ifconfig.me`

---

### 3) NACL (stateless)

**Subnet-level firewall - check if timeout persists after SG fixes**

**If NACL is restrictive, must allow both directions:**

**Inbound rules:**
- Allow TCP 22 (SSH connection initiation)
- Source: Your public IP or `0.0.0.0/0`

**Outbound rules:**
- Allow TCP ephemeral ports (1024-65535)
- Destination: Your public IP or `0.0.0.0/0`
- Required for return traffic (NACL is stateless)

**Rule order matters:**
- Lower rule numbers evaluated first
- Explicit deny overrides later allows

---

### 4) Classify the symptom

**Use `nc` to test port reachability:**
```bash
nc -vz -w 3 <public-ip> 22
```

**Symptom interpretation:**

| Symptom | Likely Cause | Where to Investigate |
|---------|--------------|---------------------|
| **Timeout** | Network path issue | SG source IP wrong/missing, NACL blocking, routing issue, no public IP |
| **Connection refused** | Port closed or service down | sshd not running, OS firewall blocking, wrong port |
| **Connects but auth fails** | Credentials/permissions | Wrong username, wrong key, key permissions (should be 400 or 600) |

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
