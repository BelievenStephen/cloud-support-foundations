# AWS SAA Study Notes

## Mar 2, 2026 (Week 5)

### IAM Policy Evaluation + AccessDenied Triage

**Focus:**
- Understanding IAM policy structure
- Policy evaluation logic
- AccessDenied troubleshooting workflow

---

### IAM Policy Structure Review

**Lab exercise:** Examined AWS-managed policy `AmazonS3ReadOnlyAccess`

**Key JSON fields:**
- **Version:** Policy language version (usually `2012-10-17`)
- **Statement:** Array of permission statements
- **Effect:** `Allow` or `Deny`
- **Action:** API actions permitted/denied
- **Resource:** ARN(s) the statement applies to
- **Condition:** (Optional) Conditions for when statement applies

---

### AWS-Managed Policy Example

**AmazonS3ReadOnlyAccess analysis:**

**Wildcard actions:**
- `s3:Get*` - All Get operations
- `s3:List*` - All List operations

**Resource scope:**
- `Resource: "*"` - Applies to all S3 resources

**Result:** Grants broad read and list permissions across all S3 buckets

---

### Customer-Managed Policy Review

**Lab account custom policy:** `LabIamReadOnly`

**Allowed actions:**
- `iam:GetUser`
- `iam:ListUsers`
- `iam:ListRoles`

**Purpose:** Basic IAM read/list permissions for lab users

---

### Lab Principal Configuration

**Active IAM user:** `stephen-admin`

**Attached policies:**
- `AdministratorAccess` (AWS-managed)

**Group membership:**
- `Admins` group

**Permissions boundary:**
- None set
- No guardrails restricting allowed actions

---

### IAM Policy Simulator Testing

**Test performed:**
- Principal: `stephen-admin`
- Action: `s3:ListBucket`
- Resource: `arn:aws:s3:::stephen-s3-perms-<DATE>`

**Result:** Allowed

**Allowing policy:** `AdministratorAccess`
- Action: `"*"`
- Resource: `"*"`
- Effect: Allow

**Explanation:** Administrator access allows all actions on all resources

---

### IAM Policy Evaluation Logic

**Default behavior:**
- Default deny (implicit deny)
- Explicit allow required for access
- Explicit deny always wins

**Evaluation order:**
1. Check for explicit deny (any source)
2. If no deny, check for explicit allow
3. If no allow, implicit deny applies

**Key principle:** When troubleshooting AccessDenied, look for Deny statements first

---

### Sources of Permissions

**Identity-based policies:**
- Attached to users, groups, or roles
- Defines what the identity can do

**Resource-based policies:**
- Attached to resources (e.g., S3 bucket policy)
- Defines who can access the resource

**Guardrails (restrictions):**
- Permissions boundaries
- Service Control Policies (SCPs)
- Session policies (for assumed roles)

---

### Critical S3 ARN Pattern Distinction

**Bucket-level operations:**
```
Action: s3:ListBucket
Resource: arn:aws:s3:::bucket-name
```

**Object-level operations:**
```
Action: s3:GetObject
Resource: arn:aws:s3:::bucket-name/*
```

**Common mistake:** Using bucket ARN for object operations or vice versa

---

### Support Mapping: S3 AccessDenied

**Symptoms:**
- 403 AccessDenied error
- 404 NotFound error (may also indicate permission issue)

---

**Troubleshooting workflow:**

**Step 1: Confirm caller identity**
```bash
aws sts get-caller-identity
```

**Verify:**
- IAM user or role ARN
- AWS account ID
- Whether using assumed role session

---

**Step 2: Confirm target resource and action**

**Identify:**
- Bucket name
- Object key (if applicable)
- API operation (`ListBucket`, `GetObject`, `PutObject`)

---

**Step 3: Check identity policy**

**Verify action is allowed on correct resource ARN:**

| Operation | Required Action | Resource ARN |
|-----------|----------------|--------------|
| List bucket | `s3:ListBucket` | `arn:aws:s3:::bucket-name` |
| Get object | `s3:GetObject` | `arn:aws:s3:::bucket-name/*` |
| Put object | `s3:PutObject` | `arn:aws:s3:::bucket-name/*` |

---

**Step 4: Check bucket policy**

**S3 Console → Bucket → Permissions → Bucket policy**

**Look for:**
- Explicit deny to this principal
- Allow statement for this principal
- Condition blocks that may restrict access

---

**Step 5: Check Block Public Access settings**

**If testing public access patterns:**
- S3 Console → Bucket → Permissions → Block public access
- All four settings should typically be ON

---

**Step 6: Check guardrails (if applicable)**

**Service Control Policies (SCPs):**
- If in an AWS Organization
- Check organization-level restrictions

**Permissions boundary:**
- IAM Console → User/Role → Permissions boundary
- Defines maximum allowed permissions

**Session policy:**
- If using assumed role
- Passed during `AssumeRole` call

---

