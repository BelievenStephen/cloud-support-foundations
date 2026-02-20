# AWS SAA Study Notes

## Feb 16, 2026 (Week 3)

### IAM Policy Evaluation Deep Dive

**Principal verification:**
- Confirmed current identity: IAM user `stephen-admin` (not an assumed role)
- Primary permission source: Managed policy `AdministratorAccess` attached to user
- Group membership: `Admins` group (secondary permission source)
- Permission boundary: None set (no identity-level maximum restrictions)

---

### Policy Simulator Testing

**Tested actions in IAM Policy Simulator:**
- `iam:GetUser` = **Allowed** (decision path: AdministratorAccess)
- `ec2:DescribeInstances` = **Allowed** (decision path: AdministratorAccess)
- `s3:ListBucket` = **Allowed** (decision path: AdministratorAccess)

**Key observation:** All permissions traced back to AdministratorAccess policy, confirming it as the primary grant.

---

### Policy Evaluation Rules Confirmed

**Explicit deny always wins:**
- Even with AdministratorAccess attached, an explicit `Deny` in any policy layer will override the `Allow`
- Deny sources to check: identity policies (user/group), permission boundaries, SCPs

**Policy evaluation order:**
1. Explicit deny (any layer) → **Access denied**
2. Explicit allow (identity/resource policy) → **Access allowed**
3. No explicit allow → **Implicit deny** (access denied)

---

### Identity Policy vs Resource Policy

**Identity-based policy:**
- Attached to: Users, groups, or roles
- Grants permissions: "This principal can perform these actions"
- Example: AdministratorAccess attached to `stephen-admin`

**Resource-based policy:**
- Attached to: AWS resources (S3 buckets, KMS keys, etc.)
- Controls access: "Who can access this resource and how"
- Example: S3 bucket policy allowing/denying access

**Key distinction:** Identity policies grant permissions to principals; resource policies control access at the resource itself.

---

### Support Mapping: "AccessDenied" Troubleshooting Flow

**Step 1: Identify the context**
- **Who:** Which principal (user/role/service)?
- **What:** Exact action denied (example: `s3:GetObject`)
- **Where:** Resource ARN (example: `arn:aws:s3:::my-bucket/file.txt`)

**Source:** Error message or CloudTrail event

---

**Step 2: Check for explicit deny**
- Review all identity-based policies (user + group policies)
- Search for `Deny` statements matching the action
- Remember: Explicit deny overrides all allows

---

**Step 3: Check identity-level restrictions**

**Permission boundary (if set):**
- Limits maximum permissions an identity can have
- Check: IAM user/role → Permissions boundary tab
- My user status: No permission boundary set

**SCPs (if account in AWS Organizations):**
- Can restrict entire account or OU
- Check: AWS Organizations console → Policies → SCPs
- Applied to account/OU, not individual users

---

**Step 4: Check resource policy layer**

**Common resource policy locations:**
- **S3:** Bucket policies, S3 Block Public Access settings
- **KMS:** Key policies (KMS keys have explicit permission requirements)
- **Lambda:** Function resource policies
- **SNS/SQS:** Topic/queue policies

**Important:** Resource policies can deny access even when identity permissions look correct.

---

### AccessDenied Decision Tree

**Troubleshooting checklist:**

1. **Confirm principal identity:**
   - User, role, or service making the request
   - Check CloudTrail for `userIdentity` details

2. **Verify action + resource:**
   - Exact API action denied (example: `s3:PutObject`)
   - Full resource ARN

3. **Search for explicit deny:**
   - Identity policies (user + groups)
   - Permission boundaries
   - SCPs (if Organizations)

4. **Check for missing allow:**
   - Identity policy must explicitly allow the action
   - Default deny applies if no explicit allow exists

5. **Inspect resource policy:**
   - Bucket policy, key policy, etc.
   - Resource-level blocks (S3 Block Public Access, etc.)

---

### Key Takeaways

**Permission sources:**
- **Identity policies:** Attached to users/groups/roles
- **Permission boundaries:** Maximum permission limits
- **SCPs:** Organization/account-level restrictions
- **Resource policies:** Resource-level access control

