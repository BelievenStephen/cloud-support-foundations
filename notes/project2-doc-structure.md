# Project 2 Documentation Structure

## Purpose

This document defines the documentation structure for Project 2 to ensure consistency, ease of navigation, and clear separation of concerns. The goal is to avoid confusion about where content belongs and to make the project accessible to future readers.

## Directory Structure Overview

```
project2/
├── README.md
├── docs/
│   ├── dashboard-setup.md
│   ├── alarm-configuration.md
│   ├── iam-least-privilege.md
│   ├── cloudtrail-setup.md
│   └── validation-results.md
├── runbooks/
│   ├── service-down.md
│   ├── alb-5xx-spike.md
│   ├── iam-accessdenied.md
│   ├── unhealthy-targets.md
│   └── cloudtrail-investigation.md
└── notes/
    ├── project-scope.md
    ├── build-order.md
    ├── doc-structure.md
    └── session-checklist.md
```

## README.md

**Location:** `project2/README.md`

**Purpose:** High-level project overview and entry point

**Should explain:**
- What Project 2 does and why it exists
- What the project is supposed to prove
- High-level architecture or operating model
- AWS services used and their relationships
- Cost considerations and budget alerts
- How to validate the project works
- Links to important docs and runbooks
- Key lessons learned

**Structure:**
```markdown
# Project 2: [Title]

## Overview
[What this project does]

## Goals
[What this proves]

## Architecture
[High-level diagram or description]

## Requirements
- AWS Account with budget alerts
- Region: us-west-1
- Services: CloudWatch, CloudTrail, IAM, [others]

## Cost Considerations
[Expected monthly costs, what to watch]

## Validation
[How to verify project is working]

## Documentation
- Setup: [links to docs/]
- Troubleshooting: [links to runbooks/]
- Planning: [links to notes/]

## Lessons Learned
[Key takeaways]
```

---

## docs/

**Location:** `project2/docs/`

**Purpose:** Focused project documentation, setup guides, and configuration evidence

**Content types:**
- Dashboard setup and widget descriptions
- Alarm configuration and thresholds
- IAM permission documentation
- CloudTrail configuration and event focus
- Validation results and evidence
- Teardown procedures (if needed)

**Example files:**

### dashboard-setup.md
- Dashboard name and purpose
- Widget descriptions (what each metric proves)
- How to interpret dashboard during incident
- Triage flow using dashboard

### alarm-configuration.md
- Each alarm's configuration (metric, threshold, period)
- What each alarm detects
- Alarm response workflow (what to check when it fires)
- Verification that alarms work

### iam-least-privilege.md
- Required roles and permissions
- What should NOT be over-permissioned
- How to review IAM during investigations
- Policy evaluation order reference
- Common AccessDenied scenarios

### cloudtrail-setup.md
- CloudTrail configuration (trail name, S3 bucket, log group)
- Which events are most useful for investigations
- How to query CloudTrail for common scenarios
- Example investigation using CloudTrail

### validation-results.md
- End-to-end validation steps performed
- Test results (endpoints, logs, metrics)
- Proof of correct configuration
- Baseline behavior documentation

---

## runbooks/

**Location:** `project2/runbooks/`

**Purpose:** Support-style troubleshooting guides for common scenarios

**Content types:**
- Incident investigation workflows
- Systematic troubleshooting procedures
- Clear symptom → investigation → resolution flows

**Required sections for each runbook:**
1. **Symptoms** - What users report or what alarms fire
2. **Likely causes** - Common root causes for this symptom
3. **Where to look first** - Prioritized investigation order
4. **Exact checks** - Specific commands/console paths
5. **Fix steps** - Resolution procedures
6. **Verification** - How to confirm issue resolved

**Example runbooks:**

### service-down.md
- Symptom: Service unreachable or returning errors
- Checks: Service state, port listening, logs, health checks
- Fix: Restart service, fix configuration, resolve dependencies

### alb-5xx-spike.md
- Symptom: ALB returning 5xx errors
- Checks: Split ELB vs Target 5xx, HealthyHostCount, logs
- Fix: Resolve unhealthy targets, fix connectivity, app errors

### iam-accessdenied.md
- Symptom: AccessDenied or permission denied errors
- Checks: Caller identity, policies, boundaries, SCPs
- Fix: Add required permissions, remove conflicting denies

### unhealthy-targets.md
- Symptom: Target group has no healthy targets
- Checks: Health check config, SG rules, app listening
- Fix: Correct health check, fix networking, start service

