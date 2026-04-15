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
