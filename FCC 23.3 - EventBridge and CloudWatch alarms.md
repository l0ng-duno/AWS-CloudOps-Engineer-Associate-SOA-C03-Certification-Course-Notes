# FCC 23.3 - EventBridge and CloudWatch alarms

> Video section [50:31:25](https://youtu.be/5JTBS09e7ew?t=181885) to 50:57:43 (part 3 of 3 of the CloudWatch chapter) · 26 min total · 26 min lecture, no labs
> Fast path: read this note and skip the video; it is slides only. The alarm part is short and leaves out missing-data treatment, the EC2 recover action and how alarms feed EventBridge. All three are tested, so read "Not in the video" with care.

## Summary

- EventBridge (formerly CloudWatch Events) is a serverless event bus. Sources send JSON events, rules match them, targets receive them.
- A rule fires on an event pattern (something happened) or on a schedule (cron or rate, in UTC, one minute at the finest). A rule has up to 5 targets.
- Services that emit no events of their own can still be matched through CloudTrail, as `AWS API Call via CloudTrail`.
- A CloudWatch alarm compares one metric with a threshold. It has three states: `OK`, `ALARM`, `INSUFFICIENT_DATA`. A state change can notify SNS, scale an Auto Scaling group or act on an EC2 instance.
- Composite alarms combine other alarms to cut noise. Anomaly detection replaces the fixed line with a band.

## Labs and demos

There is no lab or console demo in this part. Everything is slides.

## Lecture index

| Time | Topic |
|------|-------|
| [50:31:25](https://youtu.be/5JTBS09e7ew?t=181885) | What an event bus is; EventBridge was CloudWatch Events |
| [50:32:11](https://youtu.be/5JTBS09e7ew?t=181931) | Core components: buses, producers, events, partner sources, rules, targets |
| [50:33:58](https://youtu.be/5JTBS09e7ew?t=182038) | Anatomy of an event |
| [50:35:13](https://youtu.be/5JTBS09e7ew?t=182113) | Schedule expressions: cron and rate |
| [50:36:18](https://youtu.be/5JTBS09e7ew?t=182178) | Rules and targets |
| [50:37:16](https://youtu.be/5JTBS09e7ew?t=182236) | Configure input and the input transformer |
| [50:38:40](https://youtu.be/5JTBS09e7ew?t=182320) | Schema registry and code bindings |
| [50:41:05](https://youtu.be/5JTBS09e7ew?t=182465) | Service integration through CloudTrail |
| [50:41:52](https://youtu.be/5JTBS09e7ew?t=182512) | Event patterns and matching operators |
| [50:43:54](https://youtu.be/5JTBS09e7ew?t=182634) | Partner event sources |
| [50:44:27](https://youtu.be/5JTBS09e7ew?t=182667) | CloudWatch alarms: states and actions |
| [50:45:16](https://youtu.be/5JTBS09e7ew?t=182716) | Anatomy of an alarm: period, evaluation periods, datapoints to alarm |
| [50:46:26](https://youtu.be/5JTBS09e7ew?t=182786) | Alarm conditions: static threshold and anomaly detection |
| [50:48:15](https://youtu.be/5JTBS09e7ew?t=182895) | Composite alarms |
| [50:49:16](https://youtu.be/5JTBS09e7ew?t=182956) | Dashboards |
| [50:49:38](https://youtu.be/5JTBS09e7ew?t=182978) | ServiceLens |
| [50:52:26](https://youtu.be/5JTBS09e7ew?t=183146) | Synthetics canaries and blueprints |
| [50:56:14](https://youtu.be/5JTBS09e7ew?t=183374) | Container Insights |
| [50:57:09](https://youtu.be/5JTBS09e7ew?t=183429) | Contributor Insights |

## Notes

### EventBridge building blocks

| Part | What it is | Remember |
|------|------------|----------|
| Event bus | Receives events and holds the rules | Every account has a default bus |
| Producer | An AWS service, your own app or a SaaS partner | AWS services emit events whether you use them or not |
| Event | A JSON object | |
| Rule | Decides which events go to which targets | Many per bus |
| Target | The AWS service that receives the event | Up to 5 per rule |

| Bus type | Holds |
|----------|-------|
| Default | Events from AWS services in your account |
| Custom | Your own applications' events; also a common place to collect events from other accounts |
| Partner (SaaS) | Events from a partner source such as Datadog, PagerDuty, Auth0, Segment or MongoDB |

Common targets he names: Lambda, SQS, SNS, Firehose, ECS tasks.

### Anatomy of an event

```json
{
  "version": "0",
  "id": "unique-id-for-this-event",
  "detail-type": "EC2 Instance State-change Notification",
  "source": "aws.ec2",
  "account": "123456789012",
  "time": "2026-01-01T12:00:00Z",
  "region": "us-east-1",
  "resources": ["arn:aws:ec2:us-east-1:123456789012:instance/i-0abc1234"],
  "detail": { "instance-id": "i-0abc1234", "state": "terminated" }
}
```

The top-level fields are always present. Only the contents of `detail` change from service to service.

### Event patterns

A pattern is a smaller JSON object with the same field names as the event. Values go in arrays. Fields you leave out match anything.

```json
{ "source": ["aws.ec2"],
  "detail-type": ["EC2 Instance State-change Notification"],
  "detail": { "state": ["terminated"] } }
```

| Operator | Matches |
|----------|---------|
| Prefix | Values that start with a given string |
| Anything-but | Everything except the listed values |
| Numeric | Number comparisons |
| IP address | An IPv4 or IPv6 range |
| Exists | A field being present or absent |
| Empty value | `""` for strings, `null` for anything else |

Operators can be combined in one pattern.

### Schedules

| | Rate expression | Cron expression |
|---|-----------------|-----------------|
| Example | `rate(15 minutes)` | `cron(15 12 ? * MON-FRI *)` = 12:15, Monday to Friday |
| Strength | Easy: a number and a unit (minutes, hours, days) | Exact times and days |

- Think of it as a serverless cron job.
- Schedules on rules run in UTC. The finest precision is 1 minute.

### What the target receives

| Configure input option | The target gets |
|------------------------|-----------------|
| Matched event | The whole event |
| Part of the matched event | One branch, for example `$.detail` |
| Constant (JSON text) | Fixed JSON that you type in |
| Input transformer | Values picked from the event by JSON path into variables, then placed in a template string or JSON |

### AWS services without their own events

- Use CloudTrail as the source. The rule matches `detail-type` = `AWS API Call via CloudTrail`.
- CloudTrail must be recording for these events to exist.

### Schema registry

- Stores the structure (schema) of events seen on a bus and tracks versions of it.
- Developers can download code bindings (typed classes) for a schema, for example through the AWS Toolkit for VS Code. Low exam weight.

### CloudWatch alarms

| State | Meaning |
|-------|---------|
| `OK` | The metric is within the threshold |
| `ALARM` | The metric has breached the threshold |
| `INSUFFICIENT_DATA` | Not enough data to decide |

| Setting | Meaning |
|---------|---------|
| Metric | The data being watched, for example `NetworkIn` |
| Threshold | The line the metric is compared with: greater, greater or equal, lower or equal, lower |
| Period | The length of time rolled up into one data point |
| Evaluation periods | How many of the most recent data points are examined (N) |
| Datapoints to alarm | How many of those must breach to enter `ALARM` (M) |

Example: period 5 minutes, evaluation periods 4, datapoints to alarm 3. The alarm fires when 3 of the last 4 five-minute points breach.

| Action on a state change | Example |
|--------------------------|---------|
| Notification | Publish to an SNS topic |
| Auto Scaling | Run a scaling policy |
| EC2 | Stop, terminate or reboot the instance |

| Threshold type | How it works | Use |
|----------------|--------------|-----|
| Static | One fixed number | Billing alarm: `EstimatedCharges` greater than 50 USD |
| Anomaly detection | A band around the expected value; the alarm fires when the metric leaves the band | A daily login spike that would be a false positive against a fixed line |

### Composite alarms

- An alarm whose rule is built from the states of other alarms.
- Typical setup: give the child alarms (say CPU and `NetworkIn`) no actions, and let only the composite alarm notify.
- Purpose: fewer, more meaningful notifications.

### Other CloudWatch features in this section

| Feature | One line |
|---------|----------|
| Dashboards | Pages of widgets that graph your metrics |
| ServiceLens | Metrics, logs, traces and alarms for a distributed application in one view, with a service map. Built on X-Ray: needs X-Ray instrumentation, the X-Ray daemon and the CloudWatch agent |
| Synthetics | Canaries: scripts on a schedule that behave like a user against a URL or API. They run on Lambda with a headless browser. Blueprints: heartbeat, API canary, broken link checker, GUI workflow |
| Container Insights | Metrics and logs for ECS, Fargate, EKS and Kubernetes on EC2. Filter by cluster, node, pod, task or service |
| Contributor Insights | Rules that rank the top contributors to a problem, in real time |

### Not in the video

| Topic | Fact |
|-------|------|
| Missing data | Set per alarm. `missing` (default): the point is not counted. `notBreaching`: counts as good. `breaching`: counts as bad. `ignore`: keep the current state |
| EC2 recover | A fourth EC2 action. Alarm on `StatusCheckFailed_System`, action recover: the instance moves to healthy hardware and keeps its ID and IP addresses |
| Testing | `aws cloudwatch set-alarm-state` forces a state so you can check that the action works |
| Billing alarm | Billing metrics exist only in us-east-1, and billing alerts must be enabled in the account first |
| Alarm to EventBridge | Every alarm state change is also an event (`CloudWatch Alarm State Change`). A rule can send it to any target, such as a Systems Manager Automation runbook |
| Target permissions | Lambda, SNS, SQS: a resource-based policy that allows `events.amazonaws.com`. Most other targets: an IAM role on the rule |
| Failed deliveries | EventBridge retries for up to 24 hours. Add an SQS dead-letter queue to a target to keep events that still fail |
| Dashboards | One dashboard can show metrics from several Regions and several accounts |

## Memory hooks

- EventBridge in one line: when X happens, or at time T, tell Y. Source, bus, rule, target.
- A pattern is a stencil laid over the event: same field names, only the fields you care about.
- Alarm arithmetic: the period makes a point, evaluation periods is N, datapoints to alarm is M. "M of the last N."
- A composite alarm is an alarm about alarms. A canary is a robot user.

## Exam traps

- "Run a Lambda function every night": EventBridge schedule. Remember the time is UTC.
- "React when an instance is terminated" or "when someone edits a security group": EventBridge rule with an event pattern. For API calls, match `AWS API Call via CloudTrail`.
- "Alarm fires on short harmless spikes": raise evaluation periods and use M of N. For a regular daily pattern, use anomaly detection.
- "One incident produces a flood of alarm notifications": composite alarm.
- "Alarm sits in INSUFFICIENT_DATA because the metric reports only now and then": set missing data to `notBreaching`.
- "Recover an instance automatically when the underlying host fails": alarm on `StatusCheckFailed_System` with the recover action.
- "Check a website or API from outside every few minutes": Synthetics canary. "Who are the top talkers?": Contributor Insights.

## Outdated or wrong in the video

- **"100 rules per bus."** The default quota is 300 rules per event bus and it can be raised. Very sure.
- **"The period is how often the alarm is checked, always 5 minutes I believe."** The period is the span of time rolled up into one data point, and you choose it: 10 or 30 seconds for high-resolution metrics, otherwise any multiple of 60 seconds. Very sure.
- **"A composite alarm can only trigger an SNS topic."** It can also invoke a Lambda function or create a Systems Manager OpsItem. It still cannot run EC2 or Auto Scaling actions. Very sure.
- **Alarm actions listed as SNS, Auto Scaling and EC2 only.** A metric alarm can now also invoke a Lambda function directly and create OpsItems. Very sure.
- **"All scheduled events use UTC."** True for schedules on rules. The newer EventBridge Scheduler supports time zones and one-time schedules, and AWS recommends it for new schedules. Very sure.
- **"Events over 256 KB are not supported."** EventBridge raised its maximum event size from 256 KB to 1 MB in January 2026. Fairly sure.
- **"A custom bus is for consuming events from another account."** A custom bus is mainly for your own applications' events. Any bus, the default one included, can receive events from other accounts once its resource policy allows it. Fairly sure.
- **Synthetics "uses Node.js and Puppeteer" with "four blueprints".** There are now Python (Selenium) and Playwright runtimes as well, and more blueprints such as the canary recorder and visual monitoring. Fairly sure.
- **ServiceLens.** The name has gone from the CloudWatch console. Its map lives on as the X-Ray trace map and the Application Signals application map. Fairly sure.

Related: [[FCC 23.1 - CloudWatch Logs and Logs Insights]] · [[FCC 23.2 - CloudWatch metrics and the CloudWatch agent]] · [[FCC 24 - AWS CloudTrail]] · [[FCC 32 - SNS]] · [[FCC 20 - ASG]] · [[FCC 00 - Watch guide]]
