# FCC 16 - Amazon Route 53

> Video chapter [37:07:30](https://youtu.be/5JTBS09e7ew?t=133650) to 38:36:48 · 1h29m total · about 23 min lecture, 66 min labs
> Fast path: read this note. To see records behave, watch three short pieces: weighted routing (4 min from [38:01:20](https://youtu.be/5JTBS09e7ew?t=136880)), geolocation tested through a VPN (5 min from [38:26:10](https://youtu.be/5JTBS09e7ew?t=138370)) and an alias to a load balancer (2 min from [38:34:14](https://youtu.be/5JTBS09e7ew?t=138854)). About half of the lab time is template, account and typo trouble. The lecture leaves out alias at the root domain, health check types and logging; they are in the table of points not in the video.

## Summary

- Route 53 registers domains, hosts DNS zones, runs health checks and resolves DNS for VPCs.
- A hosted zone holds the records for one domain or subdomain. Public zones answer the internet; private zones answer only inside associated VPCs.
- An alias record points a name at an AWS resource and follows its IP addresses when they change. Prefer it over a CNAME for AWS targets.
- The routing policy decides which answer a query gets. Exam questions are solved by matching a key word to a policy.
- Health checks run every 30 seconds (10 for fast checks) and drive failover.

## Labs and demos

| Time | Len | What it shows | Takeaway | Verdict |
|------|-----|---------------|----------|---------|
| [37:30:05](https://youtu.be/5JTBS09e7ew?t=135005) | 22m | Points `www` at an EC2 instance with an A record, in the console and then with the CLI | A new zone holds only NS and SOA records. After a stop and start the instance's public IP changes and the site hangs; the fix is an Elastic IP. The CLI `UPSERT` fails on a missing comma, then creates `wwww` by typo. About 10 min (37:32:59 to 37:43:00) go on a template, the wrong account and a wrong VPC ID. | Skip |
| [37:52:00](https://youtu.be/5JTBS09e7ew?t=136320) | 40m | Simple, weighted, multivalue and geolocation records against three web servers in us-east-1 and one in Tokyo | Each policy behaves as the policy table says. Some multivalue lookups fail and he never finds out why. About 17 min go on server templates; a stack cannot create the Tokyo server from us-east-1. | Skim: [38:01:20](https://youtu.be/5JTBS09e7ew?t=136880) to 38:05:45 and [38:26:10](https://youtu.be/5JTBS09e7ew?t=138370) to 38:30:45 |
| [38:31:45](https://youtu.be/5JTBS09e7ew?t=138705) | 5m | Alias record to an Application Load Balancer | Turn on Alias, pick the load balancer type, the Region, then the load balancer. For one in another account, paste its DNS name instead. Ends with a glance at Traffic Flow. | Skim: from [38:34:14](https://youtu.be/5JTBS09e7ew?t=138854) |

### Useful moments inside the routing policies lab

| Time | What you learn |
|------|----------------|
| [37:59:40](https://youtu.be/5JTBS09e7ew?t=136780) | Simple A record, checked with `ping` because browsers cache DNS. TTL defaults to 300 seconds; he lowers it to 60. |
| [38:01:20](https://youtu.be/5JTBS09e7ew?t=136880) | Weighted: three records with the same name and type, each with a record ID and a weight of 1, 2 or 3. |
| [38:04:50](https://youtu.be/5JTBS09e7ew?t=137090) | Repeated `curl` calls: server 3 answers most often, server 1 rarely. |
| [38:06:10](https://youtu.be/5JTBS09e7ew?t=137170) | Multivalue: one record per IP address. Some lookups fail with "could not resolve". |
| [38:12:20](https://youtu.be/5JTBS09e7ew?t=137540) | Simple record holding three IP addresses: all are returned, in random order. |
| [38:26:10](https://youtu.be/5JTBS09e7ew?t=138370) | Geolocation: Canada points at server 2, Japan at the Tokyo server. |
| [38:28:12](https://youtu.be/5JTBS09e7ew?t=138492) | From his PC in Canada he gets server 2. With a VPN connected to Tokyo he gets the Tokyo server, after a short delay. |

## Lecture index

| Time | Topic |
|------|-------|
| [37:07:30](https://youtu.be/5JTBS09e7ew?t=133650) | What Route 53 does, example architecture |
| [37:10:23](https://youtu.be/5JTBS09e7ew?t=133823) | Hosted zones, public and private; delegating a subdomain |
| [37:13:43](https://youtu.be/5JTBS09e7ew?t=134023) | Record types, change batches, alias records |
| [37:16:17](https://youtu.be/5JTBS09e7ew?t=134177) | Traffic Flow |
| [37:16:57](https://youtu.be/5JTBS09e7ew?t=134217) | Routing policies overview |
| [37:18:05](https://youtu.be/5JTBS09e7ew?t=134285) | Simple, weighted ([37:19:06](https://youtu.be/5JTBS09e7ew?t=134346)), failover ([37:19:46](https://youtu.be/5JTBS09e7ew?t=134386)) |
| [37:20:54](https://youtu.be/5JTBS09e7ew?t=134454) | Geolocation, geoproximity ([37:21:44](https://youtu.be/5JTBS09e7ew?t=134504)) |
| [37:23:04](https://youtu.be/5JTBS09e7ew?t=134584) | Latency, multivalue answer ([37:24:23](https://youtu.be/5JTBS09e7ew?t=134663)) |
| [37:25:12](https://youtu.be/5JTBS09e7ew?t=134712) | Health checks |
| [37:26:34](https://youtu.be/5JTBS09e7ew?t=134794) | Route 53 Resolver |
| [37:28:01](https://youtu.be/5JTBS09e7ew?t=134881) | DNSSEC |
| [37:29:23](https://youtu.be/5JTBS09e7ew?t=134963) | Zonal shift |

## Notes

### Hosted zones

- Public zones answer the internet. Private zones answer only inside the VPCs you associate.
- Every new zone starts with an NS and an SOA record.
- To delegate `app.example.com`: create a hosted zone for it, then add an NS record named `app` in the parent zone listing the child zone's name servers. A wildcard record (`*.app.example.com`) in the child zone then gives each customer of a SaaS product a subdomain.

### Records

- A and AAAA map a name to an IPv4 or IPv6 address. CNAME maps a name to another name.
- Alias maps a name to an AWS resource. It is a Route 53 extension, set on an A or AAAA record.
- The API works in batches. Each change is `CREATE`, `DELETE` or `UPSERT` (create, or update if it exists).
- Records that share a name and type each need a set identifier. Every policy except simple uses one.
- Alias targets: CloudFront, Elastic Beanstalk, ELB, S3 website endpoints, API Gateway, VPC interface endpoints, another record in the same zone. Not an EC2 instance: use an A record to an Elastic IP.

```bash
aws route53 change-resource-record-sets --hosted-zone-id Z0123456789ABC \
  --change-batch '{"Changes":[{"Action":"UPSERT","ResourceRecordSet":{
    "Name":"www.example.com","Type":"A","TTL":300,
    "ResourceRecords":[{"Value":"203.0.113.10"}]}}]}'
```

### Routing policies

| Policy | Chooses the answer by | Key words in a question |
|--------|-----------------------|-------------------------|
| Simple | Nothing. One record; several values come back in random order | One resource, no health checks |
| Weighted | Weight from 0 to 255; share = weight ÷ total; 0 stops traffic | Canary, A/B test, 10% to the new version |
| Latency | The Region with the lowest latency for the user | Best performance worldwide |
| Failover | Primary while its health check passes, otherwise secondary | Active-passive, disaster recovery |
| Geolocation | Where the user is: continent, country or US state | Local content, legal restrictions |
| Geoproximity | Distance from user to resource, shifted by a bias from -99 to +99 | Shift part of the traffic between Regions |
| Multivalue answer | Up to 8 healthy records picked at random | Several servers, health-checked, no load balancer |

- Traffic Flow: a visual editor that combines policies, with versions. About $50 a month per policy record.

### Health checks

- Interval: 30 seconds by default, 10 seconds for fast checks.
- An unhealthy result can trigger failover. A CloudWatch alarm on the check can notify you.
- A calculated health check watches other health checks. Checks on endpoints outside AWS cost more.

### Resolver, DNSSEC and zonal shift

| Feature | What it does | Remember |
|---------|--------------|----------|
| Route 53 Resolver | The DNS server built into every VPC, at the VPC network address plus two | Formerly the ".2 resolver" |
| Inbound endpoint | Lets on-premises DNS servers query the VPC | Queries come in to AWS |
| Outbound endpoint | Forwards VPC queries to on-premises DNS, using resolver rules | Queries go out of AWS |
| DNSSEC signing | Lets resolvers verify an answer came from your zone unaltered | Create a key-signing key (KSK), then enable signing |
| Zonal shift | Moves a load balancer's traffic away from one impaired Availability Zone in one action | Part of Application Recovery Controller (ARC); one zone at a time |

### Not in the video

| Topic | Fact |
|-------|------|
| Alias | Allowed at the zone apex (`example.com`), where a CNAME is not. Queries to AWS resources are free. Evaluate Target Health lets it inherit the health of its target. |
| Health check types | Endpoint, calculated, CloudWatch alarm state. Checkers sit on the public internet, so use the alarm type for private resources. |
| Geolocation default | Add a default record, or users from unmatched locations get no answer. |
| Private zones | The VPC needs `enableDnsSupport` and `enableDnsHostnames` set to true. |
| DNSSEC details | Public zones only. The KSK uses a customer managed KMS key in us-east-1. Add a DS record to the parent zone. |
| Logging | Public zone query logs go to CloudWatch Logs. Resolver query logs go to CloudWatch Logs, S3 or Firehose. |

## Memory hooks

- DNS uses port 53, hence the name.
- Geolocation: where is the user. Geoproximity: how far is the user from the resource. Latency: which Region is fastest, not always the nearest.
- Alias follows an AWS name and may sit at the root. CNAME is name to name and may not.

## Exam traps

- "Point `example.com` itself at an ALB or CloudFront": alias A record, not a CNAME.
- "Send 10% of users to the new version": weighted routing.
- "Fail over to a standby Region automatically": failover routing with a health check on the primary.
- "Users in one country must reach servers in that country": geolocation, not latency.
- "Site unreachable after the instance was stopped and started": the public IP changed. Use an Elastic IP or an alias to a load balancer.
- "On-premises servers must resolve names in a private hosted zone": Resolver inbound endpoint. The other direction needs an outbound endpoint and a forwarding rule.

## Outdated or wrong in the video

- **Seven routing policies.** There are eight. IP-based routing, by the client's CIDR range, was added in 2022. Very sure.
- **Geoproximity can only be used through Traffic Flow.** Since January 2024 it can be set on ordinary records; the console in his lab lists it. Very sure.
- **Multivalue answer checks health automatically, and the slide should set Evaluate Target Health to true.** You attach a health check to each multivalue record; one without a check always counts as healthy. Evaluate Target Health belongs to alias records. Fairly sure.
- **Zonal shift only works with cross-zone load balancing off.** It now works with cross-zone on, and with other resources such as Auto Scaling groups. The service has dropped Route 53 from its name. Fairly sure.
- **Amazon SES may force you to use DNSSEC.** SES relies on DKIM, SPF and DMARC records, not DNSSEC. Fairly sure.
- **The record type list.** Route 53 added HTTPS, SVCB, SSHFP and TLSA records in October 2024. Very sure.

Related: [[FCC 14 - Cloud Networking]] · [[FCC 17 - CloudFront]] · [[FCC 18 - AWS Global Accelerator]] · [[FCC 19 - ELB]] · [[FCC 27 - AWS Certificate Manager]] · [[FCC 00 - Watch guide]]
