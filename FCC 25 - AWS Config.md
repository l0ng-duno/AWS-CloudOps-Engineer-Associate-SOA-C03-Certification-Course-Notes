# FCC 25 - AWS Config

> Video chapter [52:06:03](https://youtu.be/5JTBS09e7ew?t=187563) to 52:16:28 · 10 min total · all lecture, no lab
> Fast path: read this note and skip the video. It is slides only, and it leaves out several points the exam asks about. Those are in the table "Not in the video".

## Summary

- Config records how each resource is configured, keeps the history of changes, and checks resources against rules.
- It works per Region and is off by default. Turn it on in every Region you use.
- Rules are AWS managed or custom. A rule runs when a resource changes or on a schedule.
- A non-compliant resource can be remediated by hand or automatically.
- An aggregator shows many accounts and Regions in one place. A conformance pack deploys a set of rules and remediations as one unit.

## Labs and demos

There is no lab or console demo in this chapter.

## Lecture index

| Time | Topic |
|------|-------|
| [52:06:03](https://youtu.be/5JTBS09e7ew?t=187563) | Change management, compliance as code, use cases |
| [52:07:34](https://youtu.be/5JTBS09e7ew?t=187654) | Resource inventory |
| [52:08:38](https://youtu.be/5JTBS09e7ew?t=187718) | Config rules, managed rules |
| [52:09:50](https://youtu.be/5JTBS09e7ew?t=187790) | Custom rules, trigger types |
| [52:11:12](https://youtu.be/5JTBS09e7ew?t=187872) | Resource timeline |
| [52:12:00](https://youtu.be/5JTBS09e7ew?t=187920) | Remediation |
| [52:12:50](https://youtu.be/5JTBS09e7ew?t=187970) | Aggregators |
| [52:14:28](https://youtu.be/5JTBS09e7ew?t=188068) | Conformance packs |
| [52:15:47](https://youtu.be/5JTBS09e7ew?t=188147) | Advanced queries |

## Notes

### Building blocks

| Piece | What it is |
|-------|------------|
| Resource inventory | The list of resources Config has found in the Region, deleted ones included |
| Config rule | A check that marks each resource compliant or non-compliant |
| Configuration timeline | Every change made to one resource, in date order |
| Compliance timeline | When that resource was compliant and when it was not. This is the audit evidence. |
| Remediation | An action that corrects a non-compliant resource, started manually or automatically |
| Aggregator | A copy of Config data from many accounts and Regions, collected in one account |
| Conformance pack | A YAML template holding rules and remediation actions, deployed as one unit |
| Advanced queries | SQL over your resources, across Regions and accounts, with sample queries and JSON or CSV export |

### Rules

| | Managed rule | Custom rule |
|---|--------------|-------------|
| Written by | AWS | You |
| Logic | Predefined, with a few parameters | Your own Lambda function |
| Trigger type | Fixed by AWS | You choose |
| You can adjust | Scope and frequency | Everything |

| Trigger type | Runs when | Setting |
|--------------|-----------|---------|
| Configuration change | A resource in scope is created, changed or deleted | Scope: all changes, chosen resources, or a tag |
| Periodic | On a schedule | Every 1, 3, 6, 12 or 24 hours |

- Every evaluation costs a fraction of a cent.
- Change-triggered rules react at once, for example when an instance is terminated. Periodic rules wait for the next run.
- His managed rule example: check that RDS instances have backups enabled.

### Aggregator

| Setting | Choices |
|---------|---------|
| Source accounts | Listed account IDs, or the whole organisation |
| Regions | Chosen Regions, or all Regions including future ones |
| Authorisation | The source account grants the aggregator account permission to collect its data |

The data stays in the source accounts and is replicated to the aggregator account. The aggregator shows rules and resources across all of them.

### Conformance packs

- Deployed to an account and Region, or across an organisation, from the console or CLI.
- The template looks like a CloudFormation template.
- AWS publishes samples: Operational Best Practices for S3, for DynamoDB and for IAM, CIS, NIST CSF, PCI DSS, and Control Tower detective guardrails.

### Not in the video

| Topic | Fact |
|-------|------|
| Configuration recorder | Nothing is recorded until a recorder is created and started in the Region. It records all supported resource types or a list you choose. Its delivery channel is an S3 bucket, with an optional SNS topic. |
| How remediation runs | A remediation action is a Systems Manager Automation runbook, AWS-provided or your own. Config passes the resource ID and an IAM role for Automation to assume. |
| Detect, not prevent | Config reports and can repair after the event. It never blocks an API call. Blocking needs IAM policies or SCPs. |
| Notifications | Compliance changes go to EventBridge, so a rule can notify SNS or start a Lambda function. |
| Whole organisation | Organisation rules and organisation conformance packs are deployed from the management account or a delegated administrator. An organisation aggregator needs no per-account authorisation. |
| Rule names to recognise | `required-tags`, `encrypted-volumes`, `restricted-ssh`, `s3-bucket-public-read-prohibited`, `cloudtrail-enabled`, `db-instance-backup-enabled` |

## Memory hooks

- Remediation comes from "remedy": you apply a remedy to a problem.
- CloudTrail: who did it. Config: what changed and is it still compliant. CloudWatch: how is it performing.
- Conformance pack is a box set of rules. Aggregator is one window onto many accounts.

## Exam traps

- "What did this security group look like last month?": Config configuration timeline. "Who changed it?": CloudTrail.
- "Automatically fix a non-compliant resource": Config rule plus automatic remediation with a Systems Manager Automation runbook.
- "One compliance view across all accounts and Regions": a Config aggregator.
- "Roll out the same rules and remediations to every account": an organisation conformance pack.
- "Find untagged resources or unencrypted EBS volumes": managed rules `required-tags` and `encrypted-volumes`.
- "Stop users from creating the resource at all": not Config. Use an SCP or an IAM policy.

## Outdated or wrong in the video

- **"Resource inventory is totally free."** Config charges for every configuration item it records, as well as for rule evaluations. Very sure.
- **"Config rules are just Lambdas."** Managed rules are run by AWS, with no function in your account. Custom rules use your Lambda function, or a Guard policy with no function at all. Very sure.
- **"Around 100 managed rules."** There are now several hundred. Fairly sure.
- **"With custom rules it's always on a schedule."** A slip: he means periodic rules. A custom rule can use either trigger type. Very sure.

Related: [[FCC 24 - AWS CloudTrail]] · [[FCC 26 - Security Hub]] · [[FCC 22 - AWS Systems Manager]] · [[FCC 23.3 - EventBridge and CloudWatch alarms]] · [[FCC 29 - AWS CloudFormation]] · [[FCC 00 - Watch guide]]
