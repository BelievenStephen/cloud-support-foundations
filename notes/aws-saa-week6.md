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
## Mar 13, 2026

### ALB Health Checks and 5xx Error Triage

**Focus:**
- Understanding ALB architecture and traffic flow
- Distinguishing ELB-generated vs target-generated errors
- Systematic troubleshooting for 502/503 and 5xx spikes

---

### ALB Architecture Review

**ALB traffic flow:**
1. Listener (protocol + port)
2. Rule (host/path matching)
3. Forward to target group
4. Target group distributes to targets

**ALB configuration components:**

**Scheme:**
- Internet-facing: Public IPs, accessible from internet
- Internal: Private IPs only, accessible within VPC

**Network configuration:**
- VPC: Where ALB deployed
- Subnets: Must be in at least 2 AZs
- Security groups: Control inbound traffic to ALB

**Listeners:**
- Protocol: HTTP, HTTPS
- Port: 80, 443, custom
- Default action: Forward to target group

---

### Target Group Configuration

**Target group examined in lab:**

**Basic settings:**
- Protocol: HTTP or HTTPS
- Port: Application listening port (e.g., 80, 8080)
- Target type: Instance, IP, Lambda
- VPC: Must match ALB VPC

**Health check settings:**
- Protocol: HTTP or HTTPS
- Path: Health check endpoint (e.g., `/health`, `/`)
- Success codes: Expected HTTP response codes (e.g., `200`, `200-299`)
- Interval: Seconds between health checks (default 30s)
- Timeout: Seconds to wait for response (default 5s)
- Healthy threshold: Consecutive successes needed (default 5)
- Unhealthy threshold: Consecutive failures needed (default 2)

---

### Target Health Status

**Targets tab monitoring:**

**Health status indicators:**
- **Healthy:** Target passing health checks
- **Unhealthy:** Target failing health checks
- **Initial:** Registration in progress
- **Draining:** Connection draining during deregistration
- **Unused:** Target not receiving traffic

**Health check failure reasons:**
- Target not responding on health check port/path
- Health check returning wrong status code
- Security group blocking health check traffic
- Application not listening on configured port
- Health check timeout exceeded

---

### CloudWatch Metrics for ALB

**Metrics location:**
- CloudWatch → Metrics → ApplicationELB
- Dimensions: LoadBalancer, TargetGroup

**Key metrics for troubleshooting:**

**Traffic metrics:**
- **RequestCount:** Total requests received by ALB
- **ActiveConnectionCount:** Current connections to ALB

**Latency metrics:**
- **TargetResponseTime:** Time for target to respond
- Helps identify slow backend performance

**Health metrics:**
- **HealthyHostCount:** Number of healthy targets
- **UnHealthyHostCount:** Number of unhealthy targets

**Error metrics:**
- **HTTPCode_ELB_5XX:** 5xx errors generated by ALB
- **HTTPCode_Target_5XX:** 5xx errors returned by targets
- **HTTPCode_ELB_502_Count:** Specific 502 errors from ALB
- **HTTPCode_ELB_503_Count:** Specific 503 errors from ALB

---

### Understanding 5xx Error Sources

**Two sources of 5xx errors:**

**1) HTTPCode_ELB_5XX (ALB-generated):**
- ALB cannot connect to targets
- No healthy targets available
- Connection refused by target
- Target not responding

**2) HTTPCode_Target_5XX (Target-generated):**
- Application error (500 Internal Server Error)
- Application returning 5xx response
- Backend dependency failure
- Application overload

**Critical distinction:** High ELB 5xx = infrastructure/connectivity issue; High Target 5xx = application issue

---

### 502 vs 503 Error Interpretation

**502 Bad Gateway:**
- Target connection problem
- Target returned invalid response
- Target closed connection prematurely
- Common causes:
  - Application crashed
  - Application returned malformed response
  - Network issue between ALB and target

**503 Service Unavailable:**
- No healthy targets available
- All targets unhealthy
- Target group has no registered targets
- Common causes:
  - Health checks failing
  - All instances stopped/terminated
  - Auto Scaling scaled to zero
  - Targets deregistering

---

### Support Ticket Template 1: ALB Returns 502/503

