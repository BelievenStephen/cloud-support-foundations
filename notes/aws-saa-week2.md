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

---

## Feb 12, 2026

### VPC Basics Review (Console Walkthrough)

**Components reviewed:**
- Subnets
- Route Tables
- Internet Gateways
- NAT Gateways
- Security Groups
- Network ACLs

---

### Subnets Examined

I looked at two subnets in the VPC console:

**Subnet 1:**
- ID: `subnet-0ed1fa4a40bc62a26`
- Availability Zone: `us-west-1c`
- CIDR: `172.31.0.0/20`

**Subnet 2:**
- ID: `subnet-0b7f3fdc246069fbe`
- Availability Zone: `us-west-1a`
- CIDR: `172.31.16.0/20`

**Routing configuration:**
- Both subnets associated with route table containing: `0.0.0.0/0 -> igw-...`
- This means they have a direct path to the internet (public routing)

**Key insight:** "Public vs private subnet" is mainly determined by the route table default route (IGW vs NAT vs no default route), not the subnet name.

---

### Internet Gateway

**Found:**
- Internet Gateway: `igw-0d2b4afba531a9b3b`
- Attached to VPC: `vpc-0622cb5ac599f4c36`
- This is what the route tables use for internet access

---

### NAT Gateways

**Status:**
- No NAT Gateways present
- This environment does not currently have the "private subnet outbound via NAT" pattern configured

---

### Security Groups

**Observed behavior:**
- Default-style configuration: outbound allowed to `0.0.0.0/0` (all traffic)

**Example rules noted:**
- `sgr-0317186940658c253`: Inbound reference to default SG
- `sgr-026bdd4b41afffbed`: Outbound all traffic

---

### Network ACLs

**Configuration:**
- Only one Network ACL present
- Inbound rules: Allow rule + Deny rule (typical default pattern)
- Outbound rules: Allow rule + Deny rule (typical default pattern)

**Key learning:** NACLs are stateless and ordered, so they can block traffic even if the Security Group looks correct.

---

### Support Mapping: "Instance Unreachable" (SSH/RDP/App Port)

**Troubleshooting checklist:**

1. **Check instance Security Group:**
   - Verify inbound rule exists for the required port
   - Confirm source IP/CIDR is allowed

2. **Check subnet route table:**
   - Verify default route (`0.0.0.0/0`) exists
   - Confirm target (IGW vs NAT)

3. **Verify VPC has IGW attached:**
   - Required if public access is expected

4. **Check Network ACL:**
   - Verify both inbound AND outbound rules
   - Remember: NACLs are stateless (both directions matter)

---

### Support Mapping: "DNS Works but App Fails"

**Approach:** Treat as not-DNS issue

**Troubleshooting checklist:**

1. **Check Security Group rules:**
   - Verify app port is allowed (80/443/custom)
   - Confirm source is permitted

2. **Verify service health on instance:**
   - Confirm service is listening on the port
   - Check application health status

3. **Check routing path:**
   - Verify subnet route table configuration
   - Check any upstream components (ALB/target group if used)

---

### VPC Takeaways

**Core concepts:**

- **Route tables** answer: "Where can packets go?"
- **Security Groups** answer: "Which ports are allowed?"
- **Network ACLs** answer: "What can the subnet filter block?"

**Key relationships:**

| Component | Function | Scope |
|-----------|----------|-------|
| Route Table | Determines packet destination | Subnet-level |
| Security Group | Filters traffic by port/protocol | Instance-level, stateful |
| Network ACL | Ordered filter rules | Subnet-level, stateless |
| Internet Gateway | Enables internet access | VPC-level |
| NAT Gateway | Enables private subnet outbound | Subnet-specific |

---