### 403 vs 404 Error Interpretation

**403 AccessDenied:**
- Explicitly not authorized
- Missing allow in policy
- Explicit deny in policy

**404 NotFound:**
- Object doesn't exist, OR
- Not allowed to know object exists (permissions-based obfuscation)
- Behavior depends on how request is made

---

### Verification

**After implementing fix:**
```bash
# Test list bucket
aws s3 ls s3://bucket-name/

# Test get object
aws s3 cp s3://bucket-name/object-key /tmp/test
```

**Expected:** Successful operation with no errors

---

### Support Mapping: IAM Permission Denied

**Symptom:** "User is not authorized to perform: [action] on resource: [ARN]"

---

**Troubleshooting workflow:**

**Step 1: Extract details from error**
- Principal (user/role)
- Exact action being attempted
- Resource ARN

---

**Step 2: Check for explicit deny (first priority)**

**Sources to check:**
1. Identity policy Deny statements
2. Resource policy Deny statements
3. Permissions boundary Deny
4. SCP Deny effect
5. Session policy Deny (if assumed role)

**Why check deny first:** Explicit deny always wins, overrides all allows

---

**Step 3: If no deny, find missing allow**

**Common issues:**

| Issue | Check |
|-------|-------|
| **Action not included** | Policy includes wrong action or missing wildcard |
| **Wrong resource ARN** | Bucket ARN used for object operation (or vice versa) |
| **Condition blocking** | MFA required, IP restriction, VPC endpoint required |

---

**Step 4: Use Policy Simulator**

**IAM Console → Policy Simulator**

**Test with:**
- Principal (user/role)
- Action (exact API action)
- Resource (exact ARN from error)

**Review:**
- Result: Allowed or Denied
- Which policy allowed/denied
- Any conditions evaluated

---

### Verification

**After policy update:**
- Repeat exact failing action
- Confirm success (no error)
- Verify in CloudTrail (optional)

---

### AccessDenied Troubleshooting Workflow

**Systematic approach:**

1. **Who am I?**
   - `aws sts get-caller-identity`
   - Confirm principal, account, role session

2. **What action/resource?**
   - Extract from error message
   - Verify correct ARN format

3. **Identity policy allows?**
   - Check attached policies
   - Verify action and resource match

4. **Any explicit deny?**
   - Identity policy
   - Resource policy
   - Permissions boundary
   - SCPs

5. **Resource policy allows?**
   - Check bucket policy (for S3)
   - Verify principal not blocked

6. **Guardrails blocking?**
   - Permissions boundary
   - SCPs (if in organization)
   - Session policy (if assumed role)

7. **Conditions blocking?**
   - MFA required
   - IP address restrictions
   - VPC endpoint required
   - Time-based restrictions

8. **Re-test:**
   - Policy Simulator
   - Actual API call

---

### Key Takeaways

**IAM policy structure:**
- Version, Statement, Effect, Action, Resource, Condition
- Statement is array of individual permission rules

**Policy evaluation logic:**
- Default deny (implicit)
- Explicit allow required
- Explicit deny always wins (check deny first)

**Permission sources:**
- Identity policies (user/role/group)
- Resource policies (S3 bucket, etc.)
- Guardrails (boundaries, SCPs, session policies)

**S3 ARN patterns:**
- Bucket operations: `arn:aws:s3:::bucket-name`
- Object operations: `arn:aws:s3:::bucket-name/*`
- Using wrong ARN is common mistake

**AccessDenied triage order:**
1. Confirm caller identity
2. Identify action and resource
3. Check identity policy allows
4. Check for explicit denies (any source)
5. Check resource policy
6. Check guardrails (boundary/SCP)
7. Check conditions
8. Test in Policy Simulator
9. Retry actual operation

**403 vs 404:**
- 403 = clearly not authorized
- 404 = object missing OR not allowed to know it exists

**Troubleshooting tools:**
- `aws sts get-caller-identity` - Who am I?
- IAM Policy Simulator - Test permissions before making changes
- CloudTrail - Track permission changes

---

## Mar 2, 2026 (continued on Mar 3)

### Lab Exercise: Explicit Deny in Bucket Policy

**Objective:** Demonstrate that explicit deny overrides all allows, including AdministratorAccess

---

### Baseline Verification

**Confirmed caller identity:**
```bash
aws sts get-caller-identity
```

**Result:**
- ARN: `arn:aws:iam::<ACCOUNT_ID>:user/stephen-admin`
- Attached policy: AdministratorAccess

**Tested baseline access:**
```bash
aws s3 ls s3://stephen-s3-perms-<DATE>
```
**Result:** Success - bucket listing returned

---

### Test Object Creation

**Created test object:**
```bash
echo "test content" > s3-accessdenied-lab.txt
aws s3 cp s3-accessdenied-lab.txt s3://stephen-s3-perms-<DATE>/
```

**Verification:**
- Object uploaded successfully
- Visible in bucket listing

