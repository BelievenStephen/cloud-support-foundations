# Runbook: S3 Public Object Returns 403 AccessDenied

## Goal
Resolve 403 AccessDenied errors when an S3 object should be publicly readable.

## When to Use This Runbook
- You expect an S3 object to be publicly accessible
- Object URL returns **403 Forbidden** or **AccessDenied**
- Common case: "I added a public-read bucket policy but it's still denied"

## Symptoms
- Anonymous `curl` returns 403:
```bash
  curl -I https://<bucket>.s3.<region>.amazonaws.com/<key>
  # Returns: HTTP/1.1 403 Forbidden
```
- S3 console shows "Bucket and objects not public"
- Error saving bucket policy due to Block Public Access

## Key Concept: Multi-Layer Access Control

S3 public access requires alignment across multiple layers:

1. **Block Public Access (BPA)** - Account and bucket level
2. **Bucket policy** - Must explicitly allow public access
3. **Object Ownership / ACLs** - Depends on ownership mode
4. **KMS encryption** - Can block public reads even if S3 allows
5. **IAM identity policy** - Only relevant for authenticated requests

**Critical insight:** Block Public Access can override bucket policies that would otherwise allow public access.

---

## Quick Verification Test
```bash
# Anonymous access test (what public users see)
curl -I "https://<bucket>.s3.<region>.amazonaws.com/<key>"

# Expected for public access: HTTP/1.1 200 OK
# Actual if blocked: HTTP/1.1 403 Forbidden
```

---

## Troubleshooting Checklist (In Order)

### 1) Check Block Public Access Settings

**Block Public Access operates at two levels - both must allow public access:**

#### Bucket-Level BPA

**Console:**
- S3 → Buckets → Select bucket → Permissions tab
- Section: **Block public access (bucket settings)**
- Check if "Block all public access" is ON

**CLI:**
```bash
aws s3api get-public-access-block --bucket <bucket-name>
```

#### Account-Level BPA

**Console:**
- S3 console → Left sidebar → **Block Public Access settings for this account**
- Verify which checkboxes are enabled

**CLI:**
```bash
# Get your account ID first
aws sts get-caller-identity

# Check account-level BPA
aws s3control get-public-access-block --account-id <ACCOUNT-ID>
```

**What it means:**
- If "Block public bucket policies" is enabled → S3 will block public policies from being saved or taking effect
- Account-level BPA **overrides** bucket-level settings
- Both layers must allow public access for the bucket policy to work

---

### 2) Verify Bucket Policy

**Console:**
- Bucket → Permissions → **Bucket policy**

**Required elements for public read:**
- `"Principal": "*"` (allows anonymous access)
- `"Action": "s3:GetObject"` (read permission)
- `"Resource": "arn:aws:s3:::bucket-name/*"` (must match object path exactly)

**Example bucket policy (single object):**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowPublicReadSingleObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::bucket-name/object-key"
    }
  ]
}
```

**Example bucket policy (all objects):**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowPublicReadAllObjects",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::bucket-name/*"
    }
  ]
}
```

**CLI:**
```bash
# View current bucket policy
aws s3api get-bucket-policy --bucket <bucket-name>

# Check if bucket is considered public
aws s3api get-bucket-policy-status --bucket <bucket-name>
```

**Common mistakes:**
- Resource ARN doesn't match object path
- Missing `/*` for all objects
- Wrong bucket name in ARN
- Explicit `Deny` statement in policy

---

### 3) Check Object Ownership Mode

**Console:**
- Bucket → Permissions → **Object Ownership**

**Ownership modes:**

| Mode | ACL Status | Access Control Method |
|------|-----------|----------------------|
| **Bucket owner enforced** | Disabled | Policies only (IAM + bucket policy) |
| **Object writer** | Enabled | Policies + per-object ACLs |

**CLI:**
```bash
aws s3api get-bucket-ownership-controls --bucket <bucket-name>
```

**If Object Ownership = "Bucket owner enforced":**
- ACLs are disabled
- All access must be via bucket policy or IAM policy
- Per-object ACLs cannot be used

**If ACLs are enabled (Object writer mode):**
```bash
# Check object-level ACL
aws s3api get-object-acl --bucket <bucket-name> --key <object-key>
```

---

### 4) Check for KMS Encryption

**Console:**
- Bucket → Properties → **Default encryption**