**Evaluation logic:**
- Explicit deny → Always wins
- Explicit allow + no deny → Access granted
- No explicit allow → Implicit deny (access denied)

**Troubleshooting priority:**
1. Identify principal + action + resource
2. Search for explicit deny (any layer)
3. Verify explicit allow exists (identity policy)
4. Check resource policy layer
5. Validate no higher-level restrictions (boundaries, SCPs)

---

## Feb 17, 2026

### VPC Routing + IGW vs NAT

**Route table validation:**
- Confirmed subnet is **public** because route table contains `0.0.0.0/0 → igw-0d2b4afba531a9b3b`
- IGW is attached to the VPC
- Instance has public IPv4 (`13.52.179.133`), no Elastic IP
- Instance can use IGW for inbound/outbound as long as SG and NACL allow it

**Two key routes in route table:**

| Destination | Target | Purpose |
|-------------|--------|---------|
| `172.31.0.0/16` | local | VPC-internal traffic |
| `0.0.0.0/0` | IGW | Internet-bound traffic |

**Longest prefix match rule:**
- More specific route always wins
- VPC-local traffic matches `172.31.0.0/16` instead of the default route
- Internet traffic falls through to `0.0.0.0/0 → IGW`

---

### NAT Gateway Status

- No NAT gateways present in this account
- Private subnet instance would have **no internet access** without:
  1. NAT Gateway existing in a public subnet
  2. Private route table pointing `0.0.0.0/0 → NAT`

**Key distinction:**

| Scenario | Internet Access Path |
|----------|---------------------|
| Public subnet instance | `0.0.0.0/0 → IGW` (direct) |
| Private subnet instance | `0.0.0.0/0 → NAT → IGW` (via NAT) |
| Private subnet, no NAT | No internet access |

---

### Ticket A: "Can't apt/yum Update" (Outbound troubleshooting)

**Initial suspects for outbound failure:**
- SG egress rules blocking traffic
- DNS resolution failing
- Private subnet with missing NAT gateway

**Diagnostics run:**
```bash
dig example.com +short        # DNS resolution check
ip route                      # Routing table check
curl -I https://example.com   # Outbound HTTPS check
```

**Results:**
- `dig` returned IPs → DNS resolution working
- `ip route` showed default route via VPC router (`172.31.0.1`) → routing correct
- `curl` returned TLS error: `curl: (60) unable to get local issuer certificate`

**Further investigation:**
```bash
curl -k https://example.com              # TLS verification disabled
openssl s_client -connect example.com:443  # TLS handshake details
```

**`openssl` result:** `Verify return code: 20 (unable to get local issuer certificate)`

**Root cause:** Not a VPC/routing/SG issue. OS-level TLS trust store problem — instance does not trust the certificate issuer chain.

**Takeaway:** `curl -k` succeeding but `curl` (without `-k`) failing confirms TLS verification is the issue, not network connectivity.

---

### Ticket B: "Cannot SSH to Instance" (Inbound troubleshooting)

**Step 1: Test network path**
```bash
nc -vz -w 3 13.52.179.133 22
```
**Result:** Succeeded → network path to port 22 is open (SG/NACL/routing all clear)

**Step 2: Test SSH connection**
```bash
ssh -v <user>@13.52.179.133
```
**Result:** `Connection established` + `Authenticated using "publickey"` → inbound SSH working end-to-end

---

### Error Pattern Reference

| Symptom | Likely Cause | Where to Investigate |
|---------|-------------|---------------------|
| **Timeout** | SG/NACL/route/missing public IP | VPC network path components |
| **Connection refused** | sshd not running, OS firewall | Instance service status, port binding |
| **nc succeeds, SSH fails** | Wrong username or key | Authentication, username, key permissions |
| **curl TLS error** | Trust store / cert chain issue | OS cert bundle, not networking |

---

### Key Takeaways

