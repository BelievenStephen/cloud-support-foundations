# Project 2 — First Session Checklist

**Purpose:** Make the first real Project 2 work session clear, focused, and low-risk. This is a planning and orientation session, not a build session.

---

## Checklist

### 1. Confirm AWS Account and Region

| Check | Expected State |
|---|---|
| Active AWS account | Correct account selected |
| Active region | `us-west-1` |
| Budget alerts | Enabled |
| Unexpected running resources | None |

---

### 2. Decide What Project 2 Will Observe

- Decide whether Project 2 will observe the **existing Project 1 deployment** (ECS Fargate + ALB) or use a **small separate test target**
- Prefer the simplest option that delivers clear monitoring, CloudTrail, and IAM investigation value
- Document the decision before moving forward

---

### 3. Inspect CloudTrail Status

| Item | Notes |
|---|---|
| CloudTrail enabled? | |
| Trail name | |
| Single-region or multi-region? | |
| Logging active? | |
| Log delivery destination | |

---

### 4. Decide First Dashboard Metrics

Start with a small, practical set tied to the observed workload:

| Metric | Why It Matters |
|---|---|
| `RequestCount` | Confirms traffic is reaching the ALB |
| `TargetResponseTime` | Baseline latency for the observed target |
| `HTTPCode_ELB_5XX_Count` | Catches backend errors at the load balancer |
| `HealthyHostCount` | Confirms registered targets are passing health checks |

---

### 5. Decide First Alarm Candidates

Start with alarms that are easy to explain and directly relevant to a support context:

| Alarm | Priority |
|---|---|
| ALB 5xx error count | High |
| Healthy host count drops below threshold | High |
| CPU or memory utilization (if it adds clear value) | Optional |

---

### 6. Decide First Logs to Centralize

- CloudWatch Logs for the observed workload
- Request and health-check related logs if available
- Any logs that help establish a healthy baseline versus a failure state

---

### 7. Decide First Runbook to Write

**Target:** `permission-denied-iam.md`

**Reason:**
- Makes Project 2 clearly distinct from Project 1
- Directly tied to the IAM hardening and CloudTrail investigation focus
- Produces a reusable, explainable operational artifact from day one

---

### 8. Define "Done" for Session 1

Session 1 is complete when:

- [ ] First observed target is chosen and documented
- [ ] CloudTrail status is inspected and recorded
- [ ] First dashboard metrics are chosen
- [ ] First alarm candidates are chosen
- [ ] First logs to centralize are chosen
- [ ] First runbook target is confirmed
- [ ] Session notes are clear enough that the next session can start without guesswork

---

## Notes

The goal of Session 1 is not to build everything. The goal is to leave with every key decision made, documented, and ready to act on in Session 2. Focus on monitoring, IAM, CloudTrail, logs, and runbooks.