### cloudtrail-investigation.md
- Symptom: Need to determine "who changed what when"
- Checks: CloudTrail event history, specific API calls
- How to: Query CloudTrail, interpret events, build timeline

---

## notes/

**Location:** `project2/notes/`

**Purpose:** Planning documents, working notes, and early thinking

**Content types:**
- Project scope definition
- Build order and sequencing
- Observation plans
- Session checklists
- Rough ideas before formalization
- Decision records

**Example files:**

### project-scope.md
- What's in scope for Project 2
- What's explicitly out of scope
- Success criteria
- Boundaries and constraints

### build-order.md
- Numbered sequence of build steps
- Why each step comes in that order
- What each step unblocks
- Dependencies between steps

### doc-structure.md
- This document
- Defines where content belongs
- Documentation standards

### session-checklist.md
- Pre-session setup verification
- During-session working notes
- Post-session documentation tasks
- Next session preparation

---

## Content Placement Guide

**Quick reference for where to put content:**

| Content Type | Location | Example |
|-------------|----------|---------|
| **Dashboard setup** | `docs/` | `dashboard-setup.md` |
| **Alarm configuration** | `docs/` | `alarm-configuration.md` |
| **IAM permissions** | `docs/` | `iam-least-privilege.md` |
| **CloudTrail setup** | `docs/` | `cloudtrail-setup.md` |
| **Validation results** | `docs/` | `validation-results.md` |
| **Teardown procedures** | `docs/` | `teardown.md` |
| **Troubleshooting guides** | `runbooks/` | `service-down.md` |
| **Investigation workflows** | `runbooks/` | `alb-5xx-spike.md` |
| **Project planning** | `notes/` | `build-order.md` |
| **Working notes** | `notes/` | `session-checklist.md` |
| **Decision records** | `notes/` | `architecture-decisions.md` |

---

## Documentation Standards

### File naming
- Use lowercase with hyphens: `alarm-configuration.md`
- Descriptive names: `iam-least-privilege.md` not `iam.md`
- Consistent suffixes: `-setup.md`, `-configuration.md`, `-investigation.md`

### Markdown formatting
- Use `#` for main title, `##` for major sections, `###` for subsections
- Include horizontal rules (`---`) between major sections
- Use tables for structured data
- Use code blocks with syntax highlighting
- Include examples where helpful

### Runbook format
All runbooks should follow this template:
```markdown
# Runbook: [Title]

## Symptoms
[What users report or what alarms fire]

## Likely Causes
[Common root causes]

## Investigation Order
1. [First check]
2. [Second check]
3. [Third check]

## Fix Steps
[Resolution procedures]

## Verification
[How to confirm fixed]

## Notes
[Additional context]
```

### Documentation format
All docs should include:
```markdown
# [Title]

## Purpose
[Why this document exists]

## [Main Content Sections]

## Key Takeaways
[Summary of important points]
```

---

## Why This Structure Works

**Clear separation of concerns:**
- `README.md` - Entry point, high-level overview
- `docs/` - Configuration and setup evidence
- `runbooks/` - Troubleshooting procedures
- `notes/` - Planning and working documents

**Easy navigation:**
- Predictable file locations
- Consistent naming conventions
- Logical grouping of related content

**Supports project goals:**
- Monitoring: Dashboard and alarm docs in `docs/`
- Investigation: Runbooks in `runbooks/`
- Planning: Build order and scope in `notes/`
- Evidence: Validation and setup in `docs/`

**Scales well:**
- Adding new runbooks doesn't clutter other areas
- Planning notes separate from final documentation
- Clear home for each type of content

---

## Project Focus Reminder

**Project 2 priorities:**
1. Monitoring (CloudWatch dashboards and alarms)
2. IAM hardening (least-privilege documentation)
3. CloudTrail (investigation capability)
4. Centralized logs (log strategy and investigation)
5. Support-style runbooks (troubleshooting workflows)

**This structure supports these priorities by:**
- Keeping monitoring docs together in `docs/`
- Centralizing investigation workflows in `runbooks/`
- Preserving planning context in `notes/`
- Providing clear project overview in `README.md`

---

## Notes

**Philosophy:**
- Structure serves the content, not vice versa
- Clear organization reduces cognitive load
- Future readers should find what they need quickly
- Documentation should be as easy to navigate as it is to create

**When in doubt:**
- Setup/configuration → `docs/`
- Troubleshooting workflow → `runbooks/`
- Planning/thinking → `notes/`
- Overview/summary → `README.md`

---