**Public vs private subnet:**
- Public = route table has `0.0.0.0/0 → IGW`
- Private = no direct IGW route (needs NAT for outbound internet)

**Longest prefix match:**
- Always check which route is more specific before assuming traffic takes the default route

**Outbound troubleshooting priority:**
1. Check SG egress rules
2. Verify DNS resolution (`dig`)
3. Verify routing (`ip route`)
4. If private subnet, confirm NAT gateway exists and route table points to it
5. If routing is correct but request fails, investigate at the application/OS level (TLS, service config)

**Inbound troubleshooting priority:**
1. `nc` test → classifies issue as network vs instance/auth
2. Timeout → check SG, NACL, route table, public IP
3. Connection refused → check sshd status, port binding
4. Connection succeeds, SSH fails → check username, key, permissions

---

## Feb 18, 2026

### IAM Basics Review + AccessDenied Patterns

**Principal verification:**
- Confirmed working as IAM user `stephen-admin` (not an assumed role)
- Permission sources:
  - AdministratorAccess attached directly to user
  - AdministratorAccess via Admins group membership
- Permission boundary: None set
- MFA: Enabled
- Access keys: None (no long-lived credentials on this user)

---

### AccessDenied Debugging Framework

**Step 1: Identify the context (the "3 Ws"):**
1. **Who:** Principal identity (user, role, service)
2. **What:** Exact action denied (e.g., `s3:GetObject`)
3. **Where:** Full resource ARN

**Step 2: Check policy layers systematically:**
1. Identity policies (user/group/role policies)
2. Resource policies (bucket policy, key policy, etc.)
3. Permission boundaries (if set)
4. SCPs (if Organizations)
5. Session policies (if temporary credentials)

**Step 3: Apply "explicit deny wins" rule:**
- Any `Deny` in any policy layer overrides all `Allow` statements
- Check all policy layers for explicit denies before assuming missing allow

**Step 4: Validate accuracy:**
- Correct region for the resource
- Correct ARN format
- Correct API action name

---

### User Credentials vs EC2 Instance Role Credentials

**User credentials:**
- Who: My identity when working in console/CLI on my laptop
- Source: Long-term credentials (password, MFA, access keys if created)
- Permissions: Based on my user's identity policies + group policies

**EC2 instance role credentials:**
- Who: Temporary credentials available to applications running on EC2
- Source: Instance profile → IAM role → STS temporary credentials via IMDS
- Permissions: Based on the role's identity policies
- Delivery: Available at `http://169.254.169.254/latest/meta-data/iam/security-credentials/<role-name>`

**Key distinction:** Instance needs an attached IAM role (via instance profile) to have credentials. Without a role, the instance has no AWS credentials.

---

### EC2 Instance Role Verification

**Checked my bastion instance:**
- No IAM role attached
- Result: Instance has no default credentials to make AWS API calls

**How to verify instance role:**
- EC2 Console → Instance → Security tab → IAM role
- OR from the instance: `curl http://169.254.169.254/latest/meta-data/iam/security-credentials/`

---

### Support Mapping: "AccessDenied When Calling AWS API"

**Troubleshooting checklist:**

**1) Confirm principal identity:**
```bash
aws sts get-caller-identity
```
- Returns: User ARN or assumed-role ARN
- Confirms who the API client thinks it is

**2) Verify action + resource:**
- Exact API action from error message
- Full resource ARN (check for typos, wrong region, wrong account ID)

**3) Search for explicit deny:**
- Identity policies
- Resource policies
- Permission boundaries
- SCPs
- Session policies (if using temporary credentials)

**4) Check for missing allow:**
- Identity policy must explicitly allow the action on the resource
- Default deny applies if no explicit allow

**5) Validate region and ARN accuracy:**
- Some resources are region-specific
- ARN must match exactly (account ID, region, resource name)

---

### Support Mapping: "EC2 Instance Cannot Access S3"

**Troubleshooting flow:**

**Step 1: Check if instance profile role is attached**
- EC2 Console → Instance → Security tab → IAM role
- If **no role attached:**
  - Instance has no credentials to call AWS APIs
  - **Fix:** Attach an IAM role with necessary S3 permissions

