# AWS SAA — Week 9

## Mar 30, 2026

## Project 1 — ECS Service Networking Prep

### Summary

Locked the subnet IDs and security group rules that will be used during ECS service creation and ALB attachment.

---

### Subnets

| Detail | Value |
|--------|-------|
| Subnet 1 | `subnet-0b7f3fdc246069fbe` |
| Subnet 2 | `subnet-0ed1fa4a40bc62a26` |
| Region | `us-west-1` |

---

### Security Groups

| Security Group | Inbound Rule | Source |
|----------------|-------------|--------|
| `aws-hosted-app-alb-sg` | HTTP port `80` | `0.0.0.0/0` |
| `aws-hosted-app-task-sg` | TCP port `8080` | `aws-hosted-app-alb-sg` only |

The task security group restricts inbound traffic to port `8080` from the ALB security group only — no direct public access to the task.

---

### What This Unblocks

- Subnets and security groups are confirmed and ready to supply at ECS service creation time.
- The security group rules enforce the correct traffic path: internet → ALB on port `80` → task on port `8080`.

### Next Step

Create the ALB with `aws-hosted-app-alb-sg`, then create the ECS service using these subnets and `aws-hosted-app-task-sg`.

---

## Mar 31, 2026

## Project 1 — ALB Creation

### Summary

Created the Application Load Balancer and configured the HTTP listener to forward traffic to the existing target group. The frontend load balancer path is now in place ahead of ECS service creation.

---

### ALB Configuration

| Detail | Value |
|--------|-------|
| Name | `aws-hosted-app-alb` |
| Listener port | `80` (HTTP) |
| Security group | `aws-hosted-app-alb-sg` |
| Subnets | `subnet-0b7f3fdc246069fbe`, `subnet-0ed1fa4a40bc62a26` |
| Target group | `aws-hosted-app-tg` |
| DNS name | `aws-hosted-app-alb-152549554.us-west-1.elb.amazonaws.com` |
| Status | Created ✅ |

---

### What This Unblocks

- The ALB is now in place with a listener on port `80` forwarding to `aws-hosted-app-tg`.
- All prerequisites from the service creation checklist are now satisfied — the ECS service can be created in the next session.

### Next Step

Create the ECS service inside `aws-hosted-app-cluster`, attach it to `aws-hosted-app-tg`, and select the planned subnets and `aws-hosted-app-task-sg`.

---

## Apr 1, 2026

## Project 1 — ECS Service Creation

### Summary

Created the ECS service and confirmed a task launched, registered into the target group, and reached a healthy state. Project 1 is now running as a live ECS Fargate service behind an ALB.

---

### Service Configuration

| Detail | Value |
|--------|-------|
| Service name | `aws-hosted-app-service` |
| Cluster | `aws-hosted-app-cluster` |
| Task definition | `aws-hosted-app:2` |
| Desired count | `1` |
| Launch type | Fargate |
| Subnets | `subnet-0b7f3fdc246069fbe`, `subnet-0ed1fa4a40bc62a26` |
| Task security group | `aws-hosted-app-task-sg` |
| Public IP | Enabled |

---

### Deployment Verification

| Check | Result |
|-------|--------|
| Task launched | ✅ |
| Target registered in `aws-hosted-app-tg` | ✅ |
| Target health | Healthy ✅ |

---

### What This Unblocks

- The service is running with a healthy registered target, meaning the full traffic path from ALB → target group → ECS task is ready to validate.

### Next Step

Validate the ALB endpoint by testing `GET /health` and `GET /` against `aws-hosted-app-alb-152549554.us-west-1.elb.amazonaws.com`.
