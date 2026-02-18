# Runbook: Instance Has No Internet Access

## Goal
Restore outbound internet connectivity for an EC2 instance that cannot reach external sites.

## Symptoms
- Package installs fail (`apt update`, `yum update`)
- `curl` to external sites times out
- Application cannot reach external APIs

**Example checks:**
```bash
curl -I --connect-timeout 5 --max-time 10 http://example.com
# Times out or hangs

dig example.com +short
# May succeed (DNS works) even when curl fails
```

## Key Context to Capture First
- Instance ID
- Instance private IP
- Subnet ID
- VPC ID
- Does the instance have a public IPv4 or Elastic IP?
- Subnet route table ID and default route target (IGW vs NAT vs none)

## Troubleshooting Flow (In Order)

### 1) Confirm Instance Addressing

**On the instance:**
```bash
# Check private IP
ip -4 a | grep inet

# Check for public IPv4 (from instance metadata)
curl -s http://169.254.169.254/latest/meta-data/public-ipv4 ; echo
```

**Key decision:**
- **No public IPv4:** Instance must use NAT Gateway to reach the internet
- **Has public IPv4:** Instance can use IGW directly (if route table configured)

---

### 2) Check Subnet Route Table (Most Common Root Cause)

**VPC Console → Route tables → Select route table associated with the subnet**

**Verify default route:**

| Subnet Type | Expected Route | What It Means |
|-------------|---------------|---------------|
| Public subnet | `0.0.0.0/0 → igw-...` | Direct internet access via IGW |
| Private subnet (with internet) | `0.0.0.0/0 → nat-...` | Internet access via NAT Gateway |
| No internet | Missing `0.0.0.0/0` or incorrect target | No internet path configured |

**Common issues:**
- Route table has no `0.0.0.0/0` entry
- Route points to a deleted/detached NAT or IGW
- Wrong route table associated with the subnet

---

### 3) Verify NAT Gateway (If Private Subnet)

**VPC Console → NAT gateways**

**NAT Gateway requirements:**
- Status: **Available**
- Subnet: Must be in a **public subnet** (subnet with `0.0.0.0/0 → IGW`)
- Elastic IP: Must have an **EIP attached**

**Common NAT Gateway issues:**
- NAT Gateway does not exist
- NAT Gateway state is not "Available"
- NAT Gateway is in a private subnet (cannot route to internet)
- NAT Gateway has no Elastic IP attached

---

### 4) Check Security Group Egress Rules

**EC2 → Instance → Security tab → Outbound rules**

**Verify outbound allows:**
- HTTP (port 80)
- HTTPS (port 443)
- Or "All traffic" to `0.0.0.0/0`

**Note:** Default SG allows all outbound traffic. Custom SGs may be restrictive.

---

### 5) Check Network ACLs (Less Common)

**VPC Console → Network ACLs → Select NACL associated with subnet**

**Verify:**
- Outbound rules allow traffic on required ports (80, 443)
- Outbound rules allow ephemeral ports (1024-65535) for return traffic
- Inbound rules allow return traffic (ephemeral ports)

**Remember:** NACLs are stateless — both directions must be explicitly allowed.

---

### 6) DNS Sanity Check (If `dig` Fails)

**On the instance:**
```bash
cat /etc/resolv.conf
```

**Expected:** Nameserver should be VPC DNS resolver (usually `169.254.169.254` or VPC CIDR base +2)

**If DNS is misconfigured:**
- Check VPC DNS settings: VPC Console → Select VPC → Edit DNS resolution/hostnames
- Verify "Enable DNS resolution" and "Enable DNS hostnames" are both enabled

---

## Common Root Cause Patterns

| Issue | Symptoms | Where to Look |
|-------|----------|---------------|
| Private subnet route table missing default route | `curl` times out, no error | Route table → no `0.0.0.0/0` entry |
| NAT Gateway missing or unavailable | `curl` times out | VPC → NAT gateways |
| NAT Gateway in wrong subnet | `curl` times out | NAT Gateway subnet must be public |
| SG blocks outbound | `curl` times out or refused | Security Group outbound rules |
| NACL blocks outbound or return | `curl` times out | Network ACL inbound + outbound rules |

---