**Step 2: If role is attached, check role policy**
- IAM Console → Roles → Select role → Permissions
- Verify role policy allows required S3 actions:
  - `s3:GetObject` for reading objects
  - `s3:PutObject` for writing objects
  - `s3:ListBucket` for listing bucket contents

**Step 3: Check bucket policy**
- S3 Console → Bucket → Permissions → Bucket policy
- Look for explicit `Deny` statements
- Verify no conflicting conditions (source IP, VPC endpoint, etc.)

**Step 4: Check for encryption (if applicable)**
- If bucket uses SSE-KMS encryption:
  - Role must have `kms:Decrypt` permission for the KMS key
  - KMS key policy must allow the role to use the key

**Step 5: Verify from the instance**
```bash
# Check if role credentials are available
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/

# Test S3 access
aws s3 ls s3://bucket-name/
aws s3 cp s3://bucket-name/file.txt /tmp/
```

---

### Common Root Cause Patterns

| Symptom | Common Causes | Where to Check |
|---------|--------------|----------------|
| AccessDenied with no role attached | Instance has no credentials | EC2 → Security tab → IAM role |
| AccessDenied with role attached | Role policy missing action | IAM → Role → Permissions |
| AccessDenied despite broad role policy | Bucket policy explicit deny | S3 → Bucket → Permissions → Bucket policy |
| AccessDenied on encrypted objects | Missing KMS permissions | Role policy + KMS key policy |
| AccessDenied intermittently | Session policy limiting temp creds | Check how temporary credentials are obtained |

---

### Key Takeaways

**Separation of concerns:**
- **Identity policy issues:** Missing allow, permission boundary limiting access
- **Resource policy issues:** Bucket policy deny, KMS key policy deny
- Always identify which layer is causing the deny before attempting fixes

**Debugging priority:**
1. Confirm principal (who am I?)
2. Identify action + resource (what am I trying to do, where?)
3. Search for explicit denies (any layer)
4. Verify explicit allow exists (identity policy or resource policy)
5. Check for higher-level restrictions (boundaries, SCPs, session policies)

**EC2 instance credentials:**
- Instance must have an attached IAM role to access AWS services
- Role credentials are temporary and rotated automatically via IMDS
- Role policy must allow the needed actions
- Resource policies (bucket, key) can still block access even with correct role policy

---

## Feb 19, 2026

### S3 Access Control + Block Public Access + 403/404 Patterns

**Lab setup:**
- Created bucket: `stephen-s3-access-lab-0219` (us-west-1)
- Initial state:
  - Block Public Access (BPA): **ON** at bucket level
  - Bucket policy: None
  - Object Ownership: **Bucket owner enforced** (ACLs disabled)
- Uploaded test object: `public-test.txt`
- Captured object URL for anonymous testing

---

### Block Public Access (BPA) Behavior

**Attempted to add public-read bucket policy:**
- With BPA **ON:** S3 blocked policy from saving (cannot allow public access)
- With BPA **OFF:** Policy saved successfully

**Policy test after BPA disabled:**
```bash
curl -I <object-url>
```
**Result:** HTTP 403 Forbidden (despite bucket policy allowing public read)

**Missing object test:**
```bash
curl -I <object-url-with-wrong-key>
```
**Result:** HTTP 403 Forbidden (same as existing object)

**Key insight:** S3 returns 403 instead of 404 when the caller is not allowed to know if the object exists.

---

### Multi-Layer BPA Architecture

**Two levels of Block Public Access:**

1. **Account-level BPA:**
   - S3 Console → Block Public Access settings for this account
   - Applied to all buckets in the account
   - Can block public access even when bucket-level BPA is off

2. **Bucket-level BPA:**
   - S3 Console → Bucket → Permissions → Block public access
   - Applied only to this specific bucket
   - Can be overridden by account-level BPA

**Important:** Both layers must allow public access for a bucket policy to grant public access.

---

### Object Ownership and ACL Mode