---

### Breaking Change: Add Explicit Deny

**Bucket policy applied:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Principal": {
        "AWS": "arn:aws:iam::<ACCOUNT_ID>:user/stephen-admin"
      },
      "Action": "s3:ListBucket",
      "Resource": "arn:aws:s3:::stephen-s3-perms-<DATE>"
    }
  ]
}
```

**Target:**
- Action: `s3:ListBucket`
- Resource: Bucket ARN (not object ARN)
- Principal: `stephen-admin`

---

### Testing After Deny

**Attempted bucket listing:**
```bash
aws s3 ls s3://stephen-s3-perms-<DATE>
```

**Error returned:**
```
An error occurred (AccessDenied) when calling the ListObjectsV2 operation: 
Access Denied with an explicit deny in a resource-based policy
```

**Key observation:**
- AdministratorAccess attached to user
- Identity policy allows all S3 actions
- Bucket policy explicit deny **overrides** identity policy allow

---

### Resolution Applied

**Removed bucket policy:**
- S3 Console → Bucket → Permissions → Bucket policy
- Deleted entire policy document
- Saved changes

**Result:** Bucket policy section now empty

---

### Verification After Fix

**Re-tested bucket listing:**
```bash
aws s3 ls s3://stephen-s3-perms-<DATE>
```

**Result:** Success
- Bucket listing returned normally
- Object `s3-accessdenied-lab.txt` visible

---

### Cleanup

**Removed test object:**
```bash
aws s3 rm s3://stephen-s3-perms-<DATE>/s3-accessdenied-lab.txt
```

**Verification:**
- Object deleted
- Bucket listing no longer shows test object

---

### Key Observations

**Explicit deny behavior:**
- Overrides all allows from any source
- Includes AdministratorAccess policy
- Resource policy (bucket policy) can block identity policy permissions

**Error message clarity:**
- Error explicitly stated "explicit deny in a resource-based policy"
- Directs investigation to bucket policy, not identity policy

**Policy evaluation order proven:**
1. Check for explicit deny (found in bucket policy)
2. Deny matched → access blocked
3. Identity policy allows never evaluated

**Troubleshooting lesson:**
- When AdministratorAccess user gets AccessDenied, check resource policies
- Look for explicit deny statements in bucket policies, key policies, etc.
- Explicit deny is the first thing evaluated, always wins

---

### Lab Workflow Summary

| Step | Action | Result |
|------|--------|--------|
| 1. Baseline | `aws s3 ls` with AdministratorAccess | ✅ Success |
| 2. Add deny | Bucket policy denies ListBucket | ❌ AccessDenied |
| 3. Remove deny | Delete bucket policy | ✅ Success |
| 4. Cleanup | Delete test object | ✅ Complete |

**Principle reinforced:** Explicit deny > Allow (regardless of policy source or breadth)

---

## Mar 4, 2026

### EC2 SSH Access Troubleshooting

**Focus:**
- SSH connectivity debugging
- Network path verification
- Timeout vs refused interpretation

---

### Lab Instance Configuration Review

**Instance details:**
- Instance ID: `i-<INSTANCE_ID>`
- Name: `lab-unreachable`
- Region: us-west-1
- Availability Zone: us-west-1c

**Network configuration:**
- VPC: `vpc-<VPC_ID>`
- Subnet: `subnet-<SUBNET_ID>`
- Security Group: `sg-<SECURITY_GROUP_ID>`
- Key pair: `lab-unreachable-key-pair`

---

### Public Connectivity Verification

**Public IPv4 address:**
- Current: `<PUBLIC_IP>` (54.x.x.x range)
- Assigned: Yes

**Subnet public access setting:**
- Auto-assign public IPv4: Yes
- Supports direct internet connectivity

---

### Subnet Classification Verification

**Route table check:**
- Associated route table: `rtb-<ROUTE_TABLE_ID>`

**Routes present:**
```
Destination          Target
172.31.0.0/16    →  local
0.0.0.0/0        →  igw-<IGW_ID>
```

**Classification:** Public subnet (IGW route present)

---

### Security Group Analysis

**Current SSH rule:**
- Protocol: TCP
- Port: 22
- Source: `<SPECIFIC_IP>/32`

**Implication:**
- SSH only allowed from single IP address
- If client IP changes, SSH will timeout
- Most common cause of "worked yesterday, fails today"

---

### Network ACL Verification

**Associated NACL:** `acl-<NACL_ID>`

**Rules:**
```
Rule #    Type       Protocol    Port Range    Source/Dest       Allow/Deny
100       All traffic All        All           0.0.0.0/0         ALLOW
*         All traffic All        All           0.0.0.0/0         DENY
```

**Interpretation:**
- Inbound: Allow all (rule 100)
- Outbound: Allow all (rule 100)
- NACL not blocking SSH in this configuration

---

### Status Checks Review

**System status check:** Passed
- AWS infrastructure health
- Hypervisor, network, power

**Instance status check:** Passed
- Guest OS health
- Network configuration
- Software issues

**Conclusion:** Network path and instance health confirmed good

---

### Support Mapping: EC2 SSH Connection Failed

**Symptoms:**
- SSH connection timeout
- SSH connection refused
- "Permission denied" errors

---

### Symptom Classification

**SSH timeout:**
- Connection hangs, then times out
- No response from instance

**Common causes:**
- Security Group blocking (wrong source IP)
- Instance has no public IP
- Route table missing IGW route
- Wrong target IP address

---

**SSH connection refused:**
- Fast failure, explicit refusal
- Instance reachable but port closed

**Common causes:**
- SSH daemon not running
- Port 22 not listening
- Host-based firewall blocking
- Instance stopped or terminated

---

**Permission denied:**
- Connection established but auth failed

**Common causes:**
- Wrong SSH key
- Wrong username
- Key permissions incorrect (should be 400)

---

### Troubleshooting Workflow: SSH Timeout

**Step 1: Verify instance state**
```
EC2 Console → Instances → Check instance state
```
- Must be "running"
- Not "stopped", "stopping", or "terminated"

---

**Step 2: Verify public IP exists**
```
EC2 Console → Instance → Networking tab → Public IPv4 address
```
- If no public IP: Cannot SSH from internet
- If public IP present: Proceed to next step

---

**Step 3: Verify subnet is public**
```
VPC Console → Subnets → Select subnet → Route table
VPC Console → Route tables → Routes tab
```

**Required route:**
```
0.0.0.0/0 → igw-...
```

**If missing:** Subnet is private, add IGW route or use bastion host

---

**Step 4: Verify Security Group allows SSH**
```
EC2 Console → Instance → Security tab → Security groups → Inbound rules
```

**Required rule:**
```
Type: SSH
Protocol: TCP
Port: 22
Source: <YOUR_CURRENT_IP>/32 or 0.0.0.0/0
```

**Common issue:** Source IP is outdated (your IP changed)

**Fix:** Update rule to "My IP" to automatically use current IP

---

**Step 5: Check Network ACL (if custom)**
```
VPC Console → Network ACLs → Select subnet's NACL
```

**Required:**
- Inbound: Allow TCP 22
- Outbound: Allow ephemeral ports (1024-65535)

**Note:** Default NACL allows all traffic

---

**Step 6: Verify status checks**
```
EC2 Console → Instance → Status checks tab
```

**Both must pass:**
- System status check
- Instance status check

**If failed:** Investigate instance health or reboot

---

### Troubleshooting Workflow: "Worked Yesterday, Fails Today"

**Common causes and checks:**

---

**1) Instance public IP changed:**

**When this happens:**
- Instance stopped and started (not rebooted)
- Elastic IP not associated

**Check:**
```
EC2 Console → Instance → Networking → Public IPv4 address
```

**Compare with:** IP address you're trying to SSH to

**Fix:** SSH to new IP, or associate Elastic IP for persistence

---

**2) Your client IP changed:**

**When this happens:**
- ISP changed your public IP
- Switched networks (home to office)
- Connected to VPN

**Check current IP:**
```bash
curl ifconfig.me
```

**Compare with:** Security Group inbound rule source

**Fix:** Update Security Group rule to current IP

---

**3) Security Group rule drift:**

**What to check:**
```
EC2 Console → Security Groups → Select SG → Inbound rules
```

**Verify:**
- SSH rule still exists
- Port is 22
- Protocol is TCP
- Source includes your IP

**Fix:** Re-add or correct the rule

---

**4) SSH key mismatch:**

**Key pair assigned at launch:** `lab-unreachable-key-pair`

**Verify you're using:**
```bash
ssh -i /path/to/lab-unreachable-key-pair.pem ec2-user@<PUBLIC_IP>
```

**Common mistakes:**
- Using wrong key file
- Key file permissions incorrect (must be 400)

**Fix:**
```bash
chmod 400 /path/to/key.pem
ssh -i /path/to/correct-key.pem ec2-user@<PUBLIC_IP>
```

---

**5) Username mismatch:**

**AMI-dependent usernames:**

| AMI Type | Default Username |
|----------|-----------------|
| Amazon Linux | `ec2-user` |
| Ubuntu | `ubuntu` |
| Debian | `admin` or `debian` |
| RHEL | `ec2-user` or `root` |
| CentOS | `centos` |
| Fedora | `fedora` |

**Lab instance:** Amazon Linux → use `ec2-user`

---

**6) Instance-side issues:**

**Less common, check if above all pass:**

**SSH daemon not running:**
```bash
# From Systems Manager Session Manager or serial console
sudo systemctl status sshd
```

**Disk full:**
```bash
df -h
```

**Host firewall:**
```bash
sudo iptables -L -n | grep 22
```

---

### SSH Connection Testing

**Basic connection test:**
```bash
ssh -i /path/to/key.pem ec2-user@<PUBLIC_IP>
```

**Verbose output for debugging:**
```bash
ssh -v -i /path/to/key.pem ec2-user@<PUBLIC_IP>
```

**Connection timeout test:**
```bash
nc -zv -w 5 <PUBLIC_IP> 22
```

**Expected:** Connection succeeded

---

### Verification After Fix

**Successful SSH connection:**
```bash
ssh -i lab-unreachable-key-pair.pem ec2-user@<PUBLIC_IP>
```

**Expected output:**
```
Last login: [timestamp]
[ec2-user@ip-... ~]$
```

**Connection established, shell prompt available**

---

### SSH Triage Order Summary

**Systematic checklist:**

1. **Instance running?**
   - Check instance state in console

2. **Has public IP?**
   - Check networking tab for public IPv4

3. **Subnet is public?**
   - Route table has `0.0.0.0/0 → igw-...`

4. **Security Group allows SSH?**
   - Inbound rule: TCP 22 from your IP

5. **NACL allows traffic?**
   - Check inbound/outbound rules (if custom)

6. **Status checks pass?**
   - Both system and instance checks green

7. **Using correct key?**
   - Key pair matches instance

8. **Using correct username?**
   - AMI-appropriate username

9. **SSH daemon running?**
   - Check via Session Manager if accessible

---

### Timeout vs Refused Interpretation

| Symptom | Network Layer | Common Cause | First Check |
|---------|--------------|--------------|-------------|
| **Timeout** | Network/routing | Security Group source mismatch | SG inbound rules |
| **Timeout** | Network/routing | No public IP | Instance networking |
| **Timeout** | Network/routing | Private subnet (no IGW) | Route table |
| **Refused** | Transport/application | SSH daemon not running | Service status |
| **Refused** | Transport/application | Port not listening | `ss -tulpn` |
| **Refused** | Transport/application | Host firewall blocking | Firewall rules |

---

### Common Resolution Patterns

| Issue | Symptom | Resolution |
|-------|---------|------------|
| **IP changed** | Timeout after working before | Update SG source to current IP |
| **Wrong key** | Permission denied | Use correct key pair |
| **Wrong username** | Permission denied | Use AMI-appropriate username |
| **No public IP** | Timeout | Associate Elastic IP |
| **Private subnet** | Timeout | Add IGW route or use bastion |
| **SG too restrictive** | Timeout | Add rule for SSH from your IP |

---

### Key Takeaways

**SSH timeout indicators:**
- Security Group source IP doesn't match
- No public IP on instance
- Subnet not public (missing IGW route)
- Wrong target IP address

**SSH refused indicators:**
- Instance reachable but SSH daemon down
- Port 22 not listening
- Host-based firewall blocking
- Service misconfiguration

**"Worked yesterday" checklist:**
- Instance public IP changed (stop/start)
- Your client IP changed (ISP/network switch)
- Security Group rule modified or removed
- Using wrong SSH key
- Using wrong username for AMI

**SSH triage priority:**
1. Instance state (running)
2. Public IP exists
3. Subnet has IGW route
4. Security Group allows TCP 22 from source
5. NACL allows (if custom)
6. Status checks pass
7. Correct key and username
8. SSH daemon running

**Most common issues:**
- Security Group source IP mismatch (timeout)
- Wrong SSH key (permission denied)
- Wrong username (permission denied)

**Quick fixes:**
- Update SG rule to "My IP" for automatic current IP
- Associate Elastic IP to prevent IP changes on stop/start
- Use verbose SSH for detailed connection debugging

---

## Mar 4, 2026 (continued on Mar 5)

### Lab Exercise: Security Group SSH Blocking

**Objective:** Demonstrate how Security Group source IP mismatch blocks SSH access

---

### Baseline Verification

**Established working SSH connection:**
```bash
ssh -i lab-unreachable-key-pair.pem ec2-user@<PUBLIC_IP>
```

**Result:** Successfully connected

**Security Group configuration:**
- Security Group: `sg-<SECURITY_GROUP_ID>`
- Inbound SSH rule: TCP/22 from `<MY_IP>/32`

---

### Breaking Change Applied

**Modified Security Group inbound rule:**
- Original source: `<MY_IP>/32`
- Changed to: `<WRONG_IP>/32` (off by one digit)
- Result: SSH source no longer matches actual client IP

---

### Testing After Change

**Attempted SSH connection:**
```bash
ssh -i lab-unreachable-key-pair.pem ec2-user@<PUBLIC_IP>
```

**Error returned:**
```
ssh: connect to host <PUBLIC_IP> port 22: Operation timed out
```

**Key observation:**
- Connection timeout, not connection refused
- Indicates network path blocked (Security Group)
- SSH daemon running but Security Group filtering packets

---

### Resolution Applied

**Restored correct Security Group rule:**
- Security Group: `sg-<SECURITY_GROUP_ID>`
- Inbound rule: TCP/22 from correct source `<MY_IP>/32`
- Method: EC2 Console → Security Groups → Edit inbound rules

---

### Verification After Fix

**Re-tested SSH connection:**
```bash
ssh -i lab-unreachable-key-pair.pem ec2-user@<PUBLIC_IP>
```

**Result:** Successfully connected
- Connection established immediately
- Authentication succeeded
- Shell prompt available

---

### Cleanup Verification

**Final Security Group state:**
- Inbound SSH rule source: `<MY_IP>/32`
- Access remains restricted to single IP
- No overly permissive rules (not `0.0.0.0/0`)

---

### Key Observations

**Timeout vs refused behavior:**
- Wrong SG source IP → timeout (packets dropped)
- SSH daemon down → connection refused (packets reach host, port closed)

**Security Group is stateful:**
- Only inbound rule needed
- Return traffic automatically allowed
- No outbound rule modification required

**Single digit difference matters:**
- `<MY_IP>/32` works
- `<WRONG_IP>/32` (one digit off) causes complete timeout
- CIDR `/32` means exact IP match required

**Troubleshooting lesson:**
- Timeout to SSH usually indicates Security Group issue
- First check: Compare current public IP with SG rule source
- Quick fix: Use "My IP" option in console for automatic current IP

---

### Lab Workflow Summary

| Step | Action | Result |
|------|--------|--------|
| 1. Baseline | SSH with correct SG rule | ✅ Success |
| 2. Break | Change SG source to wrong IP | ❌ Timeout |
| 3. Fix | Restore correct source IP | ✅ Success |
| 4. Cleanup | Verify restricted access maintained | ✅ Complete |

**Principle reinforced:** Security Group source IP must exactly match client's public IP for `/32` rules

---

### Real-World Scenario

**"Worked yesterday, fails today" root cause:**
1. ISP assigned you new public IP
2. Security Group still has yesterday's IP
3. Result: SSH times out
4. Fix: Update Security Group to current IP

**Prevention:**
- Use Elastic IP for instances requiring consistent addressing
- Use broader CIDR if multiple known IPs needed (e.g., office range)
- Use "My IP" in console when adding rules (auto-updates)

---

## Mar 6, 2026

## VPC routing-first triage

### Lab instance review

**Instance examined:** `lab-unreachable`

**Network configuration captured:**
- Subnet ID: `subnet-0ed1fa4a40bc62a26`
- Route table: `rtb-0f874e22f221b539f`
- Security group: `sg-083e0204d570303c6`
- Public IPv4: `<REDACTED>`
- Private IPv4: `172.31.12.41`
- Availability Zone: `us-west-1c`

---

### Public subnet verification

**Route table analysis:**
- Confirmed subnet is public-routable
- Route table contains:
  - `0.0.0.0/0 → igw-0d2b4afba531a9b3b` (internet gateway route)
  - `172.31.0.0/16 → local` (VPC local route)

**Subnet settings:**
- Auto-assign public IPv4: **Yes**
- Supports launching instances with internet reachability (when SGs allow)

---

### Subnet type identification rules

**Public subnet proven by:**
- Route table has `0.0.0.0/0 → igw-...`
- Instance has public IPv4 or Elastic IP

**Private subnet with egress proven by:**
- Route table has `0.0.0.0/0 → nat-...` (NOT IGW)
- Instances usually have no public IP

---

### IGW egress requirements

**Critical requirement:**
- Instance MUST have public IPv4 or Elastic IP
- Without public IP, instance cannot use IGW for outbound internet

---

### NAT Gateway placement rules

**NAT Gateway requirements:**
- Must live in public subnet
- NAT's subnet must route `0.0.0.0/0 → IGW`
- Must have Elastic IP assigned

---

### Routing-first triage order

**Systematic troubleshooting approach:**

1. **Subnet ID** → Identify which subnet instance is in
2. **Route table default route target** → Check `0.0.0.0/0` destination:
   - IGW (public path)
   - NAT (private with egress)
   - Missing (no internet route)
3. **Public IPv4 presence** → If IGW path, verify instance has public IP
4. **Security Group egress** → Check outbound rules
5. **NACL** → Check stateless rules (both directions)
6. **DNS** → Test with `dig`
7. **HTTP** → Test with `curl`

---

### Common "egress broken" failure modes

**Typical issues encountered:**

| Failure Mode | Symptom |
|--------------|---------|
| **Missing `0.0.0.0/0` route** | No default route in route table |
| **Wrong target** | Deleted/detached IGW or deleted NAT Gateway |
| **Wrong subnet expectation** | Private instance trying to use IGW without public IP |
| **SG/NACL blocking** | Firewall rules blocking traffic |

---

### Key learnings summary

**Routing fundamentals:**
- Route table determines path (IGW vs NAT vs local only)
- Public IP required for IGW path
- NAT Gateway enables private instances to reach internet
- NAT Gateway itself must be in public subnet with EIP

**Troubleshooting strategy:**
- Start with routing (route table inspection)
- Verify path requirements (public IP for IGW, NAT for private)
- Then check firewalls (SG, NACL)
- Finally test connectivity (DNS, HTTP)

---

## Mar 6, 2026 (continued)

### Practice Exam Lessons (Tutorials Dojo Free Sampler)

**Focus:**
- Identifying knowledge gaps
- Service selection patterns
- Common AWS exam traps

---

### Streaming with ordering requirements

**Question pattern:** Durable, no loss, no duplicates, in-order processing

**Correct choice: Kinesis Data Streams**
- Guarantees ordering per shard (using partition key)
- Durable (data replicated across AZs)
- No data loss
- Exactly-once processing possible with proper consumer design

**Why not SQS Standard:**
- Does NOT guarantee ordering
- Can deliver duplicates (at-least-once delivery)
- Best-effort ordering only

**Key distinction:** Need ordering → Kinesis Data Streams (with partition key)

---

### Load balancer Layer 7 features

**Question pattern:** Host-based routing, path-based routing, gRPC support

**Correct choice: Application Load Balancer (ALB)**
- Operates at Layer 7 (HTTP/HTTPS)
- Host-based routing: Route by `Host` header
- Path-based routing: Route by URL path
- gRPC support (HTTP/2)
- WebSocket support

**Why not Network Load Balancer (NLB):**
- Operates at Layer 4 (TCP/UDP)
- Does NOT have Layer 7 routing rules
- No host/path inspection
- Use for: TCP/UDP traffic, extreme performance, static IP

**Key distinction:** Need L7 features → ALB; Need L4 performance → NLB

---

### S3 cost optimization patterns

**Question pattern:** Automatically move old objects to cheaper storage

**Correct choice: S3 Lifecycle rules**
- Automatically transition objects between storage classes
- Based on age (days since creation)
- Example: S3 Standard → S3 Standard-IA → S3 Glacier after X days

**Common lifecycle patterns:**

| Age | Action |
|-----|--------|
| 30 days | Transition to S3 Standard-IA |
| 90 days | Transition to S3 Glacier Flexible Retrieval |
| 180 days | Transition to S3 Glacier Deep Archive |
| 365 days | Delete object (if appropriate) |

**Key takeaway:** Lifecycle rules = automated storage class transitions

---

### Multi-domain HTTPS on ALB

**Question pattern:** Serve multiple domains (different root domains) on single ALB with HTTPS

**Correct choice: SNI (Server Name Indication)**
- Attach multiple TLS certificates to single ALB HTTPS listener
- ALB uses SNI to select correct certificate based on requested hostname
- Client sends hostname in TLS handshake

**Why not wildcards:**
- `*.example.com` only covers subdomains of example.com
- Does NOT cover different root domains (example.com, another.com)

**Why not SAN certificates:**
- Subject Alternative Names can cover multiple domains
- BUT requires reissuing certificate when adding new domains
- Not as flexible as SNI with multiple certs

**Key distinction:** Multiple root domains → SNI with multiple certs on single listener

---

### Shared session storage requirements

**Question pattern:** Sub-millisecond latency, shared across many instances, session data

**Correct choice: ElastiCache**
- In-memory data store
- Sub-millisecond latency
- Shared across all instances
- Two engines: Redis or Memcached

**Memcached advantages:**
- Multithreaded (better CPU utilization)
- Auto Discovery (automatic node replacement)
- Simpler architecture
- Good for: Simple caching, session storage

**Redis advantages:**
- Persistence
- Replication
- Advanced data structures
- Pub/sub
- Good for: Complex caching, leaderboards, real-time analytics

**Why not sticky sessions:**
- Sticky sessions pin user to specific instance
- Does NOT meet "shared session store" requirement
- Session data lost if instance fails

**Key distinction:** Shared session store → ElastiCache (Memcached for simplicity, Redis for persistence)

---

### Service quota monitoring

**Question pattern:** Monitor AWS service limits, automated notifications

**Correct choice: Trusted Advisor Service Limits check**
- Monitors service quotas
- Scheduled Lambda refresh (every 24 hours)
- Notifications via EventBridge → SNS

**Architecture:**
```
Trusted Advisor Service Limits check
    ↓