**Symptom:**
- Users report 502 or 503 errors from ALB DNS name
- Or CloudFront origin returning 502/503

---

**Troubleshooting checks (in order):**

**1) Confirm error scope:**
- One Availability Zone or all AZs?
- One path/host rule or all rules?
- Helps isolate to specific target group or infrastructure

**2) Identify target group:**
- ALB → Listeners → Rules
- Which target group receives traffic for failing requests?
- Note: Different paths/hosts may route to different target groups

**3) Check target health:**
- Target Group → Targets tab
- **HealthyHostCount = 0?** → No healthy targets (503 likely)
- Targets marked **Unhealthy?** → Check health check reason

**4) Verify health check configuration:**
- Target Group → Health checks tab

**Settings to verify:**

| Setting | Common Issue |
|---------|--------------|
| Protocol | HTTP vs HTTPS mismatch |
| Port | Wrong port for health check |
| Path | Health check endpoint doesn't exist |
| Success codes | App returns 200 but expecting 200-299 |

**5) Check networking between ALB and targets:**

**Security Group rules:**
- Target SG must allow inbound from ALB SG
- On application port (e.g., 80, 8080)

**Target application:**
- Verify application listening on expected port
- Check application logs for health check requests

**6) Review CloudWatch metrics:**

**HTTPCode_ELB_5XX rising:**
- ALB cannot successfully connect to healthy targets
- Check HealthyHostCount dropping

**HealthyHostCount = 0:**
- Confirms no capacity
- Root cause likely in health check or target configuration

---

**Error interpretation:**

| Error Code | Typical Root Cause |
|------------|-------------------|
| 503 | No healthy targets available |
| 502 | Target connection/response problem (bad gateway) |

---

**Verification after fix:**
```bash
curl https://alb-dns-name.region.elb.amazonaws.com
```
**Expected:** HTTP 200 or 301 response

**CloudWatch confirmation:**
- HealthyHostCount > 0
- HTTPCode_ELB_5XX returning to baseline

---

### Support Ticket Template 2: 5xx Error Spike

**Symptom:**
- Error rate spike
- Dashboards show 5xx errors increasing

---

**Troubleshooting checks (in order):**

**1) Split error source in CloudWatch:**
- Compare **HTTPCode_ELB_5XX** vs **HTTPCode_Target_5XX**
- Identifies whether ALB or application is source

---

**2) If HTTPCode_Target_5XX is high:**

**Application is failing - check:**
- Application logs for errors
- Recent deployments/changes
- Backend dependencies (database, APIs)

**Supporting metrics:**
- **TargetResponseTime:** Latency increasing?
- **RequestCount:** Traffic load increasing?

**Common causes:**
- Application bug introduced in deployment
- Database connection exhaustion
- External API timeout/failure
- Memory/CPU exhaustion on targets

---

**3) If HTTPCode_ELB_5XX is high:**

**Infrastructure/connectivity issue - check:**

**HealthyHostCount:**
- Did it drop toward zero?
- Targets failing health checks?

**Target registration events:**
- Recent instance terminations?
- Auto Scaling scale-in event?
- Manual deregistration?

**Security Group rules:**
- ALB SG allows inbound from clients
- Target SG allows inbound from ALB SG
- On correct ports

**Health check configuration:**
- Path/port mismatch
- Timeout too aggressive
- Success codes don't match app response

---

**4) Confirm if spike limited to specific rule:**

**Check per-rule metrics:**
- Host-based rule forwarding to specific target group
- Path-based rule forwarding to specific target group

**If isolated to one rule:**
- Focus on that target group's targets
- Compare healthy vs unhealthy host counts
- Verify health check configuration for that target group

---

**Verification after fix:**

**CloudWatch metrics:**
- 5xx metrics return to baseline
- HealthyHostCount stable
- TargetResponseTime normal

**Test affected paths:**
```bash
# Test specific host rule
curl -H "Host: api.example.com" https://alb-dns-name/

# Test specific path rule
curl https://alb-dns-name/api/v1/health
```
**Expected:** HTTP 200 or expected response code

---

### CloudWatch Metrics Interpretation

**Healthy capacity indicators:**

