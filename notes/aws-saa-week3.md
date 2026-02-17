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
