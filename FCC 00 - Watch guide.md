# FCC 00 - Watch guide

> Index for the notes on "AWS CloudOps Engineer Associate (SOA-C03) Certification Course" by freeCodeCamp and ExamPro (Andrew Brown): https://youtu.be/5JTBS09e7ew
> The video runs 66h50m. These 51 notes replace it. The clips worth watching add up to about 2h23m, plus 87 minutes of optional ones.

## How to use this guide

- Read a note in place of its chapter. Order inside a note: Summary, the topic tables, Exam traps.
- Every time in a note is a link that opens the video at that moment.
- The "Labs and demos" table in each note says what he is trying to show, what happens, and whether to watch it.
- "Not in the video" tables hold exam points the chapter leaves out. "Outdated or wrong in the video" lists what to unlearn, with how sure the correction is.
- Short on time: do the High notes and the watch list below, then the Exam traps of everything else.

## What the columns mean

| Term | Meaning |
|------|---------|
| **Watch** | Seeing it adds something the note cannot, and it matters for the exam |
| Skim | Only the minutes named in the note are worth it |
| Skip | The note covers it |
| Exam weight | An estimate of how much SOA-C03 asks about the topic: High, Medium or Low |
| Watch (column below) | Minutes of recommended clips, with optional minutes after the plus sign |

## Totals

| | Amount |
|---|--------|
| Video | 66h50m, 42 chapters |
| Notes | 51 (the S3, VPC, CloudWatch and AWS API chapters are split into parts) |
| Reading | About 79,000 words in all; the 20 High notes are about 42,000 |
| Recommended clips | 24 clips, 2h23m |
| Optional clips | 18 clips, 87 min |

## Study order

By exam weight, not video order. Domain percentages are from the exam guide ([[FCC 01 - Introduction]]).

| Step | Area | Notes |
|------|------|-------|
| 1 | Monitoring, logging and remediation (22%) | [[FCC 23.2 - CloudWatch metrics and the CloudWatch agent]] · [[FCC 23.3 - EventBridge and CloudWatch alarms]] · [[FCC 23.1 - CloudWatch Logs and Logs Insights]] · [[FCC 22 - AWS Systems Manager]] · [[FCC 25 - AWS Config]] · [[FCC 24 - AWS CloudTrail]] |
| 2 | Networking and content delivery (18%) | [[FCC 15.1 - VPC core, NACLs, security groups and route tables]] · [[FCC 15.2 - VPC gateways, IPv6, endpoints and flow logs]] · [[FCC 15.3 - VPC VPN, NAT, bastions, Transit Gateway and peering]] · [[FCC 16 - Amazon Route 53]] · [[FCC 14 - Cloud Networking]] · [[FCC 17 - CloudFront]] |
| 3 | Reliability and business continuity (22%) | [[FCC 19 - ELB]] · [[FCC 20 - ASG]] · [[FCC 06 - RDS]] · [[FCC 05 - EBS]] · [[FCC 40 - AWS Backup]] · [[FCC 09.5 - S3 replication, versioning and lifecycle]] |
| 4 | Deployment, provisioning and automation (22%) | [[FCC 29 - AWS CloudFormation]] · [[FCC 02 - Amazon EC2]] · [[FCC 03 - Amazon Machine Images AMIs]] · [[FCC 04 - EC2 Image Builder]] · [[FCC 28.1 - AWS API, STS and service endpoints]] · [[FCC 30 - Elastic Beanstalk Follow Along]] |
| 5 | Security and compliance (16%) | [[FCC 21 - Identity and Access Management IAM]] · [[FCC 09.4 - S3 security and encryption]] · [[FCC 27 - AWS Certificate Manager]] · [[FCC 26 - Security Hub]] |
| 6 | Supporting services | [[FCC 09.2 - S3 buckets and objects]] · [[FCC 09.3 - S3 storage classes]] · [[FCC 07 - Amazon DynamoDB]] · [[FCC 08 - Amazon ElastiCache]] · [[FCC 11 - EFS]] · [[FCC 13 - Storage Gateway]] · [[FCC 31 - SQS]] · [[FCC 32 - SNS]] · [[FCC 28.2 - AWS CLI]] |
| 7 | Low weight: Summary and Exam traps only | [[FCC 09.1 - S3 basics, CLI and IaC tooling]] · [[FCC 10 - Amazon S3 Glacier]] · [[FCC 12 - FSx]] · [[FCC 18 - AWS Global Accelerator]] · [[FCC 33 - Service Catalog]] · [[FCC 34 - OpenSearch Service]] · [[FCC 35 - AWS Firewall Manager]] · [[FCC 36 - Amazon Detective]] · [[FCC 37 - AWS Batch]] · [[FCC 38 - AWS Transfer Family]] · [[FCC 39 - AWS Directory Service]] · [[FCC 41 - AWS Compute Optimizer]] · [[FCC 42 - AWS Snow Family]] |

