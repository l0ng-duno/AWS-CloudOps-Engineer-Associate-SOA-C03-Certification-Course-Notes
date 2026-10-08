# FCC 26 - Security Hub

> Video chapter [52:16:28](https://youtu.be/5JTBS09e7ew?t=188188) to 52:21:05 · 5 min total · 1 min lecture, 4 min console demo
> Fast path: read this note and skip the video. The demo only looks at a dashboard and never enables the service.

## Summary

- Security Hub is a cloud security posture management (CSPM) service. It scores the account against security standards and lists the findings.
- A standard is a collection of controls. Most controls run as AWS Config rules that Security Hub creates.
- AWS Config must be enabled in the Region before Security Hub is useful.

## Labs and demos

| Time | Len | What it shows | Takeaway | Verdict |
|------|-----|---------------|----------|---------|
| [52:17:21](https://youtu.be/5JTBS09e7ew?t=188241) | 4m | The Security Hub dashboard in an account with workloads, then the sign-up page in a clean account | The account scores 33% against the CIS AWS Foundations Benchmark. Findings are sorted by severity, for example hardware MFA on the root account. The same controls show up as rules in the AWS Config console. The sign-up page offers a 30-day trial and asks for Config first, with a CloudFormation template that creates a bucket, an SNS topic and a configuration recorder. He does not enable it. | Skip |

## Lecture index

| Time | Topic |
|------|-------|
| [52:16:30](https://youtu.be/5JTBS09e7ew?t=188190) | What Security Hub is: standards, controls, security score |

## Notes

### Terms

| Term | Meaning |
|------|---------|
| Security standard | A named set of controls, such as the CIS AWS Foundations Benchmark |
| Control | One check, run as an AWS Config rule |
| Finding | A failed check, with a severity, that you can open and investigate |
| Security score | The share of controls that pass, per standard |

### Not in the video

| Topic | Fact |
|-------|------|
| Other standards | AWS Foundational Security Best Practices, PCI DSS, NIST SP 800-53 |
| Finding sources | GuardDuty, Inspector, Macie, IAM Access Analyzer, Firewall Manager and partner tools, all in one format (ASFF) |
| Automation | Findings are sent to EventBridge. A rule can notify SNS or start Lambda or Systems Manager Automation. |
| Many accounts and Regions | Enabled per Region. A delegated administrator through AWS Organizations sees all accounts, and other Regions can be aggregated into one home Region. |

## Memory hooks

- Config checks one rule. A conformance pack bundles rules. Security Hub bundles rules, adds a score and charts, and collects findings from other services.
- Hub and spokes: GuardDuty, Inspector and Macie are the spokes that feed it.

## Exam traps

- "One place to see findings from GuardDuty, Inspector and Macie across accounts": Security Hub.
- "Security Hub controls show no results": AWS Config is not recording in that Region.
- "Detect threats from CloudTrail, VPC flow logs and DNS logs": GuardDuty, not Security Hub.
- "Act automatically on a finding": an EventBridge rule on Security Hub findings.

## Outdated or wrong in the video

- **The name.** The service he shows is now called AWS Security Hub CSPM. "AWS Security Hub" is a newer, wider service, generally available since December 2025, that correlates findings from GuardDuty, Inspector, Macie and Security Hub CSPM. Exam questions may still use the old name for standards and scores. Fairly sure.

Related: [[FCC 25 - AWS Config]] · [[FCC 24 - AWS CloudTrail]] · [[FCC 35 - AWS Firewall Manager]] · [[FCC 36 - Amazon Detective]] · [[FCC 00 - Watch guide]]
