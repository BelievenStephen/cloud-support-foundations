# AWS SAA Study Notes

## Feb 23, 2026 (Week 4)

### CloudTrail Investigation: SSH Access Broke After Change

**Incident scenario:**
- SSH access to EC2 instance suddenly stopped working
- Symptoms indicate recent configuration change
- Need to identify what changed and when

---

### Symptom Identification

**Initial test:**
```bash
nc -vz -w 3 <EC2_PUBLIC_IP> 22
```
**Result:** Connection timeout

**Symptom classification:**
- Timeout (not connection refused or auth failure)
- Indicates network path blocked
- Most likely: Security Group or NACL change

---

### Investigation: CloudTrail Event History

**Step 1: Open CloudTrail Event history**
- Region: us-west-1 (match resource region)
- Time range: Last few hours

**Step 2: Filter to Security Group changes**
- Event name: `RevokeSecurityGroupIngress`
- Resource: `sg-<SECURITY_GROUP_ID>`

---

### Evidence: The Breaking Change

**Event found:**
- **Event time:** 2026-02-21T15:19:58Z
- **Event name:** `RevokeSecurityGroupIngress`
- **Principal:** `arn:aws:iam::<ACCOUNT_ID>:user/stephen-admin`
- **Username:** `stephen-admin`
- **Source IP:** `<MY_PUBLIC_IP>`
- **Resource:** `sg-<SECURITY_GROUP_ID>`

**What was removed:**
```json
{
  "requestParameters": {
    "groupId": "sg-<SECURITY_GROUP_ID>",
    "ipPermissions": {
      "items": [
        {
          "ipProtocol": "tcp",
          "fromPort": 22,
          "toPort": 22,
          "ipRanges": {
            "items": [
              {
                "cidrIp": "<MY_PUBLIC_IP>/32"
              }
            ]
          }
        }
      ]
    }
  }
}
```

**Root cause identified:**
- SSH inbound rule (TCP 22) was revoked
- Source IP `<MY_PUBLIC_IP>/32` no longer allowed
- Change made by `stephen-admin` from console

---

### Resolution: Re-add Security Group Rule

**Action taken:**
- EC2 Console → Security Groups → Select `sg-<SECURITY_GROUP_ID>`
- Add inbound rule: SSH (TCP 22), Source: My IP (`<MY_PUBLIC_IP>/32`)

**Evidence of fix:**
- **Event time:** 2026-02-21T15:26:55Z
- **Event name:** `AuthorizeSecurityGroupIngress`
- **Principal:** `stephen-admin`
- **Action:** Added TCP 22 for `<MY_PUBLIC_IP>/32` on `sg-<SECURITY_GROUP_ID>`

---

### Verification

**Test 1: Network connectivity**
```bash
nc -vz -w 3 <EC2_PUBLIC_IP> 22
```
**Result:** Connection succeeded

**Test 2: SSH connection**
```bash
ssh -i <key.pem> ec2-user@<EC2_PUBLIC_IP>
```
**Result:** Successfully connected

**Outcome:** SSH access restored.

---

### Troubleshooting Workflow Summary

**1) Classify symptom:**
- Timeout → Network path issue (SG, NACL, routing)
- Connection refused → Service/OS issue
- Auth failure → Credentials issue

**2) Use CloudTrail to find recent changes:**
- Filter by relevant event names (`RevokeSecurityGroupIngress`, `AuthorizeSecurityGroupIngress`)
- Filter by resource (security group ID, instance ID, etc.)
- Review events near time issue started

**3) Identify what changed:**
- Review `requestParameters` in event JSON
- Determine exact change (port, protocol, CIDR)
- Confirm change timing aligns with issue

**4) Implement fix:**
- Revert change or apply correct configuration
- Document change in CloudTrail

**5) Verify resolution:**
- Test connectivity with `nc`
- Test actual service (SSH, HTTP, etc.)
- Confirm CloudTrail shows fix event

---

### Key CloudTrail Events for Network Issues

| Issue Type | Event Names to Check | What to Look For |
|-----------|---------------------|------------------|
| **SSH timeout** | `RevokeSecurityGroupIngress`, `ModifyNetworkInterfaceAttribute` | Removed TCP 22 rule, public IP removed |
| **HTTP timeout** | `RevokeSecurityGroupIngress`, `DeleteRoute` | Removed TCP 80/443 rule, IGW route deleted |
| **Instance unreachable** | `StopInstances`, `TerminateInstances`, `ModifySubnetAttribute` | Instance stopped, subnet changed |
| **S3 AccessDenied** | `PutBucketPolicy`, `PutPublicAccessBlock` | Bucket policy changed, BPA enabled |
| **IAM permission error** | `DetachUserPolicy`, `PutUserPolicy` | Policy detached, deny added |

---

### Investigation Tips

**Speed up CloudTrail investigation:**

**Use specific filters:**
- Event name (exact API action)
- Resource name (SG ID, bucket name, etc.)
- Username (if known)
- Time range (narrow to incident window)

**Read events efficiently:**
1. Check `eventName` first (what happened)
2. Check `errorCode` (success or failure)
3. Read `userIdentity.userName` (who did it)
4. Review `requestParameters` (what changed)
5. Check `eventTime` (when it happened)

**Look for patterns:**
- Multiple changes in short time
- Same principal making many changes
- Changes to multiple related resources

---

### Common Mistakes and Prevention

**Mistake:** Accidentally removed wrong security group rule

**Prevention strategies:**
1. **Use resource tags:** Tag resources with purpose/owner
2. **Implement naming conventions:** Clear SG names (e.g., `prod-web-sg`, `dev-bastion-sg`)
3. **Use change management:** Require approval for production changes
4. **Enable CloudTrail alerts:** SNS/Lambda notifications for critical events
5. **Use infrastructure as code:** Terraform/CloudFormation with version control

**Quick recovery:**
- CloudTrail preserves exact change details
- Can reconstruct previous state from `requestParameters`
- Reference previous `AuthorizeSecurityGroupIngress` events to see what was there before

---

### Key Takeaways

**CloudTrail for incident investigation:**
- Essential tool for "what changed" questions
- 90-day event history built-in and free
- Filter by event name and resource for fast investigation

**Network timeout troubleshooting:**
1. Use `nc` to classify issue (timeout vs refused vs auth)
2. Check CloudTrail for recent SG/NACL/route changes
3. Verify public IP, route table, IGW unchanged
4. Fix configuration and verify with `nc` + actual service

**Event investigation pattern:**
- Filter Event history → Open relevant event → Read requestParameters → Identify exact change → Apply fix → Verify

**Prevention > reaction:**
- Use clear naming and tagging
- Implement change management processes
- Set up CloudTrail alerts for critical changes
- Use IaC to track and version infrastructure changes

---
