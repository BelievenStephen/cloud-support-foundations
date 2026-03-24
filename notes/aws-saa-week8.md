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
