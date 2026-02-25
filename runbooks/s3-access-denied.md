# Runbook: S3 AccessDenied (403)

## Goal
Quickly troubleshoot and resolve S3 403 AccessDenied errors without accidentally making data public.

## When to Use This Runbook
- Console or CLI returns **403 Forbidden** or **AccessDenied** for S3 operations
- Cannot list bucket contents
- Cannot read, write, or delete objects
- Access previously worked but now fails

## Symptoms
- Console error: "Access Denied" or "403 Forbidden"
- CLI/SDK error: `An error occurred (403) when calling the [Operation]`
- Error message may mention:
  - "Access Denied"
  - "Not authorized to perform"
  - "Explicit deny"
  - "KMS.AccessDeniedException" (if SSE-KMS)

---

## Troubleshooting Flow (In Order)

### 1) Confirm Caller Identity

**Who is making the request?**

**From AWS Console:**
- Top-right corner shows current identity
- Verify correct account and user/role name

**From AWS CLI:**
```bash
aws sts get-caller-identity
```

**Expected output:**
```json
{
  "UserId": "AIDA...",
  "Account": "123456789012",
  "Arn": "arn:aws:iam::123456789012:user/username"
}
```

**Verify:**
- Correct AWS account
- Expected user or role
- Not accidentally using wrong profile or credentials

---

### 2) Identify Exact Action and Resource

**Determine what operation is failing:**

| Operation | Required Action | Resource ARN |
|-----------|----------------|--------------|
| **List bucket contents** | `s3:ListBucket` | `arn:aws:s3:::bucket-name` (bucket ARN) |
| **Read object** | `s3:GetObject` | `arn:aws:s3:::bucket-name/*` (object ARN) |
| **Write object** | `s3:PutObject` | `arn:aws:s3:::bucket-name/*` |
| **Delete object** | `s3:DeleteObject` | `arn:aws:s3:::bucket-name/*` |

**Critical distinction:**
- **Bucket operations** (list, get location) use bucket ARN without `/*`
- **Object operations** (get, put, delete) use object ARN with `/*`

**Common mistake:** Using wrong ARN format in IAM policy

---

### 3) Check IAM Identity Policy

**IAM Console → Users (or Roles) → Permissions tab**

**What to check:**

**A) Missing Allow (most common):**

**For listing bucket:**
```json
{
  "Effect": "Allow",
  "Action": "s3:ListBucket",
  "Resource": "arn:aws:s3:::bucket-name"
}
```

**For reading objects:**
```json
{
  "Effect": "Allow",
  "Action": "s3:GetObject",
  "Resource": "arn:aws:s3:::bucket-name/*"
}
```

**B) Explicit Deny (always wins):**
```json
{
  "Effect": "Deny",
  "Action": "s3:*",
  "Resource": "*"
}
```

**Note:** Explicit deny overrides all allows, including AdministratorAccess

**C) Restrictive Conditions:**
- IP address restrictions
- MFA required
- VPC endpoint required
- Time-based access
- Tag-based access

**Example condition blocking access:**
```json
{
  "Effect": "Allow",
  "Action": "s3:GetObject",
  "Resource": "arn:aws:s3:::bucket-name/*",
  "Condition": {
    "IpAddress": {
      "aws:SourceIp": "203.0.113.0/24"
    }
  }
}
```

---

### 4) Check Bucket Policy

**S3 Console → Bucket → Permissions → Bucket policy**

**Common causes of AccessDenied:**

**A) Explicit Deny (overrides everything):**
```json
{
  "Effect": "Deny",
  "Principal": "*",
  "Action": "s3:*",
  "Resource": [
    "arn:aws:s3:::bucket-name",
    "arn:aws:s3:::bucket-name/*"
  ]
}
```

**B) Principal restrictions:**
```json
{
  "Effect": "Allow",
  "Principal": {
    "AWS": "arn:aws:iam::123456789012:user/allowed-user"
  },
  "Action": "s3:GetObject",
  "Resource": "arn:aws:s3:::bucket-name/*"
}
```
If you're not the specified principal, access denied.

**C) Condition restrictions:**

**IP address restriction:**
```json
{
  "Condition": {
    "IpAddress": {
      "aws:SourceIp": "203.0.113.0/32"
    }
  }
}
```

**VPC endpoint restriction:**
```json
{
  "Condition": {
    "StringEquals": {
      "aws:SourceVpce": "vpce-1234567"
    }
  }
}
```

**MFA required:**
```json
{
  "Condition": {
    "Bool": {
      "aws:MultiFactorAuthPresent": "true"
    }
  }
}
```

**HTTPS required:**
```json
{
  "Condition": {
    "Bool": {
      "aws:SecureTransport": "false"
    }
  },
  "Effect": "Deny"
}
```

