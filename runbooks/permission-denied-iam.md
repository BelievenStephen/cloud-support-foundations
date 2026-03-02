# Runbook: IAM Permission Denied (AccessDenied)

## Goal
Resolve an AWS “AccessDenied” error by identifying the denied action and the policy layer causing it.

## Symptoms
- Console error: **AccessDenied**
- API/CLI error: `AccessDenied`, `AccessDeniedException`, `UnauthorizedOperation`
- Action works for one identity but not another

## Fast triage (what to capture first)
1. Exact error message (copy/paste)
2. The **Action** (ex: `iam:CreateUser`, `s3:GetObject`)
3. The **Resource** (ARN if shown)
4. The **Principal** (who is calling: user/role, account ID)
5. When it happened (timestamp)

## Support flow (in order)

## AWS Permission Triage (Console + CLI)

**Use this workflow when encountering:**
- `AccessDenied`, `AccessDeniedException`, `UnauthorizedOperation`
- "User is not authorized to perform: [action] on resource: [ARN]"
- Actions that work for one identity but fail for another

---

### Triage Order (Follow This Sequence)

---

### 1) Identify the Principal (Who is Calling)

**From AWS Console:**
- Top-right corner shows signed-in user or role
- Note account ID and identity name

**From AWS CLI/CloudShell:**
```bash
aws sts get-caller-identity
```

**Capture:**
- `Arn` - Full ARN of caller
- `Account` - AWS account ID
- `UserId` - Unique identifier

---

### 2) Extract Action and Resource (What Failed)

**From error message, identify:**
- **Action:** API operation (e.g., `s3:ListBucket`, `iam:CreateUser`)
- **Resource:** Target ARN (if shown)

**If resource ARN not in error:**
- Infer from context (bucket name, role name, instance ID)
- Construct ARN manually if needed

**Example error:**
```
User: arn:aws:iam::123456789012:user/alice is not authorized to perform: s3:GetObject on resource: arn:aws:s3:::my-bucket/*
```

---

### 3) Check Identity-Based Policies (Where Allows Come From)

**IAM Console → Users (or Roles) → Select identity → Permissions tab**

**Review all policy sources:**
- Managed policies (AWS-managed and customer-managed)
- Inline policies
- Group policies (for IAM users)

**AWS evaluation rule:** Implicit deny unless an explicit allow matches action + resource

**Common issue:** Action is allowed but resource ARN doesn't match

---

### 4) Check for Explicit Deny (Overrides All Allows)

**Search across all policy types for:**
```json
{
  "Effect": "Deny",
  "Action": "...",
  "Resource": "..."
}
```

**Where to check:**
- Identity policies (managed and inline)
- Resource policies (bucket policy, key policy)
- Permissions boundaries
- Service Control Policies (SCPs)

**Critical rule:** Any matching explicit deny wins, even over AdministratorAccess

---

### 5) Verify Resource Policy (Service-Specific)

**Common resource policies:**

| Service | Resource Policy Location | What to Check |
|---------|------------------------|---------------|
| **S3** | Bucket → Permissions → Bucket policy | Deny statements, Principal restrictions |
| **KMS** | Key → Key policy | Principal access, key usage permissions |
| **SNS/SQS** | Topic/Queue → Access policy | Subscriber/publisher permissions |
| **Lambda** | Function → Permissions | Invocation permissions |

**For S3, verify correct ARN pattern:**
- `s3:ListBucket` → `arn:aws:s3:::bucket-name` (bucket ARN)
- `s3:GetObject` → `arn:aws:s3:::bucket-name/*` (object ARN)
- `s3:PutObject` → `arn:aws:s3:::bucket-name/*` (object ARN)

---

### 6) Check Organizations Guardrails (If Applicable)

**Service Control Policies (SCPs):**
- AWS Organizations → Policies → Service control policies
- Can deny actions account-wide, overriding IAM policies

**Permissions Boundaries:**
- IAM Console → User/Role → Permissions boundary tab
- Defines maximum allowed permissions
- Acts as guardrail even if policies grant more

---

### 7) Review Condition Blocks (Hidden Restrictions)

**Common conditions that cause "looks allowed but still denied":**

| Condition Key | Requirement | Impact |
|--------------|-------------|--------|
| `aws:MultiFactorAuthPresent` | MFA required | Fails without MFA |
| `aws:SourceIp` | IP allowlist | Fails from other IPs |
| `aws:SourceVpce` | VPC endpoint required | Fails from non-VPC sources |
| `aws:RequestedRegion` | Region restriction | Fails in other regions |
| Tag-based conditions | Specific tag values | Fails without matching tags |

**Example condition block:**
```json
{
  "Condition": {
    "IpAddress": {
      "aws:SourceIp": "203.0.113.0/24"
    }
  }
}
```

---

### 8) Use IAM Policy Simulator (When Unsure)

**IAM Console → Policy Simulator**

**Test configuration:**
- Principal: Select user or role
- Action: Exact API action from error
- Resource: Exact ARN from error (or inferred)

**Interpret results:**

| Simulator Result | Meaning | Next Steps |
|-----------------|---------|------------|
| **Allowed** | Should work | Check resource policy, conditions, SCPs |
| **ImplicitDeny** | Missing explicit allow | Add allow statement to policy |
| **ExplicitDeny** | Deny statement matched | Find and address deny statement |

