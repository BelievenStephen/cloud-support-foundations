# Project 2 — First Runbook Plan

**Target runbook:** `permission-denied-iam.md`

---

## Why This Is the Right First Choice

A permission-denied scenario is a common cloud support problem and a natural fit for Project 2's IAM hardening and investigation focus. It also makes Project 2 clearly distinct from Project 1, which centered on deployment, validation, logs, and load balancer health.

This runbook connects IAM thinking, CloudTrail thinking, log inspection, and baseline comparison in one place — which is exactly the kind of multi-signal investigation that a support-style runbook should demonstrate.

---

## First Metrics to Check

| Metric | Why It Gets Checked First |
|---|---|
| `HealthyHostCount` | Confirms whether the service is still healthy overall |
| `RequestCount` | Confirms whether traffic is still reaching the load balancer |
| `TargetResponseTime` | Shows whether the backend is still responding normally |

These metrics will not directly prove an IAM issue, but they help rule out a general service outage before narrowing the investigation toward access.

---

## First Logs to Check

**Log group:** `/ecs/aws-hosted-app` (`us-west-1`)

| Log Source | What to Look For |
|---|---|
| Most recent ECS task log stream | Startup failures, tracebacks, or unexpected runtime errors |
| Health check and request logs | Any departure from the normal pattern of `/health` and `/` returning `200` |

Logs help confirm whether the app is still running normally and whether the issue looks like an application failure rather than an access or permission problem.

---

## Where CloudTrail Fits

No trail is currently configured. Creating a trail is a planned Project 2 task. Once enabled, CloudTrail would be one of the first investigation sources for a permission-denied scenario.

**CloudTrail would help answer:**

- Who changed an IAM policy, role, or permission — and when
- Whether an ECS, ALB, or monitoring-related change happened near the same time as the reported issue
- Whether the problem timeline lines up with a recent access-related event

CloudTrail functions as the audit and timeline source in this investigation path — it provides the *why* and *when* that metrics and logs alone cannot.

---

## Healthy Baseline Reference

| Signal | Expected State |
|---|---|
| ECS service desired / running / pending | `1 / 1 / 0` |
| Target group health | `healthy` |
| `/health` response | `200` |
| `/` response | `200` |
| `RequestCount` | Recent traffic visible |
| `TargetResponseTime` | Low recent values, no distress signal |
| `HealthyHostCount` | `1` |
| CloudWatch Logs | Repeated `/health 200` responses, no error pattern |

Any deviation from this baseline during a permission-denied investigation is a signal worth documenting.

---

## What Makes This Runbook Useful

This runbook earns its place if it produces a clear, repeatable path that:

- Separates IAM and access issues from application or infrastructure failures
- Identifies which AWS signals to check first and in what order
- Explains where CloudTrail fits once a trail is active
- Uses the confirmed healthy baseline as a comparison point
- Gives a support-style investigation path that can be followed without guesswork

---

## Notes

This runbook is intentionally scoped to investigation, not remediation. The goal is to demonstrate reasoned multi-signal thinking across metrics, logs, IAM review, and CloudTrail — the kind of structured approach that distinguishes a support-focused runbook from a deployment or validation checklist.