**Object Ownership setting: Bucket owner enforced**
- ACLs are **disabled**
- Access control is via **policies only** (IAM policies + bucket policies)
- No per-object ACL configuration available

**Alternative setting: Object writer (not used in this lab):**
- ACLs enabled
- Object uploader can set per-object ACLs
- Access controlled by combination of policies + ACLs

**Key takeaway:** With "Bucket owner enforced," ACLs cannot be used. All access must be granted via IAM policies or bucket policies.

---

### Identity Policy vs Bucket Policy (Reinforced)

**Identity policy:**
- Attached to: IAM users, groups, or roles
- Scope: Grants permissions to the principal
- Example: My user's AdministratorAccess allows `s3:*` on all buckets
- Cannot grant public access (no way to specify `Principal: "*"`)

**Bucket policy:**
- Attached to: S3 bucket
- Scope: Controls who can access this bucket
- Can explicitly allow or deny specific principals
- Can grant public access using `Principal: "*"`
- Example:
```json
{
  "Effect": "Allow",
  "Principal": "*",
  "Action": "s3:GetObject",
  "Resource": "arn:aws:s3:::bucket-name/*"
}
```

---

### Support Mapping: "Public Object Returns AccessDenied (403)"

**Troubleshooting checklist (in order):**

**1) Check account-level Block Public Access:**
- S3 Console → Block Public Access settings for this account
- Verify all four settings are **OFF** if public access is required
- Account-level BPA overrides bucket-level settings

**2) Check bucket-level Block Public Access:**
- S3 Console → Bucket → Permissions → Block public access
- Verify settings are **OFF** if public access is required

**3) Review bucket policy:**
- S3 Console → Bucket → Permissions → Bucket policy
- Verify policy explicitly allows public access:
  - `"Principal": "*"`
  - `"Action": "s3:GetObject"`
  - `"Resource": "arn:aws:s3:::bucket-name/*"`
- Check for explicit `Deny` statements

**4) Check Object Ownership mode:**
- S3 Console → Bucket → Permissions → Object Ownership
- If "Bucket owner enforced": ACLs disabled, use policies only
- If "Object writer": Check both policies and per-object ACLs

**5) Verify action + resource ARN:**
- Confirm exact action needed: `s3:GetObject` for reading objects
- Verify resource ARN matches object path exactly
- Format: `arn:aws:s3:::bucket-name/object-key`

---

### 403 vs 404 Response Codes

**HTTP 403 Forbidden:**
- Meaning: Request blocked by permissions, BPA, or policy constraints
- S3 behavior: Also used when object doesn't exist but caller is not allowed to know (security through obscurity)

**HTTP 404 Not Found:**
- Meaning: Wrong bucket name, wrong object key, wrong region, or object genuinely missing
- S3 behavior: Only returned when the caller has permission to know if the object exists

**Decision rule:**
- If anonymous request → expect 403 for both "access denied" and "object missing" (when BPA is on)
- If authenticated request with list permissions → expect 404 for missing objects

---

### S3 Public Access Decision Tree

**When debugging public access issues:**

**1) Identify the context (the "3 Ws"):**
- **Who:** Principal making the request (anonymous `*` or specific principal)
- **What:** Action needed (`s3:GetObject`, `s3:ListBucket`, etc.)
- **Where:** Exact resource ARN (bucket or object key)

**2) Check BPA layers (most common blocker):**
- Account-level BPA settings
- Bucket-level BPA settings
- Both must allow public access

**3) Review policy layers:**
- Bucket policy must explicitly allow `Principal: "*"`
- Identity policy (if authenticated) must allow the action
- Check for explicit denies

**4) Validate Object Ownership mode:**
- Bucket owner enforced → policies only
- Object writer → check policies + ACLs

---

### Common Root Cause Patterns

