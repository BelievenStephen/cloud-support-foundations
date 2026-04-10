# Project 2 — Scope and Requirements

## Goal

Project 2 shifts focus from deployment to operations and support. Where Project 1 demonstrated the ability to deploy and validate a containerized app on AWS, Project 2 demonstrates the ability to monitor a running system, document security and audit practices, and investigate issues using CloudWatch, IAM, CloudTrail, and runbooks.

---

## What This Project Is Meant to Prove

| Capability | Description |
|---|---|
| Monitoring | Create useful visibility into a live or test AWS workload |
| IAM | Document and apply a least-privilege approach in a support context |
| Audit | Enable and explain CloudTrail for investigation and audit review |
| Runbooks | Write structured troubleshooting guides for common support scenarios |

---

## Requirements

### Monitoring

- CloudWatch dashboard covering key metrics
- Alarms for:
  - ALB 5xx errors
  - Target group health
  - CPU or memory utilization (if available)
- Centralized logs in CloudWatch Logs
- Short notes on what each metric or alarm is designed to catch first

### IAM Hardening

- Documented least-privilege approach for the workload
- Notes on which IAM roles and permissions are required and why
- Examples of what should not be over-permissioned
- Short explanation of how to review IAM access when a permission issue occurs

### CloudTrail

- CloudTrail enabled and documented
- Notes on what CloudTrail captures and its scope
- Examples of event types to check during a support investigation
- Short explanation of how CloudTrail supports audit review and support-style troubleshooting

### Runbooks

Support-style runbooks to include:

- Site down
- 5xx spike
- Permission denied / IAM access issue
- Unhealthy target or service
- CloudTrail investigation starting point

---

## Completion Criteria

Project 2 is considered complete when all of the following are in place:

- [ ] CloudWatch dashboard deployed and documented
- [ ] Alarms configured and verified
- [ ] Centralized logs in CloudWatch Logs
- [ ] IAM least-privilege approach documented
- [ ] CloudTrail enabled and documented
- [ ] Runbooks written for all scenarios listed above
- [ ] README explains what the project does, how to validate it, and what was learned

---

## Relationship to Project 1

Project 2 builds on the deployment foundation from Project 1 but prioritizes monitoring, auditability, and support investigation over infrastructure standup. The core app and ECS/ALB architecture can be reused or referenced; the new work is the operational layer on top of it.
