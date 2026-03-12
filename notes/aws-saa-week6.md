# AWS SAA Study Notes - Week 6

## Mar 10, 2026

### NACL vs Security Group Reachability Triage

**Focus:**
- Understanding NACL behavior and rule evaluation
- Distinguishing NACL vs Security Group issues
- Systematic troubleshooting approach

---

### Lab NACL Configuration Review

**Subnet examined:** `subnet-0ed1fa4a40bc62a26`

**Associated NACL:** `acl-0c140946f97db02aa`

**NACL type verification:**
- Default NACL: Yes
- Indicates "allow all" behavior

---

### NACL Rule Analysis

**Inbound rules:**
- Rule 100: ALLOW all traffic
- Rule `*`: DENY all
- Result: Inbound SSH not blocked by NACL

**Outbound rules:**
- Rule 100: ALLOW all traffic
- Rule `*`: DENY all
- Result: Outbound and return traffic not blocked by NACL

**NACL association scope:**
- Default NACL associated with multiple subnets
- Includes: public subnet and `lab-private-1`
- Note: NACL differences won't explain reachability issues unless subnet attached to different custom NACL

---

### Security Group vs NACL Key Differences

**Security Groups:**
- **Stateful:** Return traffic automatically allowed
- Evaluate all rules before allowing traffic
- Support allow rules only
- Applied at instance level

**Network ACLs:**
- **Stateless:** Must explicitly allow both directions
- Evaluated by rule number (lowest match wins)
- Support both allow and deny rules
- Applied at subnet level

---

### Why Custom NACLs Cause SSH Timeouts

**Common failure pattern:**
- Security Group correctly allows TCP 22 inbound
- SSH still times out

**Root cause:**
- Custom NACL allows inbound port 22
- BUT blocks ephemeral return ports (1024-65535)
- TCP handshake or session traffic never completes

**Key insight:**
- Stateless nature requires explicit rules for return traffic
- Ephemeral ports must be allowed for responses

---

### Reachability Triage Order

**Systematic troubleshooting approach:**

1. **Route table** → Verify IGW or NAT route
2. **Public IP** → If IGW path, confirm instance has public IP
3. **Security Group rules** → Check inbound/outbound as needed
4. **NACL rules** → Verify both port 22 AND ephemeral (1024-65535)
5. **Instance-side causes** → SSH daemon, OS firewall, etc.

---

### Support Ticket Template 1: SSH Timeout - NACL Ephemeral Blocked

**Symptom:**
- SSH to `<public-ip>` hangs then times out

---

**Troubleshooting checks (in order):**

**1) Confirm network path:**
- Instance has public IPv4
- Subnet route table has `0.0.0.0/0 → igw-...`

**2) Confirm Security Group:**
- Inbound rule allows TCP 22 from source IP/32

**3) Identify NACL type:**
- Open subnet's NACL
- Confirm whether default or custom
- Lab example: Subnet `subnet-0ed1fa4a40bc62a26` uses default NACL `acl-0c140946f97db02aa`

**4) If custom NACL, verify:**

**Inbound rules:**
- Allow TCP 22 from source IP/32
- No earlier deny rule matches

**Inbound + Outbound rules:**
- Allow ephemeral ports 1024-65535 for return traffic
- Remember: NACL is stateless (both directions required)

---

**Root cause example:**
- Custom NACL allows inbound port 22
- NACL blocks ephemeral return ports
- TCP handshake or session traffic never completes

---

**Resolution:**
- Add ALLOW rules for ephemeral range (1024-65535) in required directions
- OR temporarily use allow-all during lab testing

---

**Verification:**
```bash
ssh -i key.pem ec2-user@<public-ip>
```
**Expected:** SSH succeeds immediately after NACL update

---

### Support Ticket Template 2: Subnet A Works, Subnet B Doesn't

**Symptom:**
- Instances in subnet A can reach internet/SSH
- Instances in subnet B cannot

---

**Troubleshooting checks (in order):**

**1) Confirm expected route type:**
- Both subnets have correct route table
- IGW for public subnets
- NAT for private subnet egress

**2) Confirm Security Groups equivalent:**
- Same or equivalent rules between subnets

**3) Compare NACL associations:**
- Lab VPC example: Default NACL `acl-0c140946f97db02aa` associated with:
  - Multiple subnets including `subnet-0ed1fa4a40bc62a26`
  - And `lab-private-1`

