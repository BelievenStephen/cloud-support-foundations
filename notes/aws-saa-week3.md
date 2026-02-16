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
