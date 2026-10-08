# FCC 01 - Introduction

> Video chapter [00:00:00](https://youtu.be/5JTBS09e7ew?t=0) to 00:34:39 · 35 min · all lecture, no labs
> Fast path: read this note and skip the whole chapter.

## Summary

- SOA-C03 is the renamed SysOps Administrator Associate. The code stayed "SOA", and the content moved toward automation and containers.
- 65 questions in 130 minutes, pass mark 720 out of 1000, five domains.
- The instructor's study advice: split time evenly between lectures with labs and practice exams, and aim for 85% on practice exams.
- He says networking and troubleshooting are the heart of this exam, and that the labs matter more here than for Solutions Architect Associate.

## Sections

| Time | Len | Topic | Verdict |
|------|-----|-------|---------|
| [00:00:00](https://youtu.be/5JTBS09e7ew?t=0) | 1m | Welcome and sales pitch | Skip |
| [00:01:08](https://youtu.be/5JTBS09e7ew?t=68) | 12m | What the certification is, who it suits, exam logistics | Skip |
| [00:13:03](https://youtu.be/5JTBS09e7ew?t=783) | 22m | Reads through the official exam guide, domain by domain | Skip |

## Notes

### Exam facts

| Item | Value |
|------|-------|
| Questions | 65: 50 scored, 15 unscored |
| Time | 130 minutes (allow 160 minutes including check-in) |
| Pass mark | 720 / 1000 |
| Question types | Multiple choice and multiple response |
| Wrong answers | No penalty, so never leave a question blank |
| Delivery | Pearson VUE, test centre or online |
| Validity | 3 years |

### Domains

| # | Domain | Weight | What the instructor highlights |
|---|--------|--------|--------------------------------|
| 1 | Monitoring, logging, analysis, remediation, performance optimization | 22% | CloudWatch metrics, alarms, agent, dashboards; EventBridge; SSM Automation runbooks; EBS, S3, EFS and RDS performance |
| 2 | Reliability and business continuity | 22% | Caching, scaling, ELB and Route 53 health checks, Multi-AZ, backups, RTO and RPO |
| 3 | Deployment, provisioning, automation | 22% | AMIs and Image Builder, CloudFormation, CDK, StackSets, AWS RAM, Systems Manager |
| 4 | Security and compliance | 16% | IAM, CloudTrail, Access Analyzer, KMS, ACM, Secrets Manager, Security Hub, GuardDuty, Config, Inspector |
| 5 | Networking and content delivery | 18% | VPC configuration and connectivity troubleshooting |

### What changed from SOA-C02

| Change | Detail |
|--------|--------|
| Domains | Six became five; the cost and performance optimization domain was removed |
| Added | CloudWatch agent on ECS and EKS, CloudFormation and CDK, enforcing compliance by Region and service, CloudWatch network monitoring |
| Removed | S3 static website hosting |

### Listed as out of scope in the exam guide

- Designing distributed architectures and CI/CD pipelines.
- Developing software.
- Defining security, compliance and governance requirements.
- Capacity planning, total cost of ownership analysis, billing and invoicing.

### Instructor's advice

- Study 1 to 2 hours a day; he estimates 24 to 28 hours for most people and 60 or more for beginners.
- Do Solutions Architect Associate and Developer Associate first if you can, since this exam borrows from both.
- If a question looks impossibly hard, it may be one of the 15 unscored ones. Stay calm and move on.
- Know Prometheus and RDS Proxy at the "what is it" level only.
- He has never seen Terraform or Git code in a question, although both are listed in the guide.

## Outdated or wrong in the video

- **"720 means about 72%."** The score is scaled, so it is not a simple percentage. Fairly sure.
- **"You can afford to get 15 questions wrong."** This mixes up the 15 unscored questions with a margin for error. The number you can miss is not published. Fairly sure.

Related: [[FCC 00 - Watch guide]] · [[fcc-soa-c03-overview]] · [[FCC 02 - Amazon EC2]]