**4) If behavior differs, investigate:**

**Route table differences:**
- IGW vs NAT vs missing default route

**Public IPv4 presence:**
- Required for IGW path
- Not required for NAT path

**Security Group rules:**
- Verify rules match between working/broken

**Different custom NACL:**
- Check if "broken" subnet uses different NACL

---

**Root cause example:**
- Subnet B uses custom NACL
- Custom NACL blocks outbound port 443
- OR blocks ephemeral return ports

---

**Resolution:**
- Align NACL rules between subnets
- OR associate correct NACL to subnet

---

**Verification:**
```bash
# Test HTTP connectivity
curl -I http://example.com

# Test SSH (if applicable)
ssh -i key.pem ec2-user@<public-ip>
```
**Expected:** Both work after NACL correction

---

### NACL Rule Evaluation

**Rule processing:**
- Rules evaluated by number (lowest to highest)
- First matching rule wins
- Processing stops at first match

**Example rule order:**

| Rule # | Type | Protocol | Port Range | Source | Allow/Deny |
|--------|------|----------|------------|--------|------------|
| 100 | ALL | ALL | ALL | 0.0.0.0/0 | ALLOW |
| * | ALL | ALL | ALL | 0.0.0.0/0 | DENY |

**Result:** All traffic allowed (rule 100 matches first)

---

**Custom NACL example (problematic):**

| Rule # | Type | Protocol | Port Range | Source/Dest | Allow/Deny |
|--------|------|----------|------------|-------------|------------|
| 100 | SSH | TCP | 22 | 0.0.0.0/0 | ALLOW |
| 200 | Custom TCP | TCP | 1024-65535 | 0.0.0.0/0 | DENY |
| * | ALL | ALL | ALL | 0.0.0.0/0 | DENY |

**Problem:** SSH handshake allowed inbound, but return traffic (ephemeral) denied

---

### Ephemeral Port Requirements

**What are ephemeral ports:**
- Temporary ports used for return traffic
- Range: 1024-65535
- Operating system assigns dynamically

**Why they matter for NACL:**
- Client initiates SSH on port 22
- Server responds using ephemeral port (e.g., 52341)
- NACL must allow both directions

**Required NACL rules for SSH:**

**Inbound (to instance):**
- Allow TCP 22 from client IP

**Outbound (from instance):**
- Allow TCP 1024-65535 to client IP (return traffic)

**Inbound (to instance - for return traffic if client inside VPC):**
- Allow TCP 1024-65535 from server IP (if applicable)

---

### Troubleshooting Decision Matrix

**"SSH times out" root cause finder:**

