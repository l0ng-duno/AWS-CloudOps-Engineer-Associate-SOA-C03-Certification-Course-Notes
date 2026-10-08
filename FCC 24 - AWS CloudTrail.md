# FCC 24 - AWS CloudTrail

> Video chapter [50:57:43](https://youtu.be/5JTBS09e7ew?t=183463) to 52:06:03 · 1h08m total · about 5 min lecture, 63 min labs and demos
> Fast path: read this note, then watch the 5-minute console tour at [51:22:36](https://youtu.be/5JTBS09e7ew?t=184956). The three long labs are mostly IAM and bucket-policy debugging. Their lessons are in the tables below.

## Summary

- CloudTrail records the API calls made in an account: who, what, when and from where. It is the audit service, for "who to blame".
- Event history is on by default, free, and keeps 90 days of management events.
- For anything longer, create a trail. A trail delivers log files to an S3 bucket and, if you choose, to CloudWatch Logs.
- Trail options to know: all Regions, whole organisation, SSE-KMS encryption, log file validation.
- Management events are logged by default. Data events (S3 object calls, Lambda invocations) are off by default because of volume and cost.

## Labs and demos

| Time | Len | What it shows | Takeaway | Verdict |
|------|-----|---------------|----------|---------|
| [51:02:46](https://youtu.be/5JTBS09e7ew?t=183766) | 20m | Creates a trail with the CLI: bucket, bucket policy, `create-trail`, `start-logging`, then DynamoDB changes to produce events | `create-trail` fails with "Incorrect S3 bucket policy" until the policy has both statements, `s3:GetBucketAcl` and `s3:PutObject`. 8 minutes go on this. A CLI-made trail does not log until `start-logging`. Files reach S3 after about 5 minutes as gzipped JSON. | Skip |
| [51:22:36](https://youtu.be/5JTBS09e7ew?t=184956) | 5m | Console tour from an older recording: Event history, "View event", the create-trail form with every option, then the CloudWatch Logs setting on the trail page | The quickest way to see all the tested options on one screen. The "apply to my organisation" box only appears in the management account. | **Watch** |
| [51:27:28](https://youtu.be/5JTBS09e7ew?t=185248) | 22m | Sends the trail to CloudWatch Logs with `update-trail`, a log group and an IAM role | About 11 minutes of failures. The IAM console's CloudTrail use case offers one fixed policy, so he builds the role with a custom trust policy. Then "access denied": the policy had the wrong Region and a made-up stream name. Once fixed, events appear in the log group. The console does the same in a few clicks. | Skip |
| [51:49:56](https://youtu.be/5JTBS09e7ew?t=186596) | 14m | CloudTrail Lake: creates an event data store and queries it with SQL | The docs' sample query is rejected. His own query returns nothing until new events arrive, because a new store starts empty. Delete is blocked until termination protection and Lake query federation are off. Lake has been closed to new customers since 31 May 2026. | Skip |
| [52:03:35](https://youtu.be/5JTBS09e7ew?t=187415) | 2m | "Create Athena table" link in the CloudTrail console, then a `SELECT DISTINCT` query in Athena | One click builds an Athena table over the trail's bucket. This is the classic way to query trail logs with SQL. | Skip |

### Useful moments inside the CLI labs

| Time | What you learn |
|------|----------------|
| [51:06:52](https://youtu.be/5JTBS09e7ew?t=184012) | `create-trail` rejected: the bucket has no policy that lets CloudTrail write |
| [51:15:06](https://youtu.be/5JTBS09e7ew?t=184506) | Works once both policy statements are present; the console shows the new trail with logging off |
| [51:16:09](https://youtu.be/5JTBS09e7ew?t=184569) | He finds `start-logging` |
| [51:19:15](https://youtu.be/5JTBS09e7ew?t=184755) | Delivery takes about 5 minutes; he opens a gzipped log file from S3 |
| [51:39:45](https://youtu.be/5JTBS09e7ew?t=185985) | Role created with a custom trust policy for `cloudtrail.amazonaws.com` |
| [51:45:02](https://youtu.be/5JTBS09e7ew?t=186302) | The fix: the log stream part of the role policy is a prefix ending in `*`, not a stream you create |
| [51:48:10](https://youtu.be/5JTBS09e7ew?t=186490) | Trail events readable in the CloudWatch log group |

## Lecture index

| Time | Topic |
|------|-------|
| [50:57:43](https://youtu.be/5JTBS09e7ew?t=183463) | What CloudTrail is, fields of an event, Event history and the 90-day limit, Athena |
| [51:00:03](https://youtu.be/5JTBS09e7ew?t=183603) | Trail options: all Regions, organisation, SSE-KMS, log file validation |
| [51:01:00](https://youtu.be/5JTBS09e7ew?t=183660) | Delivery to CloudWatch Logs, management and data events |

## Notes

### What one event records

| Question | Fields |
|----------|--------|
| Where | Account ID, source IP address |
| When | Event time |
| Who | User identity (user or role), user agent (console, CLI, SDK, operating system) |
| What | Region, event source (the service, e.g. `iam.amazonaws.com`), event name (the action, e.g. `CreateUser`) |

### Event history, trail, Lake

| | Event history | Trail | CloudTrail Lake |
|---|---------------|-------|-----------------|
| Setup | On by default | You create it | You create an event data store |
| Keeps | 90 days | As long as the S3 bucket keeps the files | The retention you set |
| Events | Management only | Management, plus data and Insights events if selected | Chosen per store |
| Search | Console filters | Athena, or Logs Insights if also sent to CloudWatch Logs | SQL editor in the CloudTrail console |

### Trail options

| Option | What it does | CLI flag |
|--------|--------------|----------|
| All Regions | One trail covers every Region, so none is missed | `--is-multi-region-trail` |
| Organisation | One trail covers every account in AWS Organizations | `--is-organization-trail` |
| SSE-KMS | Encrypts the log files with your KMS key (about $1 a month per key) | `--kms-key-id` |
| Log file validation | Shows whether a log file was changed or deleted. It detects tampering; it does not prevent it. | `--enable-log-file-validation` |
| CloudWatch Logs | Copies events to a log group. One direction only: CloudTrail to CloudWatch. | `--cloud-watch-logs-log-group-arn`, `--cloud-watch-logs-role-arn` |
| SNS | A notification for every log file delivered | `--sns-topic-name` |

### Event types

| Type | Examples | Default |
|------|----------|---------|
| Management | Attaching a role policy, creating a user, setting up logging, routing rules | On, read and write |
| Data | S3 `GetObject`, `PutObject`, `DeleteObject`; every Lambda invocation; DynamoDB item calls | Off: high volume, charged |
| Insights | Unusual API call volume or error rates | Off |

### Commands from the labs

```bash
aws s3 mb s3://my-trail-bucket
aws s3api put-bucket-policy --bucket my-trail-bucket --policy file://bucket-policy.json
aws cloudtrail create-trail --name my-trail --s3-bucket-name my-trail-bucket
aws cloudtrail start-logging --name my-trail
aws logs create-log-group --log-group-name my-cloudtrail
aws cloudtrail update-trail --name my-trail \
  --cloud-watch-logs-log-group-arn "arn:aws:logs:ca-central-1:111122223333:log-group:my-cloudtrail:*" \
  --cloud-watch-logs-role-arn "arn:aws:iam::111122223333:role/my-cloudtrail-role"
```

| Policy | Must contain |
|--------|--------------|
| Bucket policy | Principal `cloudtrail.amazonaws.com`. `s3:GetBucketAcl` on the bucket. `s3:PutObject` on `bucket/AWSLogs/<account-id>/*`. |
| Role for CloudWatch Logs | Trusted by `cloudtrail.amazonaws.com`. `logs:CreateLogStream` and `logs:PutLogEvents` on `log-group:<name>:log-stream:<account-id>_CloudTrail_<region>*`. CloudTrail creates the stream itself. |

### Not in the video

| Topic | Fact |
|-------|------|
| Validating logs | With validation on, CloudTrail writes a signed digest file every hour. Check with `aws cloudtrail validate-logs`. |
| Alerting on an API call | Trail to CloudWatch Logs, then a metric filter, an alarm and SNS. Or an EventBridge rule that matches the API call. |
| Global services | IAM and CloudFront events are recorded in us-east-1. A multi-Region trail includes them. |

## Memory hooks

- CloudTrail trails people. CloudWatch watches performance. Config records what a resource looked like.
- Validation is a seal on the envelope: it shows the envelope was opened, it does not lock it.
- A CLI-made trail is a recorder that is not yet recording: press `start-logging`.

## Exam traps

- "Who terminated the instance last week?": CloudTrail Event history.
- "Keep audit logs for 7 years": a trail to S3, with a lifecycle rule to Glacier.
- "Prove the logs were not altered": enable log file validation.
- "Log who read or deleted objects in a bucket": turn on S3 data events on the trail.
- "One audit trail for all accounts": an organisation trail created in the management account.
- "Alarm when the root user signs in": send the trail to CloudWatch Logs and add a metric filter with an alarm.

## Outdated or wrong in the video

- **"Data events are only for S3 and Lambda."** They now cover many resource types, DynamoDB among them, as he notes himself in the lab. Very sure.
- **CloudWatch Logs "wasn't available when creating the trail".** The create-trail page now has a CloudWatch Logs section and can create the role for you. Very sure.
- **CloudTrail Lake is "a wrapper around Athena".** Lake has its own SQL engine. Lake query federation is an optional extra that lets Athena read the store. Very sure.
- **"We're locked in for the year."** The retention period is how long events are kept, not a contract. A store can be deleted once termination protection and federation are off; it then sits 7 days in pending deletion. Very sure.
- **CloudTrail Lake as "new".** AWS closed Lake to new customers on 31 May 2026 and points them to CloudWatch. Existing customers keep it. Very sure.

Related: [[FCC 25 - AWS Config]] · [[FCC 23.1 - CloudWatch Logs and Logs Insights]] · [[FCC 23.3 - EventBridge and CloudWatch alarms]] · [[FCC 21 - Identity and Access Management IAM]] · [[FCC 09.4 - S3 security and encryption]] · [[FCC 00 - Watch guide]]
