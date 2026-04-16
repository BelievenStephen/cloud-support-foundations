# Project 2 — Monitoring Starter

**Purpose:** Define the first monitoring targets for Project 2 so the first monitoring session is clear and actionable before making additional AWS changes.

---

## CloudTrail Status

No trail currently exists in this account. Creating a trail will be a Project 2 task.

| Item | Status |
|---|---|
| CloudTrail enabled? | No — no trail exists |
| Planned trail scope | To be decided (single-region vs. multi-region) |
| Log delivery destination | To be configured |

---

## First Dashboard Metrics

| Metric | What It Catches |
|---|---|
| `RequestCount` | Confirms whether traffic is reaching the ALB at all |
| `TargetResponseTime` | Detects backend slowdown or abnormal behavior before full failure |
| `HTTPCode_ELB_5XX_Count` | Catches load-balancer-level errors — fast signal that something is wrong at the traffic layer |
| `HealthyHostCount` | Shows whether the target group still has healthy registered targets |

These metrics are the best starting point because they are simple, practical, and directly tied to the observed Project 1 workload.

---

## First Alarm Candidates

| Alarm | Based On | Priority |
|---|---|---|
| ALB 5xx error count | `HTTPCode_ELB_5XX_Count` | High |
| Healthy host count drops below threshold | `HealthyHostCount` | High |
| CPU or memory utilization | ECS task-level metrics | Optional — add only if it adds clear support value |

These alarms are the best first candidates because they are easy to explain and directly support troubleshooting conversations.

---

## First Logs to Centralize

| Log Source | Why It Matters First |
|---|---|
| CloudWatch Logs from the observed workload | Fastest way to understand why a service is unhealthy |
| ECS task / application logs from the running service | Confirms whether the app started and is running correctly |
| Request and health-check logs (especially `/health`) | Establishes a healthy request baseline |
| Startup and runtime error logs | Catches failures at launch before they appear in metrics |

---

## Healthy Baseline Definition

The observed workload is considered healthy when all of the following are true:

| Signal | Expected State |
|---|---|
| ECS service desired / running / pending | `1 / 1 / 0` |
| Target group health | `healthy` |
| `/health` response | `200` |
| `/` response | `200` |
| ALB 5xx activity during normal checks | None |
| CloudWatch Logs | Repeated `/health 200` responses visible |
| Response times | Stable under normal load |

This baseline is the reference point for all future alarm thresholds and runbook scenarios.

---

## Notes

Project 2 starts with the simplest useful monitoring view possible. The goal is not to track everything at once — it is to establish the signals that best support troubleshooting, validation, and runbook writing. All alarm thresholds and log queries will be defined once a healthy baseline is confirmed.
