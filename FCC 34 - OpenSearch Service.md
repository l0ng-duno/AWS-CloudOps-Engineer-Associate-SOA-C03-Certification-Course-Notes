# FCC 34 - OpenSearch Service

> Video chapter [63:47:40](https://youtu.be/5JTBS09e7ew?t=229660) to 64:10:48 · 23 min total · 2 min lecture, 21 min lab
> Fast path: read this note and skip the video. The lab is a Ruby script against a throwaway domain. Its one operations lesson is an access policy with an explicit Deny, and that is explained below.

## Summary

- OpenSearch Service is managed full-text search and log analytics. You create a **domain**, which is a cluster, and choose the engine: OpenSearch or the older Elasticsearch.
- OpenSearch is the fork of Elasticsearch and Kibana that AWS made in 2021, after Elastic changed its licence.
- The console offers three flavours: managed domains (you pick instance type and count), Serverless, and Ingestion.
- Two separate interfaces: the AWS API and SDK manage the domain; data goes in and out over HTTPS on the domain endpoint.
- Access is decided by the network choice (VPC or public), the domain access policy, and optional fine-grained access control.

## Labs and demos

| Time | Len | What it shows | Takeaway | Verdict |
|------|-----|---------------|----------|---------|
| [63:49:38](https://youtu.be/5JTBS09e7ew?t=229778) | 21m | Creates the smallest possible domain in the console, then runs a Ruby script that creates an index, adds a document, searches it and deletes it | It works in the end. Creating and deleting the domain each take many minutes. Three stumbles: the request signer wants the access keys passed explicitly; every request fails with "explicit deny in a resource-based policy"; adding an Allow for his user changes nothing until he removes the Deny. About 6 minutes go on Ruby set-up and 4 on the policy. | Skip |

### Useful moments inside the OpenSearch lab

| Time | What you learn |
|------|----------------|
| [63:50:45](https://youtu.be/5JTBS09e7ew?t=229845) | Domain creation choices: dev/test, one AZ, `t3.small.search`, one node, gp3 storage, public access |
| [63:54:00](https://youtu.be/5JTBS09e7ew?t=230040) | Finds Serverless and Ingestion in the console menu |
| [63:55:00](https://youtu.be/5JTBS09e7ew?t=230100) | The AWS SDK only creates, updates and deletes domains. Queries need an OpenSearch client pointed at the endpoint. |
| [64:02:41](https://youtu.be/5JTBS09e7ew?t=230561) | Domain ready: IPv4 or dual-stack endpoint. The Dashboards link answers "User: anonymous is not authorized". |
| [64:05:15](https://youtu.be/5JTBS09e7ew?t=230715) | The explicit Deny error and the access policy under Security configuration |
| [64:08:05](https://youtu.be/5JTBS09e7ew?t=230885) | Removes the Deny statement; the script then runs through index, document, search and delete |

## Lecture index

| Time | Topic |
|------|-------|
| [63:47:44](https://youtu.be/5JTBS09e7ew?t=229664) | What the service is, why AWS forked Elasticsearch, the ELK stack |

## Notes

### The ELK stack and its OpenSearch names

| Elastic product | Role | OpenSearch side |
|-----------------|------|-----------------|
| Elasticsearch | Search and analytics engine, built on Lucene | OpenSearch |
| Logstash | Data processing pipeline | Logstash still works; OpenSearch Ingestion is the AWS option |
| Kibana | Web UI for charts and searching | OpenSearch Dashboards |

Typical use: put application logs in, then search and chart them.

### Domain settings in the lab

| Setting | His choice | Note |
|---------|-----------|------|
| Template | Dev/test, no standby, one AZ | Production offers Multi-AZ with standby |
| Engine | Latest OpenSearch | Elasticsearch versions are also listed |
| Data nodes | One `t3.small.search` | Console says T3 is for test and development only, and Auto-Tune does not support T2 or T3 |
| Dedicated master node | Off | |
| Storage | EBS gp3, a little above the 10 GB minimum | |
| Network | Public access | Console recommends VPC access |
| Fine-grained access control | Off | The form showed an error until he configured it or switched it off |
| Access policy | Left on "do not set" | The domain ended up with a policy that denies everyone |

Free tier he quotes: 750 hours a month on the small instance type.

### The access policy lesson

- Error: `not authorized to perform es:ESHttpPut ... with an explicit deny in a resource-based policy`.
- An explicit Deny always wins. The order of statements does not matter, so adding an Allow next to it does nothing.
- The fix is to replace the Deny with an Allow for the right principal.
- Saving a policy is a domain configuration change. It takes minutes to apply.
- Data requests are signed with SigV4 using the service name `es`. The IAM actions are `es:ESHttpGet`, `es:ESHttpPut`, `es:ESHttpPost` and `es:ESHttpDelete`.

```json
{ "Version": "2012-10-17", "Statement": [ { "Effect": "Allow",
    "Principal": { "AWS": "arn:aws:iam::111122223333:user/example" },
    "Action": "es:ESHttp*",
    "Resource": "arn:aws:es:ca-central-1:111122223333:domain/my-domain/*" } ] }
```

### Not in the video

| Topic | Remember |
|-------|----------|
| Getting logs in | CloudWatch Logs subscription filter, or Amazon Data Firehose, delivers to a domain |
| Cluster health | Green: all shards assigned. Yellow: a replica shard is unassigned (normal on a one-node domain). Red: a primary shard is unassigned. |
| High availability | Multi-AZ, and three dedicated master nodes for production |
| Storage tiers | Hot, UltraWarm (backed by S3), cold |
| Backups | Automated hourly snapshots kept for 14 days; manual snapshots go to your own S3 bucket |
| Alarms | `ClusterStatus.red`, `FreeStorageSpace`, `JVMMemoryPressure` |

## Memory hooks

- OpenSearch is Elasticsearch after the split. Dashboards is Kibana after the split.
- "Domain" means cluster, not a DNS name.
- Manage with the SDK, query over HTTPS.
- Deny beats Allow, wherever it is written.

## Exam traps

- "Search and visualise logs from many sources in near real time": OpenSearch Service with Dashboards.
- "Full-text search for an application": OpenSearch, not RDS or DynamoDB.
- "Kibana" or "Amazon Elasticsearch Service" in an older question: the same service under its old names.
- "Request denied although the IAM user has an Allow": look for an explicit Deny in the domain access policy.
- "Cluster status is yellow on a single-node domain": replicas have nowhere to go; add a node.
- "SQL over files in S3": Athena, not OpenSearch.

## Outdated or wrong in the video

- **He does not know whether Allow or Deny is evaluated first.** Order is irrelevant. An explicit Deny in any applicable policy wins. Very sure.
- **"I'm not sure why it's `es`."** `es` is the service name for managed domains, kept from the days of Amazon Elasticsearch Service. Serverless collections use `aoss`. Very sure.
- **He cannot open Dashboards and leaves it.** A browser does not sign its requests, so it arrives as an anonymous user. Dashboards needs Cognito, SAML, fine-grained access control with a master user, or an IP-based policy. Very sure.
- **"Elastic still market this as free and open."** The licence story has moved on: in 2024 Elastic added an open-source licence again and OpenSearch moved to the Linux Foundation. Not exam material. Fairly sure.

Related: [[FCC 23.1 - CloudWatch Logs and Logs Insights]] · [[FCC 21 - Identity and Access Management IAM]] · [[FCC 28.1 - AWS API, STS and service endpoints]] · [[FCC 00 - Watch guide]]
