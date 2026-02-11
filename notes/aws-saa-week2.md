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


## Feb 10, 2026

### Policy Evaluation Basics (Proved with Policy Simulator)

**Setup:**
- Attached two policies to `lab-user`:
  - `LabIamReadOnly` (allows IAM list/read actions)
  - `LabIamDenyUserCreate` (explicitly denies `iam:CreateUser`)

**Tested actions in IAM Policy Simulator:**
- `iam:ListUsers` = **Allowed** (explicit allow from `LabIamReadOnly`)
- `iam:ListRoles` = **Allowed** (explicit allow from `LabIamReadOnly`)
- `iam:ListPolicies` = **Allowed** (explicit allow from `LabIamReadOnly`)
- `iam:CreateUser` = **Denied** (explicit deny from `LabIamDenyUserCreate`)
- `iam:DeleteUser` = **Denied** (implicit deny—no policy allowed it)

**Key rule proven:** Explicit deny overrides allow, and anything not explicitly allowed is denied by default.

---

### Where Permissions Come From (High Level)

**Identity-based policies** attached to:
- The **user** (example: `LabIamReadOnly`, `LabIamDenyUserCreate`)
- A **group** the user is in (group policies apply to members)
- A **role** (permissions you get when you assume it)

**Permission boundaries** (if used):
- Limit the **maximum** permissions an identity can have, even if other policies allow more

**SCPs (AWS Organizations)**:
- Can restrict what an entire **account** or **OU** can do, even if IAM policies would allow it

---

### Troubleshooting "Should Be Allowed but Is Denied"

When debugging permission denials, check in this order:
1. Check the user's attached policies
2. Check group memberships and group policies
3. Search for **explicit deny** statements
4. Check whether a **permission boundary** is set
5. If the account is in AWS Organizations, check for **SCPs**

**Takeaway:** Permission evaluation follows a specific order—explicit denies always win, and multiple policy sources can affect the final decision.

---

## Feb 11, 2026

### CloudTrail + Support Flow (IAM / AccessDenied)

* I opened **CloudTrail → Event history** and filtered to **Last 1 hour** to find a real event from today.
* I selected an event named **AssumeRole** with **eventSource = sts.amazonaws.com**.
* I learned **AssumeRole** means STS issued temporary credentials for a role session.
* In this event, `userIdentity.type` was **AWSService** and `userIdentity.invokedBy` was **resource-explorer-2.amazonaws.com`, which means an AWS service initiated the call.
* The record showed the **role being assumed** (`requestParameters.roleArn`) was:
  `arn:aws:iam::886219357247:role/aws-service-role/resource-explorer-2.amazonaws.com/AWSServiceRoleForResourceExplorer`
* The record showed the resulting **assumed role session ARN** (`responseElements.assumedRoleUser.arn`) was:
  `arn:aws:sts::886219357247:assumed-role/AWSServiceRoleForResourceExplorer/resource-explorer-2`
* I confirmed there was **no `errorCode`**, so the role assumption succeeded.
* I noticed not every event includes `userIdentity.arn` or `sessionContext.sessionIssuer`. For AWSService events, the useful identifiers were `invokedBy`, `roleArn`, and the assumed role session ARN.
* CloudTrail is useful because it answers: **who/what made the call, what API action happened, when, from where, and whether it succeeded**.
* Support workflow: start from the **denied action** in the error (ex: `iam:CreateUser`), then use CloudTrail to confirm the **principal context** (user/role/service) and the **exact API call**, then check permission layers: identity policies, group policies, explicit denies, permission boundaries, and any org SCPs.


