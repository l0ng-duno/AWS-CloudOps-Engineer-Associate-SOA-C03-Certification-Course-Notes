# FCC 41 - AWS Compute Optimizer

> Video chapter [66:29:07](https://youtu.be/5JTBS09e7ew?t=239347) to 66:39:30 · 10 min total · 1 min lecture, 9 min console demo
> Fast path: read this note and skip the video. The demo runs in an account with little to optimise, and he spends most of it wondering why his EC2 instance and Auto Scaling group are not listed.

## Summary

- Compute Optimizer reads the configuration of your resources and their CloudWatch utilisation metrics, then recommends a better size. This is called rightsizing.
- It looks at the last 14 days by default and needs at least 30 hours of metrics before it says anything.
- It covers EC2 instances, Auto Scaling groups, EBS volumes, Lambda functions, ECS services on Fargate and commercial software licences such as SQL Server.
- Each resource gets a finding such as under-provisioned, over-provisioned or optimized, with the recommended setting and the price difference.
- The service is free. You opt in per account, or for a whole organisation.

## Labs and demos

| Time | Len | What it shows | Takeaway | Verdict |
|------|-----|---------------|----------|---------|
| [66:29:50](https://youtu.be/5JTBS09e7ew?t=239390) | 9m | Opens Compute Optimizer in his company account and goes through the Lambda, EBS, Auto Scaling group and EC2 pages | Lambda: 5 of 6 functions are under-provisioned, and one is advised to go from 128 MB to 160 MB of memory. EBS: his volume is already optimized. The Auto Scaling group and the EC2 instance never appear, and he does not find out why. | Skip |

### Useful moments inside the demo

| Time | What you learn |
|------|----------------|
| [66:30:53](https://youtu.be/5JTBS09e7ew?t=239453) | Dashboard: findings are per Region, grouped by resource type |
| [66:32:58](https://youtu.be/5JTBS09e7ew?t=239578) | Account settings: opt-in status and rightsizing preferences |
| [66:34:00](https://youtu.be/5JTBS09e7ew?t=239640) | A Lambda recommendation: current and recommended memory, with the cost of each |
| [66:35:02](https://youtu.be/5JTBS09e7ew?t=239702) | An EBS volume shown as optimized, with recommended size and IOPS |
| [66:38:06](https://youtu.be/5JTBS09e7ew?t=239886) | The requirements page: 30 hours of metrics; enhanced infrastructure metrics as a paid extra |

## Lecture index

| Time | Topic |
|------|-------|
| [66:29:07](https://youtu.be/5JTBS09e7ew?t=239347) | What it analyses, which resources, example recommendation |

## Notes

### What it recommends

| Resource | Recommendation |
|----------|----------------|
| EC2 instance | A different instance type or size, including Graviton options |
| Auto Scaling group | A different instance type for the group |
| EBS volume | Volume type, size and IOPS |
| Lambda function | Memory size |
| ECS service on Fargate | Task CPU and memory |
| Commercial software licence | For example a cheaper SQL Server edition |

### Not in the video

| Topic | Fact |
|-------|------|
| Memory | EC2 memory use is only considered if the CloudWatch agent publishes it. Without it, recommendations rest on CPU, network and disk. |
| Longer history | Enhanced infrastructure metrics (paid) extend the lookback from 14 days to about 3 months |
| It only advises | Nothing is changed for you. You resize the resource yourself. |
| Trusted Advisor | Broad checks across cost, security, fault tolerance, performance and service limits. Compute Optimizer is the specialist for sizing. |
| Cost Explorer | Shows what you spent. Its rightsizing recommendations come from Compute Optimizer. |

## Memory hooks

- Compute Optimizer is a tailor: it measures what you wear (14 days of metrics) and tells you the right size.
- No memory data without the CloudWatch agent, so no memory-based advice.

## Exam traps

- "Find over-provisioned EC2 instances and get a recommended instance type": Compute Optimizer.
- "Recommendations ignore memory use": install the CloudWatch agent so memory metrics exist.
- "A new instance shows no recommendation": it needs at least 30 hours of metrics.
- "Check service limits, open security groups and idle resources in one report": Trusted Advisor, not Compute Optimizer.
- "Automatically add instances when load rises": that is Auto Scaling. Compute Optimizer only recommends.

## Outdated or wrong in the video

- **"Only Auto Scaling groups with one instance type and identical desired, minimum and maximum."** He reads this from the documentation of the time. Compute Optimizer has since added support for groups with scaling policies and mixed instance types. Fairly sure.
- **The resource list.** It now also covers RDS and Aurora databases and reports idle resources. Fairly sure.
- **His EC2 instance never appears.** The video does not establish the reason, so do not draw a rule from it. Very sure.

Related: [[FCC 02 - Amazon EC2]] · [[FCC 05 - EBS]] · [[FCC 20 - ASG]] · [[FCC 23.2 - CloudWatch metrics and the CloudWatch agent]] · [[FCC 00 - Watch guide]]
