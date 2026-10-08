# FCC 35 - AWS Firewall Manager

> Video chapter [64:10:48](https://youtu.be/5JTBS09e7ew?t=231048) to 64:17:18 · 7 min total · 2 min lecture, 5 min console demo
> Fast path: read this note and skip the video. He never creates a policy because of the price, so the demo is a click-through of a wizard.

## Summary

- Firewall Manager centrally configures and manages firewall rules across the accounts of an AWS Organization.
- It is a manager, not a firewall. It rolls out policies for other services: WAF, Shield Advanced, security groups, network ACLs, Network Firewall, Route 53 Resolver DNS Firewall and third-party firewalls.
- Three prerequisites: the account is in AWS Organizations, a Firewall Manager administrator account is set, and AWS Config is enabled in each account and Region.
- It is expensive: $100 per policy, per Region, per month.

## Labs and demos

| Time | Len | What it shows | Takeaway | Verdict |
|------|-----|---------------|----------|---------|
| [64:12:39](https://youtu.be/5JTBS09e7ew?t=231159) | 5m | In an account that already runs AWS Config, he sets the Firewall Manager administrator account, then clicks through "create policy" for security groups without saving | Wizard steps: policy type, rules, action, scope (accounts and resource types), tags. Switching the type to WAF gives a completely different form. Nothing is created. | Skip |

## Lecture index

| Time | Topic |
|------|-------|
| [64:10:50](https://youtu.be/5JTBS09e7ew?t=231050) | What it manages, why the forms differ per service, cost |
| [64:11:54](https://youtu.be/5JTBS09e7ew?t=231114) | Prerequisites, and his view that it sits on top of AWS Config |

## Notes

### What a policy can manage

| Policy type | What that service protects |
|-------------|----------------------------|
| AWS WAF | Web requests to CloudFront, load balancers, API Gateway |
| Shield Advanced | DDoS protection |
| Security groups | Instance and network interface traffic |
| Network ACLs | Subnet traffic |
| AWS Network Firewall | Traffic entering and leaving a VPC |
| Route 53 Resolver DNS Firewall | Outbound DNS queries from VPCs |
| Third-party firewalls | Partner firewalls from AWS Marketplace |

### Prerequisites

| Need | Detail |
|------|--------|
| AWS Organizations | The account must be a member of an organisation |
| Administrator account | One account is designated as the Firewall Manager administrator. In the demo this takes a minute to appear under Settings. |
| AWS Config | Enabled for every account and every Region you want covered. Config is set up per Region. |
| AWS RAM | Only for some policy types, such as Network Firewall and DNS Firewall |

### Security group policy options he reads out

- Rules: report when a security group created by the policy becomes non-compliant; remove any other security groups from resources in scope; copy tags from the primary security group.
- Action: identify resources that do not comply. The other choice is to remediate them automatically.
- Scope: which accounts, which resource types (he ticks EC2 instance), and whether protection is removed when a resource leaves the scope.

### Not in the video

| Topic | Remember |
|-------|----------|
| Why use it | A policy applies to existing resources and to new accounts and resources as they appear in scope |
| Security group policy kinds | Common (push the same groups everywhere), content audit (find or fix rules that break your standard), usage audit (unused or redundant groups) |

## Memory hooks

- Firewall Manager is the manager, not a firewall.
- Three keys to switch it on: Organization, Administrator account, Config.
- WAF writes the rules. Firewall Manager makes every account use them.

## Exam traps

- "Apply the same WAF rules to every load balancer in all accounts, including accounts created later": Firewall Manager.
- "Audit or remove over-permissive security group rules across the organisation": Firewall Manager security group policy.
- "Cannot create a Firewall Manager policy": check Organizations membership, the administrator account, and AWS Config in that Region.
- "Block bad domains for every VPC in the organisation": DNS Firewall rules deployed through Firewall Manager.
- Service control policies limit IAM permissions. They do not configure firewalls.

## Outdated or wrong in the video

- **"$50" in the lecture.** The console in his own demo shows the real figure: $100 per policy per Region per month. Very sure.
- **"If you're an admin you have access to everything", so the administrator requirement is no issue.** The Firewall Manager administrator is an account that the organisation's management account designates, not an IAM permission level. He sets it a minute later. Very sure.
- **"You could just make AWS Config rules yourself."** Only partly. Firewall Manager does use Config to find resources and check compliance, but it also creates and attaches the web ACLs, security groups and firewalls in each account. Fairly sure.

Related: [[FCC 25 - AWS Config]] · [[FCC 15.1 - VPC core, NACLs, security groups and route tables]] · [[FCC 26 - Security Hub]] · [[FCC 21 - Identity and Access Management IAM]] · [[FCC 00 - Watch guide]]