**Important:** If bucket uses SSE-KMS encryption, public read access typically won't work because anonymous users cannot access KMS keys.

**CLI:**
```bash
aws s3api get-bucket-encryption --bucket <bucket-name>
```

**KMS and public access:**
- SSE-S3 (AES256) → Compatible with public access
- SSE-KMS → Generally incompatible with anonymous public access
- No encryption → Compatible with public access

---

### 5) Check IAM Identity Policy (Authenticated Requests Only)

**Only relevant if the request is authenticated (CLI, SDK, signed request)**

**Verify caller identity:**
```bash
aws sts get-caller-identity
```

**Check identity policy:**
- IAM Console → Users/Roles → Select identity → Permissions
- Verify policy allows `s3:GetObject` on the resource ARN

**Note:** Anonymous `curl` requests skip this layer entirely.

---

## Resolution Steps

### Option A: Enable Public Access (Lab/Testing Only)

**Use only for temporary testing. Not recommended for production.**

**Step 1: Disable Block Public Access**
- S3 → Bucket → Permissions → **Block public access (bucket settings)**
- Edit → Uncheck "Block all public access" → Save changes
- Confirm the change

**Step 2: Add Bucket Policy**
- Bucket → Permissions → **Bucket policy** → Edit
- Paste policy allowing public read (see examples above)
- Save changes

**Step 3: Verify**
```bash
curl -I "https://<bucket>.s3.<region>.amazonaws.com/<key>"
# Should return: HTTP/1.1 200 OK
```

---

### Option B: Secure Alternatives (Recommended for Production)

**Instead of public access, use:**

**CloudFront + Origin Access Control (OAC):**
- CloudFront distribution with OAC
- Bucket stays private
- CloudFront serves content securely

**Presigned URLs:**
- Generate time-limited signed URLs
- Bucket stays private
- Users get temporary access

**Authenticated Access:**
- Users authenticate via IAM
- Use roles and identity policies
- No public access needed

---

## Verification

**Anonymous access test:**
```bash
curl -I "https://<bucket>.s3.<region>.amazonaws.com/<key>"
```

**Expected results:**
- Public access enabled: `HTTP/1.1 200 OK`
- Public access blocked: `HTTP/1.1 403 Forbidden`

**Authenticated access test (if applicable):**
```bash
aws s3 cp s3://<bucket>/<key> /tmp/test-download
```

---

## Cleanup After Lab Testing

**IMPORTANT: Do not skip cleanup steps to avoid unintended public access and costs.**

**Step 1: Remove Bucket Policy**
- Bucket → Permissions → **Bucket policy** → Delete → Save changes

**Step 2: Re-enable Block Public Access**
- Bucket → Permissions → **Block public access (bucket settings)**
- Edit → Check "Block all public access" → Save changes

**Step 3: Restore Account-Level BPA (if changed)**
- S3 console → **Block Public Access settings for this account**
- Restore to previous state (typically all ON)

**Step 4: Delete Test Objects**
- Bucket → Objects → Select test objects → Delete

**Step 5: Delete Bucket (Optional)**
```bash
# Empty bucket first
aws s3 rm s3://<bucket> --recursive

# Delete bucket
aws s3 rb s3://<bucket>
```

---

## Common Root Cause Patterns

| Symptom | Root Cause | Fix |
|---------|-----------|-----|
| Console won't save policy | Bucket-level BPA is ON | Disable bucket BPA first |
| Policy saved but still 403 | Account-level BPA is ON | Check account BPA settings |
| Some objects work, others don't | Resource ARN doesn't match | Fix bucket policy Resource ARN |
| Works in console, fails via curl | Using authenticated access in console | Verify bucket policy allows `Principal: "*"` |
| KMS encrypted objects return 403 | Anonymous users can't decrypt KMS | Use SSE-S3 or keep bucket private |

---

## Notes

**403 vs 404 Behavior:**
- S3 may return **403** to hide object existence when access is blocked
- Don't assume 403 always means the object exists
- 404 only returned when caller has permission to know about object existence

**Security Best Practices:**
- Avoid public S3 buckets when possible
- Use CloudFront + OAC for public content distribution
- Use presigned URLs for temporary access
- Enable S3 access logging for audit trail
- Monitor for unintended public access with AWS Config or Trusted Advisor

**If Console Blocks Policy Save:**
- This usually indicates Block Public Access is preventing the save
- Check bucket-level BPA first, then account-level BPA
- Both must allow public policies