| Metric | Healthy State | Unhealthy State |
|--------|--------------|-----------------|
| HealthyHostCount | > 0 (ideally 2+) | 0 or near 0 |
| UnHealthyHostCount | 0 | > 0 |
| HTTPCode_ELB_5XX | Low/zero | Elevated |
| TargetResponseTime | Stable | Increasing or spiking |

---

**Error rate analysis pattern:**

**Both ELB and Target 5xx low:**
- System healthy
- No action needed

**ELB 5xx high, Target 5xx low:**
- Connectivity/health check issue
- Focus on infrastructure layer

**ELB 5xx low, Target 5xx high:**
- Application issue
- Focus on app layer, logs, dependencies

**Both ELB and Target 5xx high:**
- Cascading failure
- Start with HealthyHostCount
- Then investigate application

---

### Health Check Best Practices

**Health check endpoint design:**
- Dedicated health check path (e.g., `/health`)
- Lightweight check (fast response)
- Validates critical dependencies (database, cache)
- Returns 200 on success, 5xx on failure

**Health check configuration:**
- Interval: 30s typical (balance between quick detection and overhead)
- Timeout: 5s typical (must be less than interval)
- Healthy threshold: 2-5 (default 5, lower for faster recovery)
- Unhealthy threshold: 2 (default, quick failure detection)

**Success codes:**
- Use specific codes (e.g., `200`) for strict validation
- Or range (e.g., `200-299`) for flexible validation

---

### Networking Requirements: ALB to Targets

**Security Group rule requirements:**

**ALB Security Group:**
- Inbound: Allow HTTP/HTTPS from clients (0.0.0.0/0 for internet-facing)
- Outbound: Allow traffic to target SG on application port

**Target Security Group:**
- Inbound: Allow traffic from ALB SG on application port
- Example: TCP 8080 from sg-alb-xxxxx

**Common mistake:**
- Target SG allows port 80 from 0.0.0.0/0
- But ALB uses different port (e.g., 8080)
- Health checks fail, targets marked unhealthy

---

### Troubleshooting Decision Tree

**User reports 502/503 from ALB:**

**Step 1: Check HealthyHostCount**
- = 0 → All targets unhealthy (likely 503)
- > 0 but < expected → Some targets unhealthy (intermittent errors)
- = expected → Target configuration issue (likely 502)

**Step 2: If HealthyHostCount = 0**
- Check health check configuration (path/port/codes)
- Check target SG allows ALB SG on app port
- Check application listening on port
- Review target health check failure reasons

**Step 3: If HealthyHostCount > 0 but errors persist**
- Check target application logs
- Verify target responses valid HTTP
- Check TargetResponseTime for latency
- Test direct connection to target (if accessible)

---

### Key Takeaways

**ALB architecture:**
- Listener → Rule (host/path) → Target Group → Targets
- Each target group has own health check settings
- Targets must pass health checks to receive traffic

**Error source identification:**
- **HTTPCode_ELB_5XX:** ALB cannot reach healthy targets (infrastructure)
- **HTTPCode_Target_5XX:** Application returning errors (application)
- Always compare both metrics first

**502 vs 503 distinction:**
- **502:** Target connection/response problem (bad gateway)
- **503:** No healthy targets available

**Critical metrics:**
- **HealthyHostCount:** Must be > 0 for traffic
- **TargetResponseTime:** Indicates application performance
- **RequestCount:** Shows traffic load

**Health check importance:**
- Determines target eligibility
- Misconfigured health checks = all targets unhealthy = 503 errors
- Verify protocol, port, path, success codes match application

**Troubleshooting order for 502/503:**
1. Confirm scope (which AZ/rule)
2. Identify target group
3. Check HealthyHostCount
4. Verify health check settings
5. Verify networking (SG rules)
6. Check application listening on port

**Troubleshooting order for 5xx spike:**
1. Split ELB vs Target 5xx
2. If Target 5xx high: Check application
3. If ELB 5xx high: Check HealthyHostCount and health checks
4. Confirm if isolated to specific rule/target group

**Security Group requirements:**
- Target SG must allow ALB SG on application port
- Both health checks and actual traffic use same port
- Misconfigured SG = health checks fail = targets unhealthy

---
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

