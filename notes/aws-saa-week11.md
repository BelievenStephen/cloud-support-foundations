# AWS SAA — Week 11

### Project 2 CloudTrail readiness check

- Checked CloudTrail in `us-west-1`.
- CloudTrail enabled: `no confirmed trail found`
- Trail found: `none found`
- Scope: `not applicable`
- Logging status: `not applicable`
- Log delivery location: `not applicable`

What this means for Project 2:
- CloudTrail does not yet give an audit source for support-style investigations.
- Right now, I would not be able to use CloudTrail in this account to explain who changed ECS, ALB, target group, IAM, or monitoring settings and when those changes happened.
- For Project 2, this identifies a real gap to address later if I want audit and investigation coverage.

What still needs to be decided later:
- whether I should create a trail for Project 2
- whether that trail should be multi-Region or scoped differently
- whether CloudWatch Logs integration is needed in addition to S3 delivery
- what event coverage is most important for Project 2 investigations

---

## Project 2 — Log Observation Notes
## Apr 17, 2026

**Log group inspected:** `/ecs/aws-hosted-app` (`us-west-1`)
**Log stream inspected:** `ecs/aws-hosted-app/<task-id>`

---

### What Healthy Log Activity Looks Like

| Signal | Expected Pattern |
|---|---|
| Health check requests | Repeated `GET /health HTTP/1.1` → `200` |
| Root path requests | `GET / HTTP/1.1` → `200` |
| Startup behavior | No tracebacks, failure messages, or repeated error patterns |

The current stream matches this pattern. The task appears to be running normally and serving health checks successfully.

---

### What Error Activity Would Matter First

- Startup failures or tracebacks on launch
- Repeated `/health` failures or non-`200` responses
- Any `5xx` responses in the stream
- Exception or timeout messages
- A sudden shift away from the current pattern of stable `200` responses

---

### How Logs Support Runbooks and Investigation

- Logs confirm whether the app is healthy at the moment of inspection
- Logs separate healthy request behavior from startup or runtime failure
- The current healthy stream establishes a baseline to compare against if the service degrades
- Logs will be one of the first investigation sources in future Project 2 runbooks

---

### What This Means for Centralized Log Planning

CloudWatch Logs already provides a useful central source for inspecting application behavior in `us-west-1`. No additional log aggregation is needed to begin runbook and investigation work — the current log group is sufficient as a starting reference.

---

## Project 2 — Metric Readiness Notes
*April 18, 2026*

Confirmed ApplicationELB metrics in `us-west-1`.

---

### Metric Confirmation Results

| Metric | Namespace | Status | Observed Behavior |
|---|---|---|---|
| `RequestCount` | Per AppELB Metrics | ✅ Confirmed | Recent datapoints visible — normal traffic reaching the ALB |
| `TargetResponseTime` | Per AppELB Metrics | ✅ Confirmed | Recent datapoints with minor gaps and small spikes — values remain low, no distress signal |
| `HealthyHostCount` | Per AppELB, per TG Metrics | ✅ Confirmed | Stable value of `1` — strong healthy-baseline signal for the current single target |
| `HTTPCode_ELB_5XX_Count` | Per AppELB Metrics | ⚠️ Not surfaced | Not visible from current search view — recheck when building dashboard or alarm set |

---

### What Each Metric Helps Catch

| Metric | What It Catches |
|---|---|
| `RequestCount` | Confirms whether traffic is reaching the ALB |
| `TargetResponseTime` | Detects backend slowdown or degradation before full failure |
| `HealthyHostCount` | Shows whether the target group still has healthy registered targets |
| `HTTPCode_ELB_5XX_Count` | Catches ALB-generated 5xx failures — still an important failure-focused metric |

---

### What This Means for Dashboard and Alarm Setup

- `RequestCount`, `TargetResponseTime`, and `HealthyHostCount` are confirmed candidates for the first Project 2 dashboard
- `HealthyHostCount` at `1` is the clearest healthy-vs-failure signal available for the current single-target workload
- `HTTPCode_ELB_5XX_Count` should be rechecked when building the dashboard or alarm set — it was not surfaced in this session but remains a required metric for the first alarm candidates defined in `project2-monitoring-starter.md`