| Symptom | Common Causes | Where to Check |
|---------|--------------|----------------|
| Policy won't save | Bucket-level BPA is ON | S3 → Bucket → Block public access |
| Policy saved but 403 | Account-level BPA is ON | S3 → Account Block Public Access settings |
| Authenticated works, public fails | Missing `Principal: "*"` in bucket policy | Bucket policy statement |
| 403 on missing object | Caller not allowed to know existence | Expected behavior with BPA/no permissions |
| Policy looks correct but 403 | Explicit deny in bucket policy or identity policy | Search for `"Effect": "Deny"` |

---

### Key Takeaways

**S3 public access is multi-layered:**
1. Account-level BPA (highest priority, applies to all buckets)
2. Bucket-level BPA (bucket-specific)
3. Bucket policy (explicit allow/deny)
4. Object Ownership mode (ACLs vs policies)

**BPA is often the hidden blocker:**
- Even when bucket policy allows public access
- Always check both account-level and bucket-level BPA
- Account-level BPA overrides bucket-level settings

**403 vs 404 interpretation:**
- 403 can mean "access denied" OR "object doesn't exist but you're not allowed to know"
- 404 only appears when the caller has permission to know about object existence
- Don't assume 403 always means the object exists

**Debugging priority:**
1. Identify principal, action, and exact resource ARN
2. Check account-level BPA
3. Check bucket-level BPA
4. Review bucket policy for explicit allow/deny
5. Verify Object Ownership mode (ACLs vs policies)
6. Confirm exact action matches what's needed (`GetObject` vs `ListBucket`)

---

## Feb 20, 2026

### CloudWatch Monitoring Basics

**Lab setup:**
- Launched EC2 instance: `cw-lab-bastion-0220` (`i-0286f7b6fcfda6866`)
- Region: us-east-1c
- Status: Running, status checks passing

---

### Default EC2 Metrics in CloudWatch

**EC2 Console → Instance → Monitoring tab:**

**Default metrics available (no CloudWatch agent required):**
- **CPUUtilization:** Percentage of allocated CPU in use
- **NetworkIn/NetworkOut:** Network traffic volume
- **DiskReadOps/DiskWriteOps:** Disk I/O operations
- **StatusCheckFailed:** Combined system and instance status check failures
- **CPUCreditUsage/CPUCreditBalance:** T-series instance credit metrics (burstable performance)

**Key characteristics:**
- Metrics are **region-scoped** (check console region if metrics missing)
- May show "No data" initially (wait a few minutes for first datapoints)
- Default monitoring: 5-minute intervals (detailed monitoring: 1-minute intervals)

---

### CloudWatch Core Concepts

**Three main components:**

| Component | Description | Example |
|-----------|-------------|---------|
| **Metrics** | Numeric time-series data | CPUUtilization: 45.2% at 10:15 AM |
| **Logs** | Text-based event records | Application logs, system logs, VPC flow logs |
| **Alarms** | Evaluate metric thresholds and trigger actions | Alert when CPU > 70% for 5 minutes |

**Relationship:**
- Metrics provide data points over time
- Alarms monitor metrics and change state based on thresholds
- Logs capture detailed event information (requires CloudWatch agent for custom logs)

---

### CloudWatch Alarms Created

**Alarm 1: CPU High**
- Name: `cw-lab-cpu-high`
- Metric: `CPUUtilization`
- Threshold: `> 70%`
- Evaluation: Single datapoint
- Action: None (monitoring only)

**Alarm 2: Status Check Failed**
- Name: `cw-lab-statuscheck-failed`
- Metric: `StatusCheckFailed`
- Threshold: `>= 1`
- Purpose: Detect instance or system impairment

---

### Alarm States and Missing Data Treatment

**Three alarm states:**
- **OK:** Metric within threshold
- **ALARM:** Metric breached threshold
- **INSUFFICIENT_DATA:** Not enough data to evaluate

**Missing data treatment affects state:**
- Different alarms can show different states (OK vs INSUFFICIENT_DATA) even when the instance appears healthy
- Depends on how the alarm is configured to handle missing datapoints
- Options: Treat as breaching, not breaching, ignore, or maintain last state

**Example:**
- CPU alarm may be OK (data available, below threshold)
- Status check alarm may be INSUFFICIENT_DATA (waiting for enough datapoints)

