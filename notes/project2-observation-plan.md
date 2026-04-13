```markdown
# Project 2 Observation Plan

## Purpose

This document defines what Project 2 will observe, which signals matter most, what constitutes a healthy baseline, and what artifacts will prove the project is working. The goal is to establish clear observation targets before building monitoring and investigation capabilities.

---

## First Observation Target

**Project 2 will observe the existing Project 1 deployment.**

**Target workload components:**
- ECS Fargate service behind Application Load Balancer
- CloudWatch metrics already flowing from ALB and target group
- CloudWatch Logs for running ECS container (`/ecs/aws-hosted-app`)
- CloudTrail activity related to AWS API actions
- IAM-related access and permission checks for the environment

**Why this is the best first target:**
- Already deployed and validated (known good state)
- Already has monitoring infrastructure in place
- Low risk approach - no new infrastructure needed
- Allows focus on support-style visibility and investigation
- Establishes baseline before adding complexity

---

## Signals That Matter First

### Metrics (CloudWatch)

**Priority metrics for initial observation:**

| Metric | Purpose | What It Proves |
|--------|---------|----------------|
| `RequestCount` | Traffic volume | Traffic reaching ALB |
| `TargetResponseTime` | Backend latency | Application responding normally |
| `HTTPCode_ELB_5XX_Count` | ALB errors | Infrastructure health |
| `HealthyHostCount` | Target availability | Capacity exists |

**Why these metrics:**
- Fastest high-level signals for service health
- Answer: "Is traffic reaching app? Is app responding? Is target healthy?"
- Support immediate troubleshooting decisions

---

### Logs (CloudWatch Logs)

**Priority log sources:**

**ECS task logs:** `/ecs/aws-hosted-app`

**Key log patterns to monitor:**

| Pattern | Meaning | Investigation Value |
|---------|---------|-------------------|
| Repeated `/health` `200` responses | Health checks passing | Normal baseline |
| Startup messages | Container initialization | Deployment success |
| Runtime error messages | Application failures | Root cause investigation |
| Changed request patterns | Traffic anomaly | Incident correlation |

**Why these logs:**
- Confirm container running and healthy
- Prove application behaving normally
- Provide error context during incidents

---

### CloudTrail Events

**Priority event types:**

**ECS-related events:**
- `UpdateService` - Service configuration changes
- `RegisterTaskDefinition` - New task definition versions
- `StopTask` - Manual or automatic task terminations

**Load balancer events:**
- `ModifyLoadBalancerAttributes` - ALB configuration changes
- `ModifyTargetGroup` - Target group modifications
- `RegisterTargets` / `DeregisterTargets` - Target registration changes

**IAM-related events:**
- `CreatePolicy` / `AttachUserPolicy` / `PutUserPolicy` - Permission changes
- `CreateRole` / `AttachRolePolicy` - Role modifications
- `AssumeRole` - Role assumption (who accessed what)

**CloudWatch events (if relevant):**
- `PutMetricAlarm` - Alarm creation/modification
- `PutDashboard` - Dashboard changes

**Why these events:**
- Show what changed when something breaks
- Answer "who changed what when"
- Support audit and investigation workflows
- Enable timeline correlation during incidents

---

### IAM-Related Checks

**Priority IAM investigation capabilities:**

**Access audit:**
- Who made a change (principal identification)
- Whether correct IAM user or role performed action
- Whether action was authorized (not accidental)

**Permission verification:**
- Whether access failures caused by missing permissions
- Whether permissions broader than necessary
- Whether explicit denies blocking intended access

**Least-privilege validation:**
- Whether roles follow principle of least privilege
- Whether policies grant only required permissions
- Whether trust relationships appropriately scoped

**Why these checks:**
- Support Project 2 focus on IAM hardening
- Enable support-style access investigation
- Prove principle of least privilege
- Demonstrate audit capability

---

## Healthy Baseline

**What "healthy" looks like for observed target:**

### Service state
```
ECS service metrics:
- Desired count: 1
- Running count: 1
- Pending count: 0
```

### Target health
```
Target group health:
- Target state: healthy
- Health check status: passing
```

### Endpoint validation
```bash
# Health endpoint
curl -I https://aws-hosted-app-alb-<ID>.us-west-1.elb.amazonaws.com/health
# Expected: HTTP/1.1 200 OK

