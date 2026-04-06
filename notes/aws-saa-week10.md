# AWS SAA — Week 10

## Apr 6, 2026

## Project 1 — CloudWatch Dashboard

### Summary

Created a CloudWatch dashboard to provide single-pane visibility into ALB and target group health. The dashboard consolidates key metrics for quick incident detection and triage.

---

### Dashboard Configuration

| Detail | Value |
|--------|-------|
| Dashboard name | `aws-hosted-app-dashboard` |
| Region | `us-west-1` |
| Resource | `aws-hosted-app-alb` / `aws-hosted-app-tg` |

---

### Widgets Added

**Widget 1: RequestCount**

| Metric | Purpose |
|--------|---------|
| `RequestCount` | Confirms whether traffic is reaching the ALB |

**What this catches first:**
- Complete traffic loss (count drops to zero)
- Traffic spikes or anomalies
- Baseline traffic patterns for comparison

---

**Widget 2: TargetResponseTime**

| Metric | Purpose |
|--------|---------|
| `TargetResponseTime` | Shows whether backend response time is rising |

**What this catches first:**
- Application performance degradation
- Database or dependency slowness
- Resource saturation on tasks (CPU/memory)

---

**Widget 3: HTTPCode_ELB_5XX_Count**

| Metric | Purpose |
|--------|---------|
| `HTTPCode_ELB_5XX_Count` | Shows whether ALB is returning 5xx errors |

**What this catches first:**
- ALB connectivity to targets broken
- No healthy targets available
- Infrastructure-layer failures

---

**Widget 4: HealthyHostCount**

| Metric | Purpose |
|--------|---------|
| `HealthyHostCount` | Shows whether target group still has healthy targets |

**What this catches first:**
- Health check failures
- Tasks terminated or stopped
- Service scaled to zero
- Deployment issues preventing new tasks from becoming healthy

---

### Dashboard Triage Flow

**Using dashboard to diagnose issues:**

| Symptom Pattern | Likely Issue | Investigation Path |
|----------------|--------------|-------------------|
| **RequestCount > 0, HealthyHostCount = 0, ELB 5xx rising** | No healthy targets | Check ECS service, health check config |
| **RequestCount > 0, HealthyHostCount > 0, TargetResponseTime rising** | Application slowdown | Check task CPU/memory, database performance |
| **RequestCount drops to zero** | Traffic loss | Check DNS, ALB configuration, upstream routing |
| **ELB 5xx rising, HealthyHostCount stable** | ALB-to-target connectivity | Check security groups, NACLs, task networking |

---

### What This Enables

**Operational benefits:**
- **Single-pane view:** All critical metrics in one dashboard
- **Quick triage:** Pattern recognition speeds up root cause identification
- **Baseline establishment:** Visual history helps identify anomalies
- **Incident response:** Share dashboard URL with team during incidents

---

### Dashboard Access

**Console path:**
```
CloudWatch → Dashboards → aws-hosted-app-dashboard
```

**Direct link pattern:**
```
https://console.aws.amazon.com/cloudwatch/home?region=us-west-1#dashboards:name=aws-hosted-app-dashboard
```

---

### Next Steps (Future Dashboard Enhancements)

**Potential additional widgets:**
- `HTTPCode_Target_5XX_Count` - Application errors
- `ActiveConnectionCount` - Current connections to ALB
- `ProcessedBytes` - Data transfer volume
- ECS service metrics - Running task count, CPU/memory utilization
- Custom application metrics - Business-level KPIs

---