| Route Table | Public IP | SG Allows 22 | NACL Allows 22 | NACL Allows Ephemeral | Likely Issue |
|-------------|-----------|--------------|----------------|----------------------|--------------|
| ✅ IGW | ✅ Yes | ✅ Yes | ✅ Yes | ✅ Yes | Check instance (sshd down) |
| ✅ IGW | ✅ Yes | ✅ Yes | ✅ Yes | ❌ No | **NACL ephemeral blocked** |
| ✅ IGW | ✅ Yes | ✅ Yes | ❌ No | N/A | **NACL port 22 blocked** |
| ✅ IGW | ✅ Yes | ❌ No | Any | Any | **SG blocks SSH** |
| ✅ IGW | ❌ No | Any | Any | Any | **No public IP** (can't use IGW) |
| ❌ No IGW | Any | Any | Any | Any | **Route table missing IGW** |

---

### Common NACL Mistakes

**Mistake 1: Forgetting ephemeral ports**
- Allow inbound 22 only
- Forget outbound ephemeral return
- Result: Connection hangs

**Mistake 2: Wrong rule order**
- Deny rule with lower number than allow
- Deny matches first, blocks traffic
- Result: Unexpected blocking

**Mistake 3: Treating NACL like Security Group**
- Only configure inbound rules
- Assume return traffic automatic (it's not)
- Result: One-way traffic only

**Mistake 4: Testing with wrong subnet**
- Different subnets may have different NACLs
- Assume all subnets same
- Result: Inconsistent behavior across subnets

---

### Best Practices

**For production:**
- Use Security Groups as primary firewall (stateful simplicity)
- Use NACLs as secondary defense layer
- Default NACL usually sufficient (allow all)
- Only use custom NACLs when explicit deny needed

**For troubleshooting:**
- Always check NACL after Security Group
- Verify ephemeral ports allowed (1024-65535)
- Check rule order (lowest number wins)
- Confirm which NACL associated with subnet

**For lab testing:**
- Start with default NACL (allow all)
- Test Security Group rules first
- Only add custom NACL restrictions after SG working
- Document which subnets use which NACLs

---

### Quick Reference: NACL vs SG

| Aspect | Security Group | Network ACL |
|--------|---------------|-------------|
| **Level** | Instance | Subnet |
| **State** | Stateful | Stateless |
| **Rules** | Allow only | Allow and Deny |
| **Evaluation** | All rules | First match wins |
| **Return traffic** | Automatic | Must explicitly allow |
| **Default** | Deny all inbound, allow all outbound | Default NACL allows all |
| **Rule order** | No specific order | Numbered (lowest first) |
| **Best for** | Application-level control | Subnet-level defense |

---

### Key Takeaways

**NACL fundamentals:**
- Stateless firewall at subnet level
- Must explicitly allow both directions
- Ephemeral ports (1024-65535) required for return traffic
- Rule evaluation by number (lowest wins)

**Security Group fundamentals:**
- Stateful firewall at instance level
- Return traffic automatic
- Evaluate all rules before decision
- Allow rules only

**Troubleshooting order:**
1. Route table (IGW/NAT)
2. Public IP presence (if IGW)
3. Security Group rules
4. NACL rules (both port and ephemeral)
5. Instance-level issues

**Common NACL failure modes:**
- Allow inbound 22, block ephemeral outbound
- Deny rule with lower number than allow
- Different NACLs per subnet causing inconsistent behavior
- Treating NACL as stateful (forgetting return traffic)

**Ticket template patterns:**
- SSH timeout: Check NACL ephemeral ports
- Subnet A works, B doesn't: Check NACL associations
- All subnets broken: Check route table or SG first

---

## Mar 12, 2026

### Lab Exercise: NACL Break-Fix with Deny-All Association

**Objective:** Demonstrate impact of deny-all NACL on existing and new connections

---

### Baseline Verification

**SSH connectivity test:**
```bash
ssh -i lab-unreachable-key-pair.pem ec2-user@<REDACTED>
```
**Result:** Successfully connected

**HTTP egress test:**
```bash
curl -I http://example.com
```
**Result:** `HTTP/1.1 200 OK`

**Original NACL configuration:**
- Subnet: `subnet-0ed1fa4a40bc62a26`
- Associated NACL: `acl-0c140946f97db02aa` (default)
- VPC: `vpc-0622cb5ac599f4c36`

---

### Breaking Change: Create and Apply Deny-All NACL

**Custom NACL created:**
- Name: `lab-deny-all-nacl-0312`
- VPC: `vpc-0622cb5ac599f4c36`

**NACL rules configured:**

**Inbound rules:**
- Rule `*`: DENY all traffic
- No allow rules

**Outbound rules:**
- Rule `*`: DENY all traffic
- No allow rules

**Action taken:**
- Associated `subnet-0ed1fa4a40bc62a26` to `lab-deny-all-nacl-0312`
- Replaced default NACL association

---

### Impact After NACL Change

**Existing SSH session:**
- Session froze
- Dropped with error: `Read from remote host ... Operation timed out`
- Additional error: `Broken pipe`

**New SSH attempts:**
```bash
ssh -i lab-unreachable-key-pair.pem ec2-user@<REDACTED>
```
**Result:** Connection timeout

**Key observation:**
- Deny-all NACL immediately breaks even established connections
- Unlike Security Group changes, NACL affects existing sessions
- Stateless nature means return traffic blocked

---

### Resolution Applied

**Re-association to default NACL:**
- Associated `subnet-0ed1fa4a40bc62a26` back to default NACL `acl-0c140946f97db02aa`
- Waited briefly for change to propagate

---

### Verification After Fix

**SSH connectivity:**
```bash
ssh -i lab-unreachable-key-pair.pem ec2-user@<REDACTED>
```
**Result:** Successfully connected

**HTTP egress test:**
```bash
curl -I http://example.com
```
**Result:** `HTTP/1.1 200 OK`

**Connectivity restored immediately after NACL re-association**

---

### Cleanup

**NACL cleanup verification:**
1. Confirmed `lab-deny-all-nacl-0312` has no subnet associations
2. Deleted custom NACL `lab-deny-all-nacl-0312`

**Final state:**
- Subnet `subnet-0ed1fa4a40bc62a26` associated with default NACL
- No orphaned custom NACLs

---

### Key Observations

**NACL vs Security Group behavior:**

| Aspect | Security Group | Network ACL |
|--------|----------------|-------------|
| **Affects existing connections** | No (stateful) | Yes (stateless) |
| **Connection drop behavior** | Existing connections continue | Existing connections drop |
| **Change impact timing** | Immediate but only new connections | Immediate including active sessions |

**Deny-all NACL impact:**
- Blocks all inbound traffic (including SSH handshakes)
- Blocks all outbound traffic (including SSH return packets)
- Existing sessions cannot send/receive packets
- Results in frozen connection then timeout/broken pipe

**Stateless vs stateful firewall:**
- Security Group (stateful): Tracks connections, allows return traffic
- NACL (stateless): Evaluates every packet independently
- NACL deny blocks both directions, no connection state awareness

---

### Troubleshooting Lesson

**"SSH was working, now frozen and dropped":**
- Sudden connection freeze → Check for NACL changes
- Broken pipe error → Likely NACL blocking return traffic
- Different from timeout pattern (which suggests connection never established)

**Diagnostic pattern:**

| Symptom | Likely Cause |
|---------|--------------|
| **Connection freezes then drops with "Broken pipe"** | NACL change blocked active session |
| **New connections timeout immediately** | NACL or SG blocking initial handshake |
| **Existing connections continue, new fail** | Security Group change (stateful) |

---

### Lab Workflow Summary

| Step | Action | SSH Status | HTTP Status |
|------|--------|------------|-------------|
| 1. Baseline | Default NACL associated | ✅ Connected | ✅ 200 OK |
| 2. Break | Associate deny-all NACL | ❌ Frozen → Broken pipe | ❌ No connectivity |
| 3. Fix | Re-associate default NACL | ✅ Connected | ✅ 200 OK |
| 4. Cleanup | Delete deny-all NACL | ✅ Complete | ✅ Complete |

**Principle reinforced:** NACL changes affect active connections immediately due to stateless packet evaluation

---

### Real-World Scenario

**Support ticket pattern: "All SSH sessions suddenly dropped"**

**Investigation order:**
1. Check recent NACL association changes
2. Verify current NACL rules (inbound + outbound)
3. Compare with previous working configuration
4. Check for deny-all or missing allow rules

**Common root causes:**
- Accidental association to restrictive NACL
- NACL rule modification removing required allows
- Automation/IaC applying wrong NACL configuration
- Subnet moved to different NACL during network restructure

**Resolution:**
- Re-associate subnet to correct NACL
- Or fix rules in current NACL
- Verify both inbound and outbound rules (stateless requirement)

---

### NACL Best Practices

**Production environment:**
- Document which NACLs associated with which subnets
- Use descriptive NACL names indicating purpose
- Test NACL changes in non-production first
- Implement change control for NACL associations

**Lab/testing:**
- Create custom NACLs with clear temporary naming
- Always verify no subnet associations before deleting
- Keep default NACL as fallback (allow-all)
- Document original NACL associations before changes

**Troubleshooting:**
- Check NACL first if sudden connection drops
- Remember stateless nature (both directions required)
- Compare current vs previous NACL associations
- Test with temporary allow-all to isolate issue

---

### Connection Drop Signature

**Deny-all NACL connection drop characteristics:**
- Connection freezes mid-operation
- Eventually times out
- Error: `Read from remote host ... Operation timed out`
- Error: `Broken pipe`
- No graceful TCP termination

**Why this happens:**
- SSH keepalive packets blocked by NACL
- TCP acknowledgments cannot return
- Session cannot maintain state
- Client detects unresponsive connection
- Timeout triggers connection termination

---

### Quick Reference: NACL Troubleshooting

**Symptoms suggesting NACL issue:**
- ✅ Route table correct (IGW route exists)
- ✅ Security Group allows required ports
- ✅ Instance has public IP
- ✅ Instance status checks passing
- ❌ All connections still timing out or dropping

**Next step:** Check NACL

**What to verify:**
1. Which NACL associated with subnet
2. NACL inbound rules (must allow required traffic)
3. NACL outbound rules (must allow return traffic)
4. Rule evaluation order (lowest number wins)
5. Recent NACL association changes

---
