# FCC 23.1 - CloudWatch Logs and Logs Insights

> Video section [44:03:12](https://youtu.be/5JTBS09e7ew?t=158592) to 46:57:20 (part 1 of 3 of the CloudWatch chapter) · 2h54m total · about 13 min lecture, 2h41m labs
> Fast path: read this note and skip almost all of the video. About two of the three hours are spent debugging bash, Python and Ruby scripts that only exist to upload fake log lines. If you watch one thing, watch the metric filter being created, [46:39:50](https://youtu.be/5JTBS09e7ew?t=167990) to 46:44:27. Logs Insights gets a 4-minute slide talk and no demo; its query syntax is under "Not in the video".

## Summary

- Hierarchy: a **log group** holds **log streams**, and a log stream holds **log events**. Settings (retention, encryption, filters) belong to the log group.
- Retention defaults to "Never expire". Set it per log group, from 1 day to 10 years. It is a separate call from creating the group.
- Log data is encrypted at rest by default. A KMS key of your own can be attached per log group.
- A **filter pattern** searches events. A **metric filter** uses a pattern to turn matching events into a CloudWatch metric that an alarm can watch. It only counts events that arrive after it is created.
- **Logs Insights** queries one or more log groups with its own piped query language. Fields it generates start with `@`.

## Labs and demos

| Time | Len | What it shows | Takeaway | Verdict |
|------|-----|---------------|----------|---------|
| [44:10:53](https://youtu.be/5JTBS09e7ew?t=159053) | 87m | Creates a log group, a 1-day retention policy and a log stream with the CLI, then tries to upload 100 fake web server log lines with `put-log-events` from a bash script | The three setup commands work in the first 10 minutes. The bash upload never works: about 50 minutes go on `date` rejecting the log's date format and then on JSON quoting. He gives up at [45:25:49](https://youtu.be/5JTBS09e7ew?t=163549) and uses a Ruby SDK script, which works at [45:36:02](https://youtu.be/5JTBS09e7ew?t=164162). His conclusion: time parsing and character escaping are the hard parts, and in real work the CloudWatch agent does this for you. | Skim: [44:15:46](https://youtu.be/5JTBS09e7ew?t=159346) to 44:18:48 (3 min) |
| [45:42:19](https://youtu.be/5JTBS09e7ew?t=164539) | 55m | Searches log events in the console with filter patterns, first on plain-text logs, then on JSON logs, then with `filter-log-events` in the CLI | `/login` finds nothing, because special characters need double quotes. `home login` is not "or"; two terms mean AND. `%home\|login%` works as a regular expression. About 34 minutes go on producing JSON logs and repairing the uploader. On JSON events `{ $.method = "PUT" }` works. | Skim: [45:45:15](https://youtu.be/5JTBS09e7ew?t=164715) to 45:49:48 (4 min) and [46:24:41](https://youtu.be/5JTBS09e7ew?t=167081) to 46:27:30 (3 min) |
| [46:38:10](https://youtu.be/5JTBS09e7ew?t=167890) | 19m | Creates a metric filter for the term `login`, then an alarm on the new metric (sum above 20 in 5 minutes) | The metric is missing at first: no matching events had arrived since the filter was created. After more uploads the namespace appears. "Test pattern" confirms the pattern matches. Days later the data points show on a 1-week graph and the alarm still reads Insufficient data. He never finds the reason for the delay. Five minutes in the middle are Python edits. | Skim: [46:39:50](https://youtu.be/5JTBS09e7ew?t=167990) to 46:44:27 (5 min) and [46:53:29](https://youtu.be/5JTBS09e7ew?t=168809) to 46:56:31 (3 min) |

### Useful moments inside the first lab (create, retain, upload)

| Time | What you learn |
|------|----------------|
| [44:13:25](https://youtu.be/5JTBS09e7ew?t=159205) | `aws logs create-log-group`; naming the group with forward slashes |
| [44:15:46](https://youtu.be/5JTBS09e7ew?t=159346) | Console form for a new log group: retention, log class (Standard or Infrequent Access), KMS key |
| [44:17:05](https://youtu.be/5JTBS09e7ew?t=159425) | Retention is not an option of `create-log-group`. It is a second call, `put-retention-policy` |
| [44:18:30](https://youtu.be/5JTBS09e7ew?t=159510) | `create-log-stream`, named with a Unix timestamp |
| [44:23:10](https://youtu.be/5JTBS09e7ew?t=159790) | He finds `put-log-events`. An event is a timestamp plus a message |
| [45:26:25](https://youtu.be/5JTBS09e7ew?t=163585) | 4-minute walk through the Ruby SDK version: create group and stream, parse the file, send one batch |
| [45:36:02](https://youtu.be/5JTBS09e7ew?t=164162) | Success. Events are visible at once. Each failed run left an empty stream behind |

### Useful moments inside the filter lab

| Time | What you learn |
|------|----------------|
| [45:45:15](https://youtu.be/5JTBS09e7ew?t=164715) | Log stream view: expand rows, copy, download as CSV (timestamp and message only) |
| [45:46:20](https://youtu.be/5JTBS09e7ew?t=164780) | The filter pattern syntax page: one term, several terms, optional terms, exact phrase |
| [45:49:05](https://youtu.be/5JTBS09e7ew?t=164945) | A regular expression goes between percent signs: `%home\|login%` |
| [46:07:10](https://youtu.be/5JTBS09e7ew?t=166030) | Error: events in one `PutLogEvents` call must be in chronological order |
| [46:17:40](https://youtu.be/5JTBS09e7ew?t=166660) | The upload reports success but the stream stays empty. Cause, found at [46:21:41](https://youtu.be/5JTBS09e7ew?t=166901): the timestamp was in seconds, not milliseconds |
| [46:24:41](https://youtu.be/5JTBS09e7ew?t=167081) | JSON events expand into fields. `{ $.ip = "..." }` and `{ $.method = "PUT" }` filter on them |
| [46:27:25](https://youtu.be/5JTBS09e7ew?t=167245) | `aws logs filter-log-events` with `--filter-pattern`, plus `--start-time` and `--end-time` in epoch milliseconds |

### Useful moments inside the metric filter lab

| Time | What you learn |
|------|----------------|
| [46:40:20](https://youtu.be/5JTBS09e7ew?t=168020) | Type a pattern in the log group's search box, then "Create metric filter". The form: filter name, metric name, metric value `1`, default value `0`, unit Count, namespace |
| [46:42:00](https://youtu.be/5JTBS09e7ew?t=168120) | The new metric cannot be found under All metrics |
| [46:42:55](https://youtu.be/5JTBS09e7ew?t=168175) | Alarm created straight from the metric filter: static threshold, Sum over 5 minutes, above 20, SNS action removed |
| [46:44:27](https://youtu.be/5JTBS09e7ew?t=168267) | Alarm state is Insufficient data. The next 5 minutes are Python edits; resume at 46:49:23 |
| [46:51:45](https://youtu.be/5JTBS09e7ew?t=168705) | The custom namespace appears, under "Metrics with no dimensions" |
| [46:54:00](https://youtu.be/5JTBS09e7ew?t=168840) | "Test pattern" on the metric filter, with pasted log lines, shows the matches |
| [46:55:00](https://youtu.be/5JTBS09e7ew?t=168900) | Days later: data points appear when the graph range is set to 1 week |

## Lecture index

| Time | Topic |
|------|-------|
| [44:03:12](https://youtu.be/5JTBS09e7ew?t=158592) | CloudWatch as an umbrella service and its nine parts |
| [44:04:55](https://youtu.be/5JTBS09e7ew?t=158695) | How the parts build on each other |
| [44:05:50](https://youtu.be/5JTBS09e7ew?t=158750) | Pillars of observability: metrics, logs, traces |
| [44:07:15](https://youtu.be/5JTBS09e7ew?t=158835) | CloudWatch Logs: integrations, encryption, retention |
| [44:08:55](https://youtu.be/5JTBS09e7ew?t=158935) | Log groups |
| [44:09:45](https://youtu.be/5JTBS09e7ew?t=158985) | Log streams |
| [45:37:52](https://youtu.be/5JTBS09e7ew?t=164272) | Log events and the simple filter box |
| [45:38:38](https://youtu.be/5JTBS09e7ew?t=164318) | Logs Insights |
| [45:40:23](https://youtu.be/5JTBS09e7ew?t=164423) | Discovered fields and the five system fields |
| [46:37:15](https://youtu.be/5JTBS09e7ew?t=167835) | Metrics in one minute (belongs to part 2) |

## Notes

### CloudWatch at a glance

| Part | What it does |
|------|--------------|
| Logs | Central store for log data from any source |
| Metrics | Time-ordered data points, such as `CPUUtilization` |
| Events, now EventBridge | Reacts to an event or a schedule |
| Alarms | Notify or act when a metric crosses a threshold |
| Dashboards | Graphs built from metrics |
| ServiceLens | One view of an application's health across metrics, logs and traces |
| Container Insights | Metrics and logs from containerised applications |
| Synthetics | Scripted tests ("canaries") that check a website or API on a schedule |
| Contributor Insights | The top contributors affecting performance, in real time |

- He says to know the first five well, and that the last four matter for the SysOps exam. That is the exam these notes are for, so learn the one-line definitions.
- Observability has three pillars that must be used together: metrics (numbers over time), logs (text records of what happened) and traces (the path of one request through several services). He adds alarms as an unofficial fourth.
- Metrics, briefly: a variable watched over time, grouped in a namespace per service. EC2 examples: `CPUUtilization`, `DiskReadOps`, `DiskWriteBytes`, `NetworkIn`, `NetworkOut`, `NetworkPacketsIn`. Part 2 covers them.

### Log group, log stream, log event

| Level | What it is | Typical name | What you set here |
|-------|-----------|--------------|-------------------|
| Log group | A collection of log streams | Path style, e.g. `/aws/lambda/my-function` | Retention, KMS key, log class, metric filters, subscription filters |
| Log stream | The sequence of events from one source | EC2: the instance ID. Lambda: date plus an ID. Glue: the job name | Nothing; the service or agent normally creates it |
| Log event | One record | n/a | A timestamp and a message |

| Log group setting | Detail |
|-------------------|--------|
| Retention | Default "Never expire". Choose 1 day up to 10 years (120 months). Stored logs cost money, so set it. |
| Encryption | Always encrypted at rest. Optionally attach your own KMS key. |
| Log class | Standard or Infrequent Access (cheaper, fewer features) |
| Integrations | Export to S3, stream to OpenSearch, receive CloudTrail and EventBridge events, query with Logs Insights |

```bash
aws logs create-log-group  --log-group-name "/example/basic/app"
aws logs put-retention-policy --log-group-name "/example/basic/app" --retention-in-days 1
aws logs create-log-stream --log-group-name "/example/basic/app" --log-stream-name "$(date +%s)"
aws logs put-log-events    --log-group-name "/example/basic/app" --log-stream-name "$STREAM" \
  --log-events '[{"timestamp":1717027200000,"message":"hello"}]'
aws logs filter-log-events --log-group-name "/example/basic/app" \
  --filter-pattern '{ $.method = "PUT" }' --start-time 1717027200000 --end-time 1717030800000
```

### What the upload labs teach about PutLogEvents

| Rule | Detail |
|------|--------|
| Event shape | `timestamp` in milliseconds since 1970 (UTC), plus `message` |
| Batching | `logEvents` is an array. Send many events per call, in chronological order. |
| Wrong timestamp | A value in seconds looks like a date in 1970. The call still returns success and the events are dropped as too old. |
| Speed | Accepted events are visible within seconds |

### Filter patterns

The same case-sensitive syntax is used by the console search box, `filter-log-events`, metric filters and subscription filters.

| Log type | Pattern | Matches |
|----------|---------|---------|
| Any text | `ERROR` | Events containing the term |
| Any text | `ERROR Timeout` | Events containing both terms (AND) |
| Any text | `?ERROR ?WARN` | Events containing either term (OR) |
| Any text | `"Internal Server Error"` | The exact phrase |
| Any text | `"/login"` | A term with a slash, dot or other special character must be quoted |
| Any text | `%home\|login%` | A regular expression |
| Space-delimited | `[ip, id, user, timestamp, request, status_code = 5*, size]` | Fields by position; you choose the names |
| JSON | `{ $.method = "PUT" }` | A field equal to a string |
| JSON | `{ ($.status >= 500) && ($.method = "PUT") }` | Numeric comparison and combined conditions |

### Metric filters

| Field in the form | His value | Meaning |
|-------------------|-----------|---------|
| Filter pattern | `login` | Which events count |
| Metric namespace | A custom name | Where the metric appears under All metrics |
| Metric name | Login attempts | What alarms and graphs refer to |
| Metric value | `1` | The number published for each matching event |
| Default value | `0` | Published when events arrive but none match, so the graph has no gaps |
| Unit | Count | Optional |

- Chain to remember: log events → metric filter → metric → alarm → notification.
- Not retroactive. Events already in the log group are never counted.
- The metric and its namespace only exist once the first data point is published.
- "Test pattern" checks a pattern against existing or pasted log lines.
- If no log events arrive at all, nothing is published, not even the default value. An alarm then shows Insufficient data.
- Alarms are billed while they exist. He deletes the alarm, the filter and the log groups at the end.

### Logs Insights

| Point | Detail |
|-------|--------|
| What it is | Interactive search and analysis of log data, more capable than the filter box |
| Why use it | Less work than exporting to S3 and querying with Athena |
| Language | Its own piped query language, not SQL. The console offers sample queries and lets you save your own. |
| Output | A result table and a bar graph over time |
| Limits he gives | 20 log groups per query, 15-minute timeout, results kept 7 days (the first two have been raised) |
| Fields | It discovers fields in each event. JSON keys become fields. Use `parse` to extract fields from other formats. |

| System field | Content |
|--------------|---------|
| `@message` | The raw, unparsed log event |
| `@timestamp` | The time recorded in the event |
| `@ingestionTime` | When CloudWatch Logs received the event |
| `@logStream` | The log stream name |
| `@log` | `account-id:log-group-name` |

Extra fields are discovered automatically for VPC Flow Logs, Route 53 logs, Lambda logs, CloudTrail logs and any JSON log.

### Not in the video

These points are tested but not covered in this part.

| Topic | Remember |
|-------|----------|
| Logs Insights commands | `fields`, `filter`, `stats ... by`, `sort`, `limit`, `parse`, `display`, joined with pipes |
| Subscription filter | Sends matching events in near real time to Kinesis Data Streams, Data Firehose, Lambda or OpenSearch Service. Cross-account delivery goes through a Logs "destination". |
| Export to S3 | `create-export-task` is a batch job. Data can take up to 12 hours to be exportable, so it is not real time. For continuous delivery to S3 use a subscription filter to Firehose. |
| Your own KMS key | `associate-kms-key` on the log group. The key policy must allow the CloudWatch Logs service in that Region to use the key. |
| Logs from EC2 | Nothing is sent by default. Install the CloudWatch agent and give the instance role permission to write logs (part 2). |
| Live tail | `aws logs tail <group> --follow`, or Live Tail in the console |
| Infrequent Access class | No metric filters and no subscription filters. The class cannot be changed after the group is created. |

```text
fields @timestamp, @message
| filter @message like /ERROR/
| sort @timestamp desc
| limit 20
```

```text
filter @message like /ERROR/
| stats count(*) as errors by bin(5m)
```

## Memory hooks

- Filing cabinet, folder, sheet of paper: log group, log stream, log event. The rules are written on the cabinet.
- Three readers of a log group: a metric filter makes a **number** (for an alarm), a subscription filter makes a **stream** (for another service), Logs Insights makes an **answer** (for a person).
- `@` means "Insights made this field".
- Log timestamps are always in milliseconds.

## Exam traps

- "Log storage cost keeps growing": the default retention is Never expire. Set a retention period on the log group.
- "Alarm when ERROR appears in an application log" or "count 404 responses": metric filter, then an alarm on the metric, then SNS.
- "The metric filter did not count last week's errors": expected. Filters only apply to new events.
- "Deliver logs in near real time to OpenSearch, Lambda or a central account": subscription filter, not export to S3.
- "Find the top talkers, or search several log groups ad hoc": Logs Insights.
- "Memory or disk usage of an EC2 instance is missing": not a default metric; it needs the CloudWatch agent (part 2).

## Outdated or wrong in the video

- **"All CloudWatch services are built off CloudWatch Logs" and metrics "leverage logs".** Metrics are a separate data store. Services such as EC2 publish metrics directly. Only metric filters and the embedded metric format create metrics from logs. Very sure.
- **"Contributor Insights does not use logs."** It works by analysing log groups with rules. Very sure.
- **Sequence tokens for `put-log-events`.** Since January 2023 the token is ignored and no longer needed. Very sure.
- **"A message is no larger than 256 KB."** The limit per log event was raised to 1 MB in April 2025. Very sure.
- **Logs Insights: "20 log groups, queries time out after 15 minutes."** Now 50 log groups per query and 60 minutes. Results are still kept for 7 days. Very sure.
- **"Stream logs to Elasticsearch Service" and "customer master key".** The names are now OpenSearch Service and KMS key. Very sure.
- **The console filter box is "finicky and not very reliable".** Most of what he saw follows the syntax rules: `/login` needs quotes, and two bare terms mean AND. Very sure.
- **A metric from a metric filter can take hours or days to show up.** Normally the first data point appears within a few minutes of the first matching event that arrives after the filter exists. The video does not establish why his was slow; the made-up timestamps in his generated logs are the likely cause. Fairly sure.

Related: [[FCC 23.2 - CloudWatch metrics and the CloudWatch agent]] · [[FCC 23.3 - EventBridge and CloudWatch alarms]] · [[FCC 24 - AWS CloudTrail]] · [[FCC 15.2 - VPC gateways, IPv6, endpoints and flow logs]] · [[FCC 34 - OpenSearch Service]] · [[FCC 00 - Watch guide]]
