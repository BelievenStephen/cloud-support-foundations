# AWS SAA — Week 8

## Mar 24, 2026

## Project 1 — ECS Pre-Flight: Log Group and Execution Role

### Summary

Verified the two IAM and logging dependencies required before registering the ECS task definition — the CloudWatch log group and the task execution role.

---

### CloudWatch Log Group

| Detail | Value |
|--------|-------|
| Log group name | `/ecs/aws-hosted-app` |
| Retention | 7 days |
| Status | Created and confirmed ✅ |

---

### Task Execution Role

| Detail | Value |
|--------|-------|
| Role name | `ecsTaskExecutionRole` |
| Already existed | No |
| Status | Needs to be created before task definition registration ⚠️ |

The standard execution role (`ecsTaskExecutionRole`) was not found. It must be created and the AWS-managed `AmazonECSTaskExecutionRolePolicy` policy attached before the task definition can be registered.

---

### What This Unblocks

- The CloudWatch log group is ready to receive container logs via the `awslogs` driver once the task is running.
- Identifying the missing execution role now isolates it as the next dependency, keeping the task definition step clean and unblocking the rest of the ECS setup.

### Next Step

Create `ecsTaskExecutionRole` and attach `AmazonECSTaskExecutionRolePolicy`.

---

## Mar 25, 2026

## Project 1 — Execution Role and Task Definition

### Summary

Created the missing task execution role and registered the first revision of the ECS task definition. The application image, IAM role, networking mode, and logging are now fully wired together.

---

### Task Execution Role

| Detail | Value |
|--------|-------|
| Role name | `ecsTaskExecutionRole` |
| Policy attached | `AmazonECSTaskExecutionRolePolicy` |
| Status | Created and confirmed ✅ |

---

### Task Definition

| Detail | Value |
|--------|-------|
| Family | `aws-hosted-app` |
| Revision | `1` |
| Container image | `886219357247.dkr.ecr.us-west-1.amazonaws.com/aws-hosted-app:latest` |
| Container port | `8080` |
| Network mode | `awsvpc` |
| Launch type compatibility | Fargate |
| CPU | `256` |
| Memory | `512` MiB |
| Log group | `/ecs/aws-hosted-app` |
| Status | Registered ✅ |

---

### What This Unblocks

- The application image, execution role, and CloudWatch Logs configuration are now wired into a valid task definition.
- The next phase can proceed: creating the target group, ECS cluster, and ECS service.

### Next Step

Create the target group, ECS cluster, and ECS service.

---

## Mar 26, 2026

## ECS Service Deployment — Signals and Triage

### Key Concepts

| Concept | What it means |
|---------|---------------|
| **Service events** | First place to check when a deployment fails or tasks do not stay healthy — surface task launch failures, target registration problems, and replacement activity |
| **Stopped task reasons** | Separates a launch problem from an app problem — shows whether the failure was image pull, startup, health check, or another runtime issue |
| **Health check grace period** | Time ECS ignores unhealthy ELB and container health checks after a task starts — default is `0`, so ECS reacts to unhealthy checks immediately if not set |
| **CloudWatch Logs** | First place to check if the task starts but the app does not behave correctly — confirms whether the container launched and what it printed before failing |
| **Target group health** | Matters once the service is tied to a load balancer — if the task is running but the target is unhealthy, check the health check path, port, and app response before investigating the ALB |

---

### Triage: Launch Failure vs App Failure

**Task never reaches a stable running state → think launch issue first**
- Check service events
- Check stopped task reasons
- Check IAM permissions, image access, and networking

**Task runs but service still fails → think app issue first**
- Check CloudWatch Logs
- Check health check behavior
- Check target group health

---

### Project 1 Notes

- If the task starts but `/health` fails, check CloudWatch Logs first.
- Once the ECS service is attached to the target group, target group health becomes one of the first checks if tasks run but traffic still fails.

### Next Step

Create the target group, ECS cluster, and ECS service.

---
