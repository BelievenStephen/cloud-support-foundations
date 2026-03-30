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