# Root endpoint
curl -I https://aws-hosted-app-alb-<ID>.us-west-1.elb.amazonaws.com/
# Expected: HTTP/1.1 200 OK
```

### Log patterns
```
CloudWatch Logs (/ecs/aws-hosted-app):
- Repeated GET /health 200 responses
- No error messages
- Normal request/response pattern
```

### Metrics baseline
```
CloudWatch metrics:
- RequestCount: Steady traffic pattern
- TargetResponseTime: < 100ms typical
- HTTPCode_ELB_5XX_Count: 0
- HealthyHostCount: 1
```

### Alarm state
```
All alarms: OK state
- aws-hosted-app-alb-5xx-alarm: OK
- aws-hosted-app-healthy-host-count-alarm: OK
```

---

## Artifacts That Prove Project 2 Is Working

**Project 2 will be complete when these artifacts exist:**

### 1) CloudWatch Dashboard
- Dashboard tied to observed workload
- Widgets showing priority metrics (RequestCount, TargetResponseTime, 5xx, HealthyHostCount)
- Clear visualization of baseline vs anomaly

**Location:** `project2/docs/dashboard-setup.md`

---

### 2) CloudWatch Alarms
- Alarms appropriate for workload
- At minimum: HealthyHostCount and ELB 5xx
- Documented alarm response workflows

**Location:** `project2/docs/alarm-configuration.md`

---

### 3) Centralized Logs
- CloudWatch Logs receiving ECS container logs
- Log patterns documented
- Healthy vs unhealthy signatures identified

**Location:** `project2/docs/cloudwatch-logs.md`

---

### 4) CloudTrail Investigation Notes
- CloudTrail configuration documented
- Priority events identified
- Example investigation workflow
- How to query CloudTrail for common scenarios

**Location:** `project2/docs/cloudtrail-setup.md`

---

### 5) IAM Least-Privilege Documentation
- Required permissions documented
- Over-permissioned scenarios identified
- AccessDenied investigation workflow
- Policy evaluation order reference

**Location:** `project2/docs/iam-least-privilege.md`

---

### 6) Support-Style Runbooks
- Runbooks tied to real signals (alarms, logs, CloudTrail)
- Systematic troubleshooting procedures
- Clear symptom → investigation → resolution flows

**Location:** `project2/runbooks/`

**Priority runbooks:**
- `iam-accessdenied.md` (first - differentiates Project 2)
- `alb-5xx-spike.md`
- `unhealthy-targets.md`
- `cloudtrail-investigation.md`

---

### 7) Validation and Evidence Notes
- What healthy baseline looks like
- How to validate project working
- Proof of correct configuration
- Test results and screenshots if helpful

**Location:** `project2/docs/validation-results.md`

---

## First Runbook Priority

**First runbook to write:** `iam-accessdenied.md`

**Why this comes first:**

**Differentiates Project 2:**
- Project 2 adds IAM hardening and CloudTrail investigation
- Not just app deployment (that's Project 1)
- Demonstrates Cloud Support style access troubleshooting

**Supports project goals:**
- IAM least-privilege focus
- CloudTrail investigation capability
- Audit and compliance thinking
- Support-style systematic troubleshooting

**Practical value:**
- Common real-world scenario
- Uses multiple signals (IAM policies, CloudTrail events, Policy Simulator)
- Demonstrates systematic investigation approach

**Runbook will include:**
- Symptoms: AccessDenied errors, permission failures
- Investigation order: Caller identity → policies → boundaries → SCPs
- CloudTrail correlation: Who made permission changes
- Fix steps: Add required permissions, remove conflicting denies
- Verification: Retry action, confirm success

---

## Project 2 Approach

**Core principle:**
- Build on Project 1, don't replace it
- Observe already-working environment first
- Layer on monitoring, CloudTrail, IAM hardening, investigation guidance
- Focus on operational visibility and support capability

**Success criteria:**
- Can detect when observed workload fails (alarms)
- Can investigate failure (logs, CloudTrail, metrics)
- Can prove least privilege (IAM documentation)
- Can demonstrate support-style troubleshooting (runbooks)
- Can answer "who changed what when" (CloudTrail)

**Philosophy:**
- Start with known good state (Project 1 baseline)
- Add observation capabilities systematically
- Document evidence and investigation workflows
- Prove support readiness, not just deployment success

---

## Next Steps

**Immediate next actions:**
1. Confirm Project 1 still healthy (baseline verification)
2. Document current healthy state (metrics, logs, endpoints)
3. Create CloudWatch dashboard for observation
4. Configure CloudWatch alarms for critical failures
5. Document CloudTrail event priorities
6. Write first runbook (iam-accessdenied.md)
7. Create validation checklist

**Session-by-session build:**
- Each session adds one artifact from "Artifacts That Prove Project 2 Is Working"
- Build incrementally, validate continuously
- Document evidence as you go
- Don't move forward until current piece validated

---

## Notes

**What makes Project 2 different from Project 1:**

| Aspect | Project 1 | Project 2 |
|--------|-----------|-----------|
| **Focus** | Deployment success | Operational visibility |
| **Evidence** | "Does it work?" | "Can I detect and investigate when it breaks?" |
| **IAM** | Sufficient permissions | Least privilege + investigation |
| **CloudTrail** | Not emphasized | Core investigation capability |
| **Logs** | Deployment validation | Systematic investigation |
| **Runbooks** | Deployment steps | Support-style troubleshooting |
| **Monitoring** | Basic health checks | Comprehensive observability |

**Key philosophy:**
- Project 1 proves deployment
- Project 2 proves operation and support
- Together they demonstrate end-to-end capability

---
```