**D) Wrong Resource ARN:**
- Bucket policy must use correct ARN format
- Bucket operations: `arn:aws:s3:::bucket-name`
- Object operations: `arn:aws:s3:::bucket-name/*`

---

### 5) Check Block Public Access (BPA)

**S3 Console → Bucket → Permissions → Block public access (bucket settings)**

**What BPA does:**
- **Prevents public access** (when `Principal: "*"`)
- **Does NOT grant access** to authenticated users
- Usually not the cause of AccessDenied for authenticated requests

**Four BPA settings:**
1. Block public access to buckets and objects granted through new ACLs
2. Block public access to buckets and objects granted through any ACLs
3. Block public access to buckets and objects granted through new public bucket policies
4. Block public access to buckets and objects granted through any public bucket policies

**Recommended for most use cases:** All four ON

**When BPA causes issues:**
- You intentionally need public access (e.g., public website)
- Bucket policy with `Principal: "*"` won't work if BPA is ON

---

### 6) Check Encryption (SSE-KMS Only)

**S3 Console → Bucket → Properties → Default encryption**

**If encryption type is SSE-KMS:**

**Symptoms:**
- S3 permissions look correct
- Still getting AccessDenied
- Error may mention "KMS" or decryption

**Required IAM permissions:**
```json
{
  "Effect": "Allow",
  "Action": [
    "kms:Decrypt",
    "kms:DescribeKey"
  ],
  "Resource": "arn:aws:kms:region:account-id:key/key-id"
}
```

**Required KMS key policy:**
- Key policy must allow the principal
- Check: KMS Console → Keys → Select key → Key policy

**Note:** SSE-S3 (default) does NOT require KMS permissions

---

### 7) Use CloudTrail to Find Recent Changes

**When to check CloudTrail:**
- Access worked previously but now fails
- Investigating "who changed what"
- Need proof of configuration changes

**CloudTrail Console → Event history**

**Filter by event names:**
- `PutBucketPolicy` - Bucket policy changed
- `DeleteBucketPolicy` - Bucket policy removed
- `PutPublicAccessBlock` - BPA settings changed
- `PutBucketAcl` - Bucket ACL changed
- `PutBucketEncryption` - Encryption changed

**What to capture:**
- `eventTime` - When the change occurred
- `userIdentity.userName` - Who made the change
- `sourceIPAddress` - Where the change came from
- `requestParameters` - What exactly changed

---

## Resolution Steps

### Case A: Missing IAM Allow

**Problem:** IAM policy doesn't grant necessary permissions

**Fix:**
1. IAM Console → Users (or Roles) → Permissions
2. Add or modify policy to grant needed action
3. Use correct resource ARN (bucket vs object)

