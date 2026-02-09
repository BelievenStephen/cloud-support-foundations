# AWS SAA Study Notes - Week 2

## Feb 9, 2026

### Account Security Verification

- Confirmed using IAM identity (not root) for console access
- Verified MFA is enabled on my account
- Confirmed AWS Budget and alert emails are configured

### IAM Fundamentals

**Key Concepts:**

- **IAM User:** Long-term identity for people or applications
- **IAM Role:** Temporary identity assumed by AWS services (EC2, Lambda) or users
- **Policy:** JSON document defining allowed/denied actions
- **Default Deny:** Only explicitly allowed actions are permitted
- **Least Privilege:** Grant minimum permissions required for the task

**Hands-on Lab:**

- Created test user `lab-user` (console access only, no access keys)
- Created custom policy `LabIamReadOnly` with read-only IAM permissions
- Attached policy and verified least privilege works (can view IAM resources, cannot create/delete)

**Takeaway:** Reinforced that AWS uses explicit allow model—everything is denied unless a policy grants access.
