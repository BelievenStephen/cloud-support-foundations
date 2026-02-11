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