**Example policy fix:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:ListBucket"
      ],
      "Resource": "arn:aws:s3:::bucket-name"
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::bucket-name/*"
    }
  ]
}
```

**Verify:**
```bash
aws s3 ls s3://bucket-name/
aws s3 cp s3://bucket-name/object-key /tmp/test
```

---

### Case B: Explicit Deny in Bucket Policy

**Problem:** Bucket policy contains explicit Deny statement

**Fix:**
1. S3 Console → Bucket → Permissions → Bucket policy
2. Remove or narrow the Deny statement
3. Save changes

**Before (blocking all):**
```json
{
  "Effect": "Deny",
  "Principal": "*",
  "Action": "s3:*",
  "Resource": "*"
}
```

**After (removed or narrowed):**
- Remove statement entirely, OR
- Add condition to exclude specific principals

**Verify:**
```bash
aws s3 ls s3://bucket-name/
```

---

### Case C: Restrictive Conditions in Bucket Policy

**Problem:** Bucket policy conditions don't match your request

**Common fixes:**

**IP address changed:**
```json
{
  "Condition": {
    "IpAddress": {
      "aws:SourceIp": ["203.0.113.0/32", "198.51.100.0/32"]
    }
  }
}
```
Add your current public IP to the list.

**Wrong principal:**
```json
{
  "Principal": {
    "AWS": [
      "arn:aws:iam::account-id:user/old-user",
      "arn:aws:iam::account-id:user/new-user"
    ]
  }
}
```
Add your current user/role ARN.

**VPC endpoint restriction:**
- If accessing from outside VPC, remove VPC endpoint condition
- OR access from within VPC using specified endpoint

---

### Case D: SSE-KMS Decrypt Blocked

**Problem:** Missing KMS permissions for encrypted objects

**Fix:**

**1) Add KMS permissions to IAM policy:**
```json
{
  "Effect": "Allow",
  "Action": [
    "kms:Decrypt",
    "kms:DescribeKey",
    "kms:GenerateDataKey"
  ],
  "Resource": "arn:aws:kms:region:account-id:key/key-id"
}
```

**2) Verify KMS key policy allows principal:**
- KMS Console → Keys → Select key → Key policy
- Ensure your user/role is listed

**Verify:**
```bash
aws s3 cp s3://bucket-name/encrypted-object /tmp/test
```

---

### Case E: Unintended Public Access

**Problem:** Bucket became public when it shouldn't be

**Fix (secure the bucket):**

**1) Enable Block Public Access:**
- S3 Console → Bucket → Permissions → Block public access
- Edit → Check all 4 boxes → Save changes

**2) Remove public bucket policy statements:**
- Remove any statements with `"Principal": "*"`

**3) Enforce Object Ownership:**
- S3 Console → Bucket → Permissions → Object Ownership
- Select "Bucket owner enforced"

**Verify:**
- Bucket details page shows "Bucket and objects not public"
- Test anonymous access fails:
```bash
curl -I https://bucket-name.s3.region.amazonaws.com/object-key
# Should return 403 Forbidden
```

---

## Verification

**After applying fixes:**

**Console test:**
- Navigate to S3 bucket
- Verify can list objects
- Verify can open/download object

**CLI test:**
```bash
# List bucket
aws s3 ls s3://bucket-name/

# Read object
aws s3 cp s3://bucket-name/object-key /tmp/test

# Verify file downloaded
ls -lh /tmp/test
```

**CloudTrail verification:**
- Check CloudTrail Event history
- Verify your change appears (e.g., `PutBucketPolicy`)
- Confirm timestamp and principal

**Security check:**
- S3 Console → Bucket details
- Verify "Public access" shows expected state
- Confirm no unintended public exposure

---

## Common Mistake Patterns

| Symptom | Root Cause | Fix |
|---------|-----------|-----|
| **Can't list bucket** | IAM uses object ARN instead of bucket ARN | Change Resource to `arn:aws:s3:::bucket-name` |
| **Can list but can't read** | IAM missing GetObject on object ARN | Add `s3:GetObject` on `arn:aws:s3:::bucket-name/*` |
| **Works in one region, fails in another** | Bucket in different region, ARN wrong | Verify bucket region, update ARN |
| **Worked yesterday, fails today** | IP address changed, condition blocks | Update bucket policy IP condition |
| **Works for one user, not another** | Bucket policy restricts principal | Add user/role to bucket policy Principal |
| **Can read some objects, not others** | KMS encryption on some objects | Add KMS permissions to IAM policy |

---

## Prevention and Best Practices

**IAM policies:**
- Use least privilege (grant only needed actions)
- Use correct resource ARNs (bucket vs object)
- Document why each permission is needed
- Regularly review and audit policies

**Bucket policies:**
- Minimize use of bucket policies when possible
- Prefer IAM policies over bucket policies
- Document any explicit denies with comments
- Test changes in non-production first

**Security:**
- Enable Block Public Access by default
- Use "Bucket owner enforced" object ownership
- Enable SSE-S3 or SSE-KMS encryption
- Use S3 access logging for audit trail

**Operations:**
- Use CloudTrail to track policy changes
- Set up alerts for critical S3 policy changes
- Maintain documentation of bucket access patterns
- Test access after any policy changes

---

## Cost and Safety Notes

**Cost considerations:**
- S3 charges for storage (GB-month)
- S3 charges for requests (per 1000)
- KMS encryption adds KMS API call charges
- Data transfer charges for downloads

**Safety guidelines:**
- Never disable BPA without documented reason
- Avoid `Principal: "*"` in bucket policies
- Make smallest change that fixes the issue
- Test in lab environment before production
- Document all policy changes

**Lab cleanup:**
- Delete test buckets after lab complete
- Remove temporary bucket policies
- Re-enable BPA if disabled for testing
- Verify no unintended public access remains

---

## Quick Reference

**Diagnostic commands:**
```bash
# Who am I?
aws sts get-caller-identity

# List buckets
aws s3 ls

# List bucket contents
aws s3 ls s3://bucket-name/

# Read object
aws s3 cp s3://bucket-name/object-key /tmp/test

# Get bucket policy
aws s3api get-bucket-policy --bucket bucket-name

# Get bucket public access block
aws s3api get-public-access-block --bucket bucket-name
```

**Key ARN formats:**
- Bucket: `arn:aws:s3:::bucket-name`
- Objects: `arn:aws:s3:::bucket-name/*`
- Specific object: `arn:aws:s3:::bucket-name/path/to/object`
- KMS key: `arn:aws:kms:region:account-id:key/key-id`