**Simulator shows:**
- Which policy allowed/denied
- Conditions evaluated
- Policy logic applied

---

### 9) CloudTrail Lookup (For Evidence)

**When to use:**
- Need proof of which principal made the call
- Want to see exact API parameters
- Investigating recurring or past denials

**CloudTrail Console → Event history**

**Filter by:**
- Event name (the API action)
- Error code: `AccessDenied`

**Capture from event:**
- `userIdentity.arn` - Caller ARN
- `sourceIPAddress` - Request origin
- `eventTime` - When it occurred
- `requestParameters` - What was requested
- `errorCode` and `errorMessage` - Why it failed

---

### Error Message Interpretation

**403 vs 404 (S3-specific):**

| Error | Common Meaning | Troubleshooting |
|-------|---------------|----------------|
| **403 AccessDenied** | Permission denied or explicit deny | Check policies for missing allow or explicit deny |
| **404 NotFound** | Object doesn't exist OR caller not allowed to know it exists | Verify object exists with admin identity, then debug permissions |

**"Not authorized to perform" vs "Access denied":**
- Both indicate missing or denied permissions
- CLI errors often include exact Action and Resource (use these for triage)

---

### Verification (Always Required)

**After implementing fix:**

1. **Re-run the exact same request:**
   - Same principal (user/role)
   - Same action (API call)
   - Same resource (ARN)

2. **Confirm success:**
   - No error message
   - Expected result returned

3. **Optional: CloudTrail verification:**
   - Check latest event for this action
   - Verify `errorCode` is null (success)
   - Confirm no `AccessDenied` in recent events

**Example verification:**
```bash
# Original failing command
aws s3 ls s3://my-bucket/
# Previously: AccessDenied

# After fix - should succeed
aws s3 ls s3://my-bucket/
# Returns: List of objects (no error)
```

---

### Common Resolution Patterns

| Root Cause | Symptom | Resolution |
|-----------|---------|------------|
| **Missing allow** | ImplicitDeny in simulator | Add allow statement to identity policy |
| **Wrong resource ARN** | Specific objects fail | Correct ARN format (bucket vs object) |
| **Explicit deny in bucket policy** | Works for admin, fails for user | Remove or narrow deny in bucket policy |
| **SCP blocking** | Fails even with AdministratorAccess | Adjust SCP at organization level |
| **Permission boundary** | Allowed action still denied | Expand boundary or adjust identity policy |
| **IP condition** | Works from office, fails from home | Add IP to allowlist or remove condition |
| **MFA required** | Fails without MFA | Authenticate with MFA or remove condition |

---

### 1) Identify the denied action
- From the error text, find the action (often shown as `service:Action`)
- If not shown, reproduce once and capture the console banner or CLI output

### 2) Check identity-based permissions (most common)
IAM → **Users** (or Roles) → select identity → **Permissions**
- Attached policies (managed + inline)
- Group memberships and group policies (for users)

**Rule:** default deny unless an allow matches.

### 3) Check for explicit deny (overrides allow)
- Search policies for `"Effect": "Deny"`
- If **any** explicit deny matches the action/resource, it wins.

### 4) Check permission boundaries (if present)
IAM → User/Role → **Permissions boundary**
- Boundary limits maximum permissions even if policies allow more.

### 5) Check Organizations SCPs (if account is in an org)
AWS Organizations → Accounts/OUs → **Service control policies**
- SCP can block actions even if IAM allows them.

### 6) Check resource-based policies (service specific)
Common examples:
- S3 bucket policy / object ACL
- KMS key policy
- SNS/SQS resource policies

## Verification
- Re-run the same action
- Confirm CloudTrail shows **who**, **action**, and **result = success**

## Notes
- **Explicit deny > allow**
- Many “it should be allowed” cases are actually: boundary/SCP/resource policy

---

## Lab Example: S3 Upload AccessDenied (Feb 16, 2026)

### Scenario
S3 console upload failed with `Access denied` error despite having AdministratorAccess policy.

### Context captured
- **Error:** `Access denied` (S3 console upload)
- **Principal:** IAM user `stephen-admin`
  - ARN: `arn:aws:iam::886219357247:user/stephen-admin`
- **Action:** `s3:PutObject`
- **Resource:** `arn:aws:s3:::stephen-accessdenied-lab-2-16-26/*`

### Investigation steps
1. **Verified principal type:** IAM user (not assumed role)
2. **Checked identity-based policies:** AdministratorAccess attached (broad allow)
3. **Searched for explicit deny:** Found deny in bucket policy

### Root cause
Bucket policy contained explicit `Deny` for `s3:PutObject` targeting the principal.

**Key rule proven:** Explicit deny overrides all allows, including AdministratorAccess.

### Resolution
Remove or modify the bucket policy Deny statement:
- Narrow scope to different prefix
- Add condition to exclude specific principals
- Remove deny statement entirely (if appropriate)

### Verification
- Retry S3 upload in console
- Upload succeeds after bucket policy adjustment
- CloudTrail shows `s3:PutObject` with `errorCode: null` (success)

### Takeaway
When identity policies look correct but access is denied, check **resource-based policies** (bucket policies, key policies) for explicit deny statements.
