# Project 2 Build Order

## Purpose

This document defines the build order for Project 2, focusing on monitoring, IAM hardening, CloudTrail integration, centralized logging, and support-style documentation. The approach emphasizes controlled progression and operational clarity over infrastructure complexity.

---

## Build Order

### 1) Confirm Current AWS Account Hygiene

**Pre-build checklist:**
- Verify budget alerts still active and configured correctly
- Confirm no forgotten cost-causing resources (NAT Gateways, EIPs, running instances)
- Verify working in correct region (`us-west-1`)
- Verify working in correct AWS account

**Why this comes first:**
- Prevents cost surprises during new resource creation
- Establishes clean baseline before adding complexity
- Confirms foundational guardrails in place

---

### 2) Choose What Project 2 Will Observe

**Decision point:**
- Observe existing Project 1 deployment (ECS service behind ALB), OR
- Create small test target for monitoring

**Recommendation:** Start with Project 1 deployment
- Already has metrics flowing (ALB, target group, ECS)
- Already has established baseline behavior
- Simplifies monitoring notes (known good state vs unknown)

**Why this comes second:**
- Monitoring strategy depends on what's being monitored
- Simpler target = easier to explain investigation notes
- Establishes scope boundary for documentation

---

### 3) Enable or Confirm CloudTrail

**Configuration steps:**
- Verify CloudTrail enabled in account
- Confirm trail logging to S3
- Note CloudTrail log group in CloudWatch Logs (if enabled)

**Events to focus on:**
- IAM policy changes (`CreatePolicy`, `AttachUserPolicy`, `PutUserPolicy`)
- Security Group modifications (`AuthorizeSecurityGroupIngress`, `RevokeSecurityGroupIngress`)
- ECS task/service changes (`CreateService`, `UpdateService`, `StopTask`)
- Resource creation/deletion for audit trail

**What CloudTrail proves during troubleshooting:**
- Who made a change and when
- What configuration existed before change
- Whether change correlates with incident timeline
- API call patterns during incident window

**Documentation output:**
- Note which events are most useful for investigations
- Record CloudTrail S3 bucket location
- Document how to query CloudTrail for common scenarios

---

### 4) Define IAM Least-Privilege Notes

**Identify required permissions:**
- Which roles/users actually needed for operations
- Which services need cross-service access (e.g., ECS task role, execution role)
- What should NOT be over-permissioned

**Documentation to create:**
- List of actual required permissions vs overly broad permissions
- How to review IAM during access issue investigation
- Common AccessDenied scenarios and triage workflow
- Policy evaluation order refresher (explicit deny wins)

**Investigation workflow to document:**
1. Confirm caller identity (`aws sts get-caller-identity`)
2. Check identity-based policies
3. Check resource-based policies
4. Check permissions boundaries
5. Check SCPs (if in Organization)
6. Test in Policy Simulator

**Why this comes before monitoring:**
- IAM issues often discovered during resource setup
- Establishes principle of least privilege early
- Prevents overly permissive policies becoming baseline

---

### 5) Plan CloudWatch Dashboard and Alarms

**Start small - first dashboard widgets:**
- Reuse Project 1 patterns:
  - `RequestCount` (traffic reaching ALB)
  - `TargetResponseTime` (backend latency)
  - `HTTPCode_ELB_5XX_Count` (ALB errors)
  - `HealthyHostCount` (target availability)

**First alarms to create:**
- Critical path only:
  - `HealthyHostCount < 1` (no capacity)
  - `HTTPCode_ELB_5XX_Count >= 1` (infrastructure failure)

**Support-focused approach:**
- Metrics that answer "is service up?"
- Metrics that distinguish infrastructure vs application failure
- Keep initial dashboard simple and actionable

**Documentation output:**
- What each metric proves during investigation
- Alarm response playbook (what to check when alarm fires)
- Dashboard triage workflow

---

### 6) Decide Which Logs Will Be Centralized

**Log sources to consider:**
- ECS container logs (already in CloudWatch Logs `/ecs/aws-hosted-app`)
- ALB access logs (optional, stored in S3)
- CloudTrail logs (management events)
- VPC Flow Logs (optional, network traffic)