EventBridge (scheduled, e.g., daily)
    ↓
Lambda (refresh Trusted Advisor)
    ↓
EventBridge (Trusted Advisor check result)
    ↓
SNS (notify on approaching limits)
```

**Why not AWS Config:**
- Config rules designed for compliance monitoring
- Not optimized for quota/limit monitoring
- More expensive for this use case

**Key distinction:** Service quota monitoring → Trusted Advisor + EventBridge + Lambda

---

### Private traffic between VPCs (different regions)

**Question pattern:** VPC-to-VPC communication across regions, private traffic

**Correct choice: Inter-region VPC peering**
- Direct network connection between VPCs in different regions
- Traffic stays on AWS backbone (never traverses internet)
- Requires route table updates in both VPCs
- No single point of failure
- No bandwidth bottleneck

**Required configuration:**
1. Create VPC peering connection
2. Accept peering connection in peer region
3. Update route tables in both VPCs to point to peering connection
4. Update Security Groups to allow traffic from peer CIDR

**Why not NAT Gateway:**
- NAT Gateway for outbound internet access
- NOT for VPC-to-VPC communication

**Why not VPC Endpoints:**
- VPC Endpoints for AWS services (S3, DynamoDB, etc.)
- NOT for inter-region VPC-to-VPC traffic

**Key distinction:** Inter-region VPC-to-VPC → VPC peering + route table updates

---

### EC2 hibernation for faster resume

**Question pattern:** Quick resume for Windows apps, no extra compute cost

**Correct choice: EC2 hibernation**
- Saves RAM contents to EBS
- Faster resume than cold start
- No compute charges while hibernated (only EBS charges)

**Critical limitation:**
- Must enable hibernation AT LAUNCH
- CANNOT enable after instance created

**Hibernation vs Stop:**

| Attribute | Stop | Hibernate |
|-----------|------|-----------|
| **RAM contents** | Lost | Saved to EBS |
| **Resume time** | Slower (boot OS) | Faster (restore RAM) |
| **When to use** | General stopping | Quick resume needed |
| **Configure when** | Anytime | Must be at launch |

**Key takeaway:** Need quick resume → Plan for hibernation at launch

---

### Block country access behind ALB

**Question pattern:** Block entire country from accessing application

**Correct choice: AWS WAF geo match condition**
- Create Web ACL with geo match rule
- Associate Web ACL with ALB
- Block or allow based on country of origin

**Configuration:**
1. Create Web ACL in AWS WAF
2. Add geo match condition (select countries to block)
3. Set rule action to "Block"
4. Associate Web ACL with ALB

**Why not NACL:**
- Country-based blocking via NACL requires maintaining IP range lists
- Hard to maintain (IP ranges change)
- Not recommended approach

**Why not Security Group:**
- Security Groups don't support geo-blocking
- Only IP-based rules

**Key distinction:** Geo-blocking → AWS WAF geo match + Web ACL on ALB

---

### Immutable long-term log retention

**Question pattern:** Tamper-resistant, long-term retention (e.g., 5 years), compliance

**Correct choice: S3 Glacier Vault Lock**
- Enforces immutable retention policy
- Cannot be changed once locked (even by root account)
- Compliance mode: Cannot delete or modify for retention period
- Common use: Regulatory compliance (SEC, FINRA, HIPAA)

**Vault Lock workflow:**
1. Initiate Vault Lock with retention policy
2. Test during 24-hour window (can abort)
3. Complete Vault Lock (becomes immutable)
4. Policy cannot be changed or deleted

**Why not S3 MFA Delete:**
- MFA Delete requires MFA to delete objects
- Does NOT guarantee immutability
- Root account with MFA can still delete
- Not sufficient for compliance requirements

**Why not S3 Object Lock:**
- S3 Object Lock is for S3 Standard/IA
- Question specifically about Glacier compliance

**Key distinction:** Compliance-grade immutability → S3 Glacier Vault Lock

---

### IAM policy evaluation with conditions

**Question pattern:** Deny specific IPs while allowing all others

**Policy evaluation reminder:**
- **Explicit Deny always wins** (overrides all allows)
- Check for deny before allow

**Condition example:**
```json
{
  "Effect": "Deny",
  "Action": "s3:*",
  "Resource": "*",
  "Condition": {
    "IpAddress": {
      "aws:SourceIp": "203.0.113.0/24"
    }
  }
}
```

**This policy:**
- Denies all S3 actions
- Only when source IP is in 203.0.113.0/24 range
- All other IPs unaffected (evaluated by other statements)

**Key principle:** Explicit deny + condition = conditional blocking

---

### Practice exam key patterns

**Service selection patterns learned:**

| Requirement | Correct Service |
|-------------|----------------|
| Ordering + no duplicates | Kinesis Data Streams |
| Layer 7 routing | ALB |
| Auto storage transition | S3 Lifecycle |
| Multi-domain HTTPS | SNI on ALB |
| Shared sessions | ElastiCache |
| Service quota alerts | Trusted Advisor + EventBridge |
| Inter-region VPC traffic | VPC Peering |
| Quick resume | EC2 Hibernation (plan at launch) |
| Geo-blocking | AWS WAF |
| Immutable retention | Glacier Vault Lock |

**Common exam traps:**
- SQS Standard does NOT guarantee ordering
- NLB does NOT have Layer 7 features
- Sticky sessions ≠ shared session store
- NAT Gateway NOT for VPC-to-VPC
- MFA Delete NOT sufficient for compliance immutability
- Hibernation must be enabled at launch

---
