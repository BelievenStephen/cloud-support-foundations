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