**Start with:**
- ECS container logs (already configured)
- CloudTrail logs (if sending to CloudWatch Logs)

**Define healthy log activity:**
- What does normal ECS task startup look like in logs
- What does healthy ALB health check pattern look like
- What CloudTrail events indicate normal operations

**Investigation signals from logs:**
- ECS task crash/restart events
- Application errors in container logs
- Failed health checks in logs
- IAM permission changes in CloudTrail

**Documentation output:**
- Which logs exist and where to find them
- How to filter logs for common investigation scenarios
- What log patterns indicate specific failure modes

---

### 7) Choose Runbooks to Write First

**Most likely support scenarios:**
- Service down (ECS task unhealthy/stopped)
- ALB returning 502/503 (no healthy targets)
- IAM AccessDenied (permission issue)
- Recent change caused outage (CloudTrail investigation)

**Runbook priorities:**
1. Service health check workflow (is it up? why not?)
2. ALB 5xx triage (ELB vs Target 5xx split)
3. IAM permission debugging (systematic policy check)
4. CloudTrail event investigation (who changed what when)

**Runbook format:**
- Short, practical, support-focused
- Tied to real signals (alarms, logs, CloudTrail events)
- Clear verification steps ("how do I know it's fixed?")
- Reuse Project 1 validation patterns

**Documentation output:**
- Runbook files in `cloud-support-foundations/runbooks/`
- Each runbook references specific metrics/logs/CloudTrail events
- Clear symptom → investigation → resolution flow

---

## What Project 2 Reuses from Project 1

**Monitoring patterns:**
- CloudWatch dashboard structure
- ALB and target health monitoring approach
- Alarm configuration patterns

**Validation approach:**
- Testing `/health` and `/` endpoints
- Reviewing CloudWatch Logs for expected patterns
- Verifying service state (desired vs running tasks)

**Documentation style:**
- Support-focused runbook format
- Validation checklists
- "What This Unblocks" sections
- Structured troubleshooting workflows

**Lessons learned:**
- ARM64 task definition architecture requirement
- Importance of health check configuration
- Security group rule verification
- End-to-end traffic path validation

---

## What Project 2 Adds Beyond Project 1

**New focus areas:**

**1) IAM least-privilege documentation:**
- Explicit permission documentation
- AccessDenied investigation workflow
- Policy Simulator usage patterns

**2) CloudTrail as investigation source:**
- Event-based change tracking
- Timeline correlation during incidents
- "Who changed what when" capability

**3) Auditability and access review:**
- How to review IAM permissions
- How to audit recent changes
- How to verify least privilege

**4) Centralized log strategy:**
- Defined log sources and locations
- Log-based investigation patterns
- Healthy vs unhealthy log signatures

**5) Support-style investigation notes:**
- Beyond deployment validation
- Focus on operational troubleshooting
- Real-world incident response patterns

**6) Monitoring-and-security operations story:**
- Not just "does it work"
- Also "can I detect when it breaks"
- And "can I prove what changed"

---

## Project 2 Goals

**Primary goal:**
- Create smallest useful setup that proves ability to monitor, investigate, and document AWS environment in Cloud Support style

**Not the goal:**
- Create complex infrastructure
- Deploy multiple services
- Build production-grade architecture

**Success criteria:**
- Can detect service failure via alarms
- Can investigate failure via logs and CloudTrail
- Can document investigation workflow in runbooks
- Can verify IAM permissions correctly configured
- Can demonstrate operational evidence gathering

---

## Notes

**Philosophy:**
- Start small, prove capability, document clearly
- Focus on operational evidence over infrastructure complexity
- Every component should support investigation workflow
- Documentation should enable someone else to troubleshoot

**Measurement of success:**
- Can I detect an issue? (Alarms)
- Can I investigate the issue? (Logs, CloudTrail, metrics)
- Can I document the investigation? (Runbooks)
- Can I verify the fix? (Metrics, logs, endpoint tests)

---