## Resolution Steps

### For Private Subnet Instance (No Public IP):

**1) Create NAT Gateway:**
- VPC Console → NAT gateways → Create NAT gateway
- **Subnet:** Select a **public subnet** (one with `0.0.0.0/0 → IGW`)
- **Elastic IP:** Allocate new EIP or use existing

**2) Update Private Route Table:**
- VPC Console → Route tables → Select private subnet's route table
- Add route: `0.0.0.0/0 → nat-xxxxxxxx` (NAT Gateway ID)

**3) Verify Subnet Association:**
- Confirm the private subnet is associated with the updated route table

---

### For Public Subnet Instance (Has Public IP):

**1) Verify IGW Attached:**
- VPC Console → Internet gateways
- Confirm IGW is attached to the VPC

**2) Update Route Table:**
- VPC Console → Route tables → Select public subnet's route table
- Add route: `0.0.0.0/0 → igw-xxxxxxxx` (IGW ID)

**3) Verify Security Group:**
- EC2 → Instance → Security → Outbound rules
- Ensure outbound allows 80/443 or all traffic

---

## Verification

**From the instance, run:**
```bash
# Test HTTP connectivity
curl -I --connect-timeout 5 --max-time 10 http://example.com
# Should return: HTTP/1.1 200 OK (or 301/302)

# Test DNS resolution
dig example.com +short
# Should return: IP addresses

# Test HTTPS connectivity
curl -I --connect-timeout 5 --max-time 10 https://example.com
# Should return: HTTP response headers
```

**Success indicators:**
- `curl` returns HTTP response (not timeout)
- `dig` resolves domain names
- Package manager commands work (`apt update`, `yum update`)

---

## Notes

**Private subnet internet access flow:**
- Instance → Private subnet → Route table (`0.0.0.0/0 → NAT`) → NAT Gateway (in public subnet) → Route table (`0.0.0.0/0 → IGW`) → IGW → Internet

**Public subnet internet access flow:**
- Instance (with public IP) → Public subnet → Route table (`0.0.0.0/0 → IGW`) → IGW → Internet

**Common mistakes:**
- NAT Gateway in a private subnet (NAT must be in public subnet)
- Private route table points to IGW instead of NAT (instances without public IPs cannot use IGW directly)
- Editing the wrong route table (check which route table is associated with the subnet)

---

## Cost Management and Lab Cleanup

### NAT Gateway Costs
NAT Gateways incur charges while they exist:
- **Hourly charge:** Billed per hour the NAT Gateway exists (even if unused)
- **Data processing charge:** Billed per GB of data processed through the NAT Gateway

**Important:** Charges accrue even when the NAT Gateway is idle. Always clean up after testing.

---

### Cleanup Order (After Testing)

Follow this order to avoid leaving billable resources:

**1) Delete NAT Gateway**
- VPC Console → NAT gateways
- Select NAT Gateway → Actions → Delete NAT gateway
- Wait until State = **Deleted** (may take a few minutes)

**2) Release Elastic IP**
- VPC Console → Elastic IPs
- Select EIP that was allocated for the NAT Gateway
- Actions → Release Elastic IP address
- Verify: 0 Elastic IPs listed (or all are in use by other resources)

**3) Clean Up Route Table (Optional)**
- VPC Console → Route tables
- Select private subnet's route table
- Routes tab → Select `0.0.0.0/0 → nat-...` → Delete route
- Note: Only remove if you're done testing private subnet internet access

**4) Terminate Lab Instances**
- EC2 Console → Instances
- Select lab instances → Instance state → Terminate instance
- Verify: No unattached EBS volumes remain (check Volumes section)

---

### Quick "No Silent Charges" Verification

Run this checklist before ending your session:

| Resource | Where to Check | Expected State |
|----------|---------------|----------------|
| NAT Gateways | VPC → NAT gateways | Empty list or all "Deleted" |
| Elastic IPs | VPC → Elastic IPs | Empty list or all associated with active resources |
| Unattached Volumes | EC2 → Volumes | Filter by "available" → should be empty |
| Running Instances | EC2 → Instances | Only instances you intend to keep running |

**Tip:** Set up a billing alert in AWS Budgets to catch unexpected charges early.

---