If CIDR arithmetic is not yet automatic, read [[FCC 14 - Cloud Networking]] before the VPC notes.

## Watch list

The recommended clips in video order: 2h23m in total.

| Time | Len | Note | What you see |
|------|-----|------|--------------|
| [01:09:45](https://youtu.be/5JTBS09e7ew?t=4185) | 11m | [[FCC 02 - Amazon EC2]] | Instance metadata with `curl`, then IMDSv2 enforced and a token fetched |
| [04:50:55](https://youtu.be/5JTBS09e7ew?t=17455) | 4m | [[FCC 05 - EBS]] | Growing an EBS volume with `modify-volume`, then `growpart` and `xfs_growfs` inside the instance |
| [11:57:25](https://youtu.be/5JTBS09e7ew?t=43045) | 12m | [[FCC 09.2 - S3 buckets and objects]] | Object in a Glacier class: download fails, `restore-object`, download works |
| [19:21:40](https://youtu.be/5JTBS09e7ew?t=69700) | 25m | [[FCC 09.2 - S3 buckets and objects]] | Object Lock: governance mode, compliance mode and legal hold |
| [24:01:09](https://youtu.be/5JTBS09e7ew?t=86469) | 6m | [[FCC 09.5 - S3 replication, versioning and lifecycle]] | Cross-Region replication working: rule applied, upload copied, delete not copied |
| [25:37:55](https://youtu.be/5JTBS09e7ew?t=92275) | 7m | [[FCC 09.5 - S3 replication, versioning and lifecycle]] | Lifecycle rule rejected three times, each for a rule worth knowing |
| [26:30:18](https://youtu.be/5JTBS09e7ew?t=95418) | 7m | [[FCC 11 - EFS]] | Mounting EFS from two instances and sharing a file |
| [29:28:25](https://youtu.be/5JTBS09e7ew?t=106105) | 6m | [[FCC 15.1 - VPC core, NACLs, security groups and route tables]] | A CLI-built VPC: DNS hostnames, public IP setting, default route |
| [31:09:59](https://youtu.be/5JTBS09e7ew?t=112199) | 9m | [[FCC 15.1 - VPC core, NACLs, security groups and route tables]] | Blocking one IP address with a NACL deny rule |
| [31:31:43](https://youtu.be/5JTBS09e7ew?t=113503) | 2m | [[FCC 15.1 - VPC core, NACLs, security groups and route tables]] | A security group rule change taking effect at once |
| [38:01:20](https://youtu.be/5JTBS09e7ew?t=136880) | 4m | [[FCC 16 - Amazon Route 53]] | Route 53 weighted routing |
| [38:26:10](https://youtu.be/5JTBS09e7ew?t=138370) | 5m | [[FCC 16 - Amazon Route 53]] | Route 53 geolocation routing, tested through a VPN |
| [38:34:14](https://youtu.be/5JTBS09e7ew?t=138854) | 2m | [[FCC 16 - Amazon Route 53]] | Alias record to a load balancer |
| [40:59:36](https://youtu.be/5JTBS09e7ew?t=147576) | 3m | [[FCC 20 - ASG]] | New launch template version, then the group replacing a terminated instance |
| [46:39:50](https://youtu.be/5JTBS09e7ew?t=167990) | 5m | [[FCC 23.1 - CloudWatch Logs and Logs Insights]] | Creating a metric filter and an alarm on it |
| [49:10:30](https://youtu.be/5JTBS09e7ew?t=177030) | 7m | [[FCC 23.2 - CloudWatch metrics and the CloudWatch agent]] | The CloudWatch agent configuration file |
| [50:17:52](https://youtu.be/5JTBS09e7ew?t=181072) | 4m | [[FCC 23.2 - CloudWatch metrics and the CloudWatch agent]] | Application logs arriving in CloudWatch Logs through the agent |
| [51:22:36](https://youtu.be/5JTBS09e7ew?t=184956) | 5m | [[FCC 24 - AWS CloudTrail]] | CloudTrail console tour: Event history and every trail option |
| [53:29:17](https://youtu.be/5JTBS09e7ew?t=192557) | 5m | [[FCC 28.1 - AWS API, STS and service endpoints]] | Trust policy fixed, `assume-role` works, temporary credentials in use |
| [58:23:00](https://youtu.be/5JTBS09e7ew?t=210180) | 2m | [[FCC 29 - AWS CloudFormation]] | Reading a CloudFormation change set |
| [59:58:49](https://youtu.be/5JTBS09e7ew?t=215929) | 2m | [[FCC 29 - AWS CloudFormation]] | Update denied by a stack policy |
| [60:27:13](https://youtu.be/5JTBS09e7ew?t=217633) | 4m | [[FCC 29 - AWS CloudFormation]] | Stack delete fails on a non-empty bucket; retain the resource |
| [61:36:07](https://youtu.be/5JTBS09e7ew?t=221767) | 2m | [[FCC 30 - Elastic Beanstalk Follow Along]] | Elastic Beanstalk blue/green: swapping environment URLs |
| [62:48:05](https://youtu.be/5JTBS09e7ew?t=226085) | 4m | [[FCC 31 - SQS]] | FIFO queue: three errors that show how it differs from standard |

## Optional clips

| Time | Len | Note | What you see |
|------|-----|------|--------------|
| [02:58:06](https://youtu.be/5JTBS09e7ew?t=10686) | 5m | [[FCC 03 - Amazon Machine Images AMIs]] | Copying and sharing an AMI from the CLI |
| [17:27:16](https://youtu.be/5JTBS09e7ew?t=62836) | 10m | [[FCC 09.2 - S3 buckets and objects]] | Querying S3 server access logs with Athena |
| [20:31:50](https://youtu.be/5JTBS09e7ew?t=73910) | 4m | [[FCC 09.3 - S3 storage classes]] | Storage class comparison slide |
| [21:18:22](https://youtu.be/5JTBS09e7ew?t=76702) | 6m | [[FCC 09.4 - S3 security and encryption]] | Cross-account access to a bucket with a bucket policy |
| [23:20:21](https://youtu.be/5JTBS09e7ew?t=84021) | 2m | [[FCC 09.4 - S3 security and encryption]] | SSE-C: download refused without the key, fine with it |
| [24:40:01](https://youtu.be/5JTBS09e7ew?t=88801) | 10m | [[FCC 09.5 - S3 replication, versioning and lifecycle]] | Reading versions, delete markers and restoring a deleted object |
| [25:15:37](https://youtu.be/5JTBS09e7ew?t=90937) | 10m | [[FCC 09.5 - S3 replication, versioning and lifecycle]] | MFA Delete in action, then suspended versioning |
| [26:57:49](https://youtu.be/5JTBS09e7ew?t=97069) | 4m | [[FCC 13 - Storage Gateway]] | A file written to a Storage Gateway share appearing in S3 |
| [28:21:59](https://youtu.be/5JTBS09e7ew?t=102119) | 2m | [[FCC 14 - Cloud Networking]] | A misaligned subnet CIDR being refused |
| [32:06:16](https://youtu.be/5JTBS09e7ew?t=115576) | 4m | [[FCC 15.2 - VPC gateways, IPv6, endpoints and flow logs]] | IPv6 routes with an egress-only internet gateway (more short clips are listed in the note) |
| [33:50:02](https://youtu.be/5JTBS09e7ew?t=121802) | 2m | [[FCC 15.2 - VPC gateways, IPv6, endpoints and flow logs]] | Gateway endpoint works; delete it and the function times out |
| [34:07:25](https://youtu.be/5JTBS09e7ew?t=122845) | 2m | [[FCC 15.2 - VPC gateways, IPv6, endpoints and flow logs]] | VPC flow log records arriving |
| [36:34:36](https://youtu.be/5JTBS09e7ew?t=131676) | 7m | [[FCC 15.3 - VPC VPN, NAT, bastions, Transit Gateway and peering]] | Reachability Analyzer finding a missing route to a transit gateway |
| [36:52:01](https://youtu.be/5JTBS09e7ew?t=132721) | 3m | [[FCC 15.3 - VPC VPN, NAT, bastions, Transit Gateway and peering]] | VPC peering commands |
| [40:06:14](https://youtu.be/5JTBS09e7ew?t=144374) | 4m | [[FCC 19 - ELB]] | ALB: HTTPS listener rejected, healthy target, security group chain |
| [42:11:02](https://youtu.be/5JTBS09e7ew?t=151862) | 3m | [[FCC 21 - Identity and Access Management IAM]] | Bucket ARN versus object ARN in the IAM visual editor |
| [52:32:05](https://youtu.be/5JTBS09e7ew?t=189125) | 5m | [[FCC 27 - AWS Certificate Manager]] | Requesting an ACM certificate and adding an HTTPS listener |
| [54:54:45](https://youtu.be/5JTBS09e7ew?t=197685) | 4m | [[FCC 28.2 - AWS CLI]] | `--max-items`, `--page-size` and `--starting-token` |

## All chapters

| Note | Starts | Length | Exam weight | Watch |
|------|--------|--------|-------------|-------|
| [[FCC 01 - Introduction]] | [00:00:00](https://youtu.be/5JTBS09e7ew?t=0) | 35 min | Medium | 0 |
| [[FCC 02 - Amazon EC2]] | [00:34:39](https://youtu.be/5JTBS09e7ew?t=2079) | 1h59m | High | 11 min |
| [[FCC 03 - Amazon Machine Images AMIs]] | [02:34:01](https://youtu.be/5JTBS09e7ew?t=9241) | 30 min | Medium | 0 + 5 |
| [[FCC 04 - EC2 Image Builder]] | [03:03:53](https://youtu.be/5JTBS09e7ew?t=11033) | 1h26m | Medium | 0 |
| [[FCC 05 - EBS]] | [04:30:23](https://youtu.be/5JTBS09e7ew?t=16223) | 46 min | High | 4 min |
| [[FCC 06 - RDS]] | [05:16:02](https://youtu.be/5JTBS09e7ew?t=18962) | 1h11m | High | 0 |
| [[FCC 07 - Amazon DynamoDB]] | [06:26:35](https://youtu.be/5JTBS09e7ew?t=23195) | 31 min | Medium | 0 |
| [[FCC 08 - Amazon ElastiCache]] | [06:57:34](https://youtu.be/5JTBS09e7ew?t=25054) | 45 min | Medium | 0 |
| [[FCC 09.1 - S3 basics, CLI and IaC tooling]] | [07:42:14](https://youtu.be/5JTBS09e7ew?t=27734) | 4h14m | Low | 0 |
| [[FCC 09.2 - S3 buckets and objects]] | [11:56:20](https://youtu.be/5JTBS09e7ew?t=42980) | 8h04m | Medium | 37 min + 10 |
| [[FCC 09.3 - S3 storage classes]] | [20:00:45](https://youtu.be/5JTBS09e7ew?t=72045) | 35 min | Medium | 0 + 4 |
| [[FCC 09.4 - S3 security and encryption]] | [20:35:39](https://youtu.be/5JTBS09e7ew?t=74139) | 3h04m | High | 0 + 8 |
| [[FCC 09.5 - S3 replication, versioning and lifecycle]] | [23:39:40](https://youtu.be/5JTBS09e7ew?t=85180) | 2h08m | High | 13 min + 20 |
| [[FCC 10 - Amazon S3 Glacier]] | [25:48:05](https://youtu.be/5JTBS09e7ew?t=92885) | 24 min | Low | 0 |
| [[FCC 11 - EFS]] | [26:12:24](https://youtu.be/5JTBS09e7ew?t=94344) | 25 min | Medium | 7 min |
| [[FCC 12 - FSx]] | [26:37:17](https://youtu.be/5JTBS09e7ew?t=95837) | 11 min | Low | 0 |
| [[FCC 13 - Storage Gateway]] | [26:48:28](https://youtu.be/5JTBS09e7ew?t=96508) | 20 min | Medium | 0 + 4 |
| [[FCC 14 - Cloud Networking]] | [27:08:51](https://youtu.be/5JTBS09e7ew?t=97731) | 1h21m | Medium | 0 + 2 |
| [[FCC 15.1 - VPC core, NACLs, security groups and route tables]] | [28:29:36](https://youtu.be/5JTBS09e7ew?t=102576) | 3h18m | High | 17 min |
| [[FCC 15.2 - VPC gateways, IPv6, endpoints and flow logs]] | [31:48:03](https://youtu.be/5JTBS09e7ew?t=114483) | 2h23m | High | 0 + 8 |
| [[FCC 15.3 - VPC VPN, NAT, bastions, Transit Gateway and peering]] | [34:10:38](https://youtu.be/5JTBS09e7ew?t=123038) | 2h57m | High | 0 + 10 |
| [[FCC 16 - Amazon Route 53]] | [37:07:30](https://youtu.be/5JTBS09e7ew?t=133650) | 1h29m | High | 11 min |
| [[FCC 17 - CloudFront]] | [38:36:48](https://youtu.be/5JTBS09e7ew?t=139008) | 34 min | Medium | 0 |
| [[FCC 18 - AWS Global Accelerator]] | [39:11:18](https://youtu.be/5JTBS09e7ew?t=141078) | 16 min | Low | 0 |
| [[FCC 19 - ELB]] | [39:27:09](https://youtu.be/5JTBS09e7ew?t=142029) | 1h04m | High | 0 + 4 |
| [[FCC 20 - ASG]] | [40:31:14](https://youtu.be/5JTBS09e7ew?t=145874) | 54 min | High | 3 min |
| [[FCC 21 - Identity and Access Management IAM]] | [41:25:03](https://youtu.be/5JTBS09e7ew?t=149103) | 1h17m | High | 0 + 3 |
| [[FCC 22 - AWS Systems Manager]] | [42:41:46](https://youtu.be/5JTBS09e7ew?t=153706) | 1h21m | High | 0 |
| [[FCC 23.1 - CloudWatch Logs and Logs Insights]] | [44:03:12](https://youtu.be/5JTBS09e7ew?t=158592) | 2h54m | High | 5 min |
| [[FCC 23.2 - CloudWatch metrics and the CloudWatch agent]] | [46:57:20](https://youtu.be/5JTBS09e7ew?t=169040) | 3h34m | High | 11 min |
| [[FCC 23.3 - EventBridge and CloudWatch alarms]] | [50:31:25](https://youtu.be/5JTBS09e7ew?t=181885) | 26 min | High | 0 |
| [[FCC 24 - AWS CloudTrail]] | [50:57:43](https://youtu.be/5JTBS09e7ew?t=183463) | 1h08m | High | 5 min |
| [[FCC 25 - AWS Config]] | [52:06:03](https://youtu.be/5JTBS09e7ew?t=187563) | 10 min | High | 0 |
| [[FCC 26 - Security Hub]] | [52:16:28](https://youtu.be/5JTBS09e7ew?t=188188) | 5 min | Low | 0 |
| [[FCC 27 - AWS Certificate Manager]] | [52:21:05](https://youtu.be/5JTBS09e7ew?t=188465) | 19 min | Medium | 0 + 5 |
| [[FCC 28.1 - AWS API, STS and service endpoints]] | [52:39:39](https://youtu.be/5JTBS09e7ew?t=189579) | 1h46m | Medium | 5 min |
| [[FCC 28.2 - AWS CLI]] | [54:26:00](https://youtu.be/5JTBS09e7ew?t=195960) | 3h30m | Medium | 0 + 4 |
| [[FCC 29 - AWS CloudFormation]] | [57:56:14](https://youtu.be/5JTBS09e7ew?t=208574) | 2h44m | High | 8 min |
| [[FCC 30 - Elastic Beanstalk Follow Along]] | [60:40:12](https://youtu.be/5JTBS09e7ew?t=218412) | 1h18m | Medium | 2 min |
| [[FCC 31 - SQS]] | [61:58:19](https://youtu.be/5JTBS09e7ew?t=223099) | 1h01m | Medium | 4 min |
| [[FCC 32 - SNS]] | [62:59:16](https://youtu.be/5JTBS09e7ew?t=226756) | 41 min | Medium | 0 |
| [[FCC 33 - Service Catalog]] | [63:40:12](https://youtu.be/5JTBS09e7ew?t=229212) | 7 min | Low | 0 |
| [[FCC 34 - OpenSearch Service]] | [63:47:40](https://youtu.be/5JTBS09e7ew?t=229660) | 23 min | Low | 0 |
| [[FCC 35 - AWS Firewall Manager]] | [64:10:48](https://youtu.be/5JTBS09e7ew?t=231048) | 6 min | Low | 0 |
| [[FCC 36 - Amazon Detective]] | [64:17:18](https://youtu.be/5JTBS09e7ew?t=231438) | 8 min | Low | 0 |
| [[FCC 37 - AWS Batch]] | [64:25:26](https://youtu.be/5JTBS09e7ew?t=231926) | 28 min | Low | 0 |
| [[FCC 38 - AWS Transfer Family]] | [64:53:15](https://youtu.be/5JTBS09e7ew?t=233595) | 1h17m | Low | 0 |
| [[FCC 39 - AWS Directory Service]] | [66:10:28](https://youtu.be/5JTBS09e7ew?t=238228) | 3 min | Low | 0 |
| [[FCC 40 - AWS Backup]] | [66:12:59](https://youtu.be/5JTBS09e7ew?t=238379) | 16 min | High | 0 |
| [[FCC 41 - AWS Compute Optimizer]] | [66:29:07](https://youtu.be/5JTBS09e7ew?t=239347) | 10 min | Low | 0 |
| [[FCC 42 - AWS Snow Family]] | [66:39:30](https://youtu.be/5JTBS09e7ew?t=239970) | 11 min | Low | 0 |

## Limits of these notes

- They were written from the video's automatic captions. Slides and code on screen were not seen, so commands are reconstructed from what he says and from general AWS knowledge.
- Captions are grouped by the minute. A linked time may land up to a minute before the moment described.
- Lecture and lab minutes are estimates.
- Corrections reflect AWS as of October 2026. Items marked "Fairly sure" deserve a check before you rely on them.

Related: [[FCC 01 - Introduction]] · [[FCC 23.2 - CloudWatch metrics and the CloudWatch agent]] · [[FCC 15.1 - VPC core, NACLs, security groups and route tables]] · [[FCC 29 - AWS CloudFormation]]