---

### Support Mapping: "High CPU Usage"

**Troubleshooting flow:**

**Step 1: Verify high CPU in CloudWatch**
- EC2 Console → Instance → Monitoring tab → CPUUtilization graph
- OR CloudWatch Console → Metrics → EC2 → Per-Instance Metrics
- Confirm metric shows sustained high values

**Step 2: Connect to instance and identify process**
```bash
# Real-time process monitoring
top

# Sort by CPU usage (press Shift+P in top)
# Identify top CPU-consuming processes

# Detailed process list
ps aux --sort=-%cpu | head -20
```

**Step 3: Determine root cause**
- Legitimate workload spike (expected)
- Runaway process (memory leak, infinite loop)
- Under-provisioned instance (too small for workload)
- Attack or malicious activity

**Step 4: Resolution options**

| Scenario | Action |
|----------|--------|
| Runaway process | Kill/restart the process, investigate code |
| Expected spike | No action needed, confirm normal behavior |
| Consistent high usage | Resize to larger instance type |
| Predictable patterns | Implement Auto Scaling |
| Malicious activity | Investigate security, isolate instance |

---

### Support Mapping: "Status Check Failed"

**Two types of status checks:**

**System Status Check (AWS infrastructure):**
- Checks underlying hardware/network
- Failures indicate AWS infrastructure issues
- **Resolution:** Stop and start instance (moves to new host)

**Instance Status Check (instance-level):**
- Checks OS and networking configuration
- Failures indicate guest OS or configuration issues
- **Resolution:** Reboot instance, or investigate and fix root cause

**Troubleshooting flow:**

**Step 1: Identify which check failed**
- EC2 Console → Instance → Status checks tab
- Separate indicators for System and Instance checks

**Step 2: Choose appropriate action**

| Check Failed | Common Causes | Resolution |
|--------------|---------------|------------|
| **System** | Hardware failure, network issue, AWS infrastructure problem | Stop → Start instance (migrates to new host) |
| **Instance** | Failed system status check, kernel panic, misconfigured network, full disk | Reboot instance, connect via console, investigate logs |
| **Both** | Start with system check resolution | Stop → Start first, then diagnose instance issues |

**Step 3: Verify resolution**
- Wait for status checks to complete (may take a few minutes)
- Confirm both checks show "2/2 checks passed"
- Monitor CloudWatch alarm returns to OK state

**Step 4: Consider automation (for production)**
- CloudWatch alarm → SNS notification
- Auto-recovery action (automatically recovers instance on system check failure)
- Lambda function for custom remediation

---

### CloudWatch Metrics Best Practices

**Region awareness:**
- CloudWatch is region-scoped
- If metrics are missing, verify console is set to correct region
- Metrics from us-east-1 won't appear when viewing us-west-2

**Data freshness:**
- New instances may show "No data" for first few minutes
- Allow 5-10 minutes for initial datapoints
- Refresh the console to see updated data

**Alarm design:**
- Single datapoint alarms are sensitive (may trigger on brief spikes)
- Multi-datapoint evaluation reduces false positives (e.g., 3 out of 5 datapoints)
- Choose threshold and evaluation period based on workload patterns

---

### Key Takeaways

**CloudWatch components:**
- **Metrics:** Numeric time-series data (CPU, network, disk, status checks)
- **Logs:** Text event records (requires agent for custom logs)
- **Alarms:** Monitor metrics and trigger state changes/actions

**Debugging high CPU:**
1. Verify in CloudWatch metrics
2. SSH to instance and use `top`/`ps` to identify process
3. Determine if legitimate, runaway, or under-provisioned
4. Take appropriate action (kill process, resize, autoscale)

**Status check failures:**
- System check = AWS infrastructure issue → Stop/Start instance
- Instance check = Guest OS issue → Reboot or investigate
- Both failing → Start with system check resolution

**Cost management:**
- Stop or terminate instances when not actively using them
- Running instances accrue hourly charges even when idle
- T-series instances: monitor CPU credits to avoid throttling

---
