# FCC 36 - Amazon Detective

> Video chapter [64:17:18](https://youtu.be/5JTBS09e7ew?t=231438) to 64:25:26 · 8 min total · about 2 min lecture, 6 min console demo
> Fast path: read this note and skip the video. For the exam you only need to recognise what Detective is for and how it differs from GuardDuty and Security Hub.

## Summary

- Detective helps you investigate a security finding or suspicious activity and find its root cause. It does not detect or block anything.
- It shows activity per IAM user, IAM role or EC2 instance: API call volume over time, where calls came from on a map, and which calls rose sharply.
- An investigation on an IAM user or role maps activity to attacker tactics and gives a severity.
- It is built on a **behaviour graph** of your account's activity.
- AWS does not class it as a SIEM. You would still use one alongside it.

## Labs and demos

| Time | Len | What it shows | Takeaway | Verdict |
|------|-----|---------------|----------|---------|
| [64:18:52](https://youtu.be/5JTBS09e7ew?t=231532) | 7m | Opens Detective in an account where it is already enabled, reads the summary page, then runs an investigation on his own IAM user | The investigation needs a user or role ARN and a time window of 3 hours to 30 days; his first window is rejected as too long. The report comes back in seconds: tactics with the API calls behind each, then a finding group with an AI-written summary. To see the report layout, 2 minutes from [64:21:21](https://youtu.be/5JTBS09e7ew?t=231681) is enough. | Skip |

## Lecture index

| Time | Topic |
|------|-------|
| [64:17:12](https://youtu.be/5JTBS09e7ew?t=231432) | What Detective is, trends per principal, investigations, behaviour graph (starts a few seconds before the chapter mark) |

## Notes

### What the console shows

| Screen | Content |
|--------|---------|
| Summary | Roles and users with the most API calls, EC2 instances with the most traffic, container clusters, a geolocation map |
| Investigation report | Severity (high, medium, low) and a list of tactics with the matching API calls |
| Finding group | Related findings grouped together with a generated summary, for example "new geolocation" or S3 public access being granted |

### Tactics in his report

| Tactic | Meaning | Calls listed for his user |
|--------|---------|---------------------------|
| Discovery | Learning about the environment | Many read and list calls |
| Initial access | Getting into the network | Console login |
| Persistence | Keeping a foothold | Creating access keys and users |
| Privilege escalation | Gaining higher permissions | Adding a user to a group |
| Impact | Manipulating, interrupting or destroying systems and data | Executing change sets, starting and stopping instances |

### Other points

- Pricing: by volume of data ingested, with a 30-day free trial.
- Enabling it means choosing an administrator account and attaching an IAM policy. He does not show this.
- "WoW" on the charts means week over week.

### Do not confuse

| Service | Question it answers |
|---------|---------------------|
| GuardDuty | Is something malicious happening right now? |
| Inspector | Which workloads have known vulnerabilities? |
| Macie | Where is sensitive data in S3? |
| Security Hub | What are all my findings and compliance scores, in one place? |
| Detective | Why did this finding happen, and what else did that user or instance do? |
| CloudTrail | Who called which API? (the raw record) |

### Not in the video

| Topic | Remember |
|-------|----------|
| Data sources | CloudTrail management events, VPC Flow Logs and GuardDuty findings. Optional: EKS audit logs and Security Hub findings. |
| Collection | Detective reads its own copy of these logs. You do not have to create a trail or turn on flow logs for it. |
| History | Up to one year of data |
| Multi-account | An administrator account brings in member accounts, by invitation or through AWS Organizations |

## Memory hooks

- GuardDuty is the smoke alarm. Detective is the fire investigator.
- Detect: GuardDuty. Collect: Security Hub. Investigate: Detective.

## Exam traps

- "Find the root cause of a GuardDuty finding": Detective.
- "See everything a compromised role did over the past months, without writing queries": Detective.
- "Detect compromised credentials or crypto-mining": GuardDuty, not Detective.
- "One dashboard of findings and compliance checks across accounts": Security Hub.
- "Run custom SQL over CloudTrail logs": Athena or CloudTrail Lake, not Detective.

## Outdated or wrong in the video

- **He reads the tactics list as "what this admin user is able to do".** The report maps what the user did in the chosen window, taken from CloudTrail, to attacker tactics. Detective does not analyse IAM permissions; IAM Access Analyzer does that. Fairly sure.

Related: [[FCC 26 - Security Hub]] · [[FCC 24 - AWS CloudTrail]] · [[FCC 15.2 - VPC gateways, IPv6, endpoints and flow logs]] · [[FCC 21 - Identity and Access Management IAM]] · [[FCC 00 - Watch guide]]
