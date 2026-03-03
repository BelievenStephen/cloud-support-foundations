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
