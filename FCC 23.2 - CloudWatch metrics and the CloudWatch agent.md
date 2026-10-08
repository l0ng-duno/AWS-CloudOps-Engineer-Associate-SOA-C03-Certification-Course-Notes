# FCC 23.2 - CloudWatch metrics and the CloudWatch agent

> Video section [46:57:20](https://youtu.be/5JTBS09e7ew?t=169040) to 50:31:25 (part 2 of 3 of the CloudWatch chapter) · 3h34m total · about 5 min lecture, 3h29m labs
> Fast path: read this note and skip almost all of the video. The 5-minute lecture at the start holds nearly every exam fact. The rest is one 3.5-hour lab, and about 11 minutes of it are worth watching: the agent configuration file ([49:10:30](https://youtu.be/5JTBS09e7ew?t=177030), 7 min) and the logs arriving in CloudWatch ([50:17:52](https://youtu.be/5JTBS09e7ew?t=181072), 4 min). He never shows agent metrics, the config wizard or Parameter Store; those are in "Not in the video".

## Summary

- EC2 sends metrics every 5 minutes (basic monitoring) or every 1 minute (detailed monitoring). Most other services send every 1 minute.
- EC2 only sees an instance from the outside. Memory, disk space, swap and log files need the CloudWatch agent inside the OS.
- Install the agent with Systems Manager Run Command: document `AWS-ConfigureAWSPackage`, action `Install`, name `AmazonCloudWatchAgent`. The instance role must allow it to write to CloudWatch.
- The agent is driven by one JSON file with three sections: `agent`, `metrics`, `logs`. Load it with `amazon-cloudwatch-agent-ctl -a fetch-config`.
- Your own numbers go in with `aws cloudwatch put-metric-data`. Only custom metrics can be high resolution (down to 1 second).

## Labs and demos

It is one lab, recorded in two sittings. The rows below are its phases.

| Time | Len | What it shows | Takeaway | Verdict |
|------|-----|---------------|----------|---------|
| [47:02:46](https://youtu.be/5JTBS09e7ew?t=169366) | 92m | Sitting 1. Builds a small Ruby on Rails API ("Shogun") that writes a log file, and a Ruby script that fires random requests at it. Launches an EC2 instance with CloudFormation and installs Ruby by hand through Session Manager. | No CloudWatch content at all. Nearly every minute is Rails trouble: database config, production mode not answering, gem folder permissions, forced HTTPS, no log file. Two points matter to an operator: the instance role needs `s3:GetObject` to pull the app zip, and the app must log to a file because the agent tails files. | Skip |
| [48:35:18](https://youtu.be/5JTBS09e7ew?t=174918) | 47m | Sitting 2. Writes SSM Command documents (one per install step) and an Automation runbook that chains them, in CloudFormation, by cutting down an old private template. Adds the agent's JSON config. | Mostly deleting steps he does not need. The useful parts: Command document versus runbook, `AWS-ConfigureAWSPackage`, sending step output to CloudWatch Logs, and the agent config file. | Skim: [49:10:30](https://youtu.be/5JTBS09e7ew?t=177030) to 49:17:38 (7 min) |
| [49:21:52](https://youtu.be/5JTBS09e7ew?t=177712) | 48m | Deploys the stacks and runs the runbook until every step passes. | Six failed CloudFormation deploys, then four failed runbook runs. He deletes and rebuilds the stack after each one. The lessons are in "Useful moments". | Skip |
| [50:09:38](https://youtu.be/5JTBS09e7ew?t=180578) | 12m | Runbook finishes and starts the agent. He starts the app, sends traffic and watches the log group with Live Tail. | Seven minutes go on a Rails secret-key error. Then it works: the agent created its own log group, named in the config, and the request lines stream in. | Skim: [50:17:52](https://youtu.be/5JTBS09e7ew?t=181072) to 50:21:56 (4 min) |
| [50:21:56](https://youtu.be/5JTBS09e7ew?t=181316) | 9m | Exports the log group to S3, then cleans up. | Export is refused until the bucket policy allows the CloudWatch Logs service. The first export holds one line because of the time range. Output is gzip. Log groups made by the agent and by Run Command survive stack deletion. | Skip |

### Useful moments inside the agent lab

| Time | What you learn |
|------|----------------|
| [47:55:55](https://youtu.be/5JTBS09e7ew?t=172555) | The instance cannot download the app from S3 until the role in the template gets `s3:GetObject` on the bucket |
| [48:33:05](https://youtu.be/5JTBS09e7ew?t=174785) | The app only produced log files after a Puma setting redirected stdout to `log/puma.stdout.log` |
| [48:39:30](https://youtu.be/5JTBS09e7ew?t=175170) | Run Command (Command documents) versus Automation (runbooks) in the Systems Manager console |
| [48:49:22](https://youtu.be/5JTBS09e7ew?t=175762) | `CloudWatchOutputConfig` on a step sends its output to a CloudWatch log group |
| [48:50:08](https://youtu.be/5JTBS09e7ew?t=175808) | Shape of a Command document: `aws:runShellScript`, every command wrapped in `sudo su - ec2-user -c "..."` |
| [48:52:09](https://youtu.be/5JTBS09e7ew?t=175929) | AWS-owned documents: `AWS-UpdateSSMAgent`, then `AWS-ConfigureAWSPackage`, which installs Distributor packages such as the agent |
| [49:08:10](https://youtu.be/5JTBS09e7ew?t=176890) | A step in his template times out after 60 seconds; he raises the slow one to 600 |
| [49:10:30](https://youtu.be/5JTBS09e7ew?t=177030) | The runbook step that copies `amazon-cloudwatch-agent.json` into place and starts the agent |
| [49:11:55](https://youtu.be/5JTBS09e7ew?t=177115) | `agent` and `logs` sections: 60-second interval, run as `ec2-user`, file path, log group name |
| [49:15:47](https://youtu.be/5JTBS09e7ew?t=177347) | `metrics` section: memory, swap, disk, CPU. The agent fills in the instance ID |
| [49:23:30](https://youtu.be/5JTBS09e7ew?t=177810) | Nested stack fails: `TemplateURL` must be an S3 URL. Fix: `aws cloudformation package`, then deploy the packaged template |
| [49:36:52](https://youtu.be/5JTBS09e7ew?t=178612) | Running a runbook: Automation, Execute automation, "Owned by me", simple execution, pick the instance |
| [49:38:45](https://youtu.be/5JTBS09e7ew?t=178725) | Instant failure: the instance ID parameter was `String`; the step needs `StringList` |
| [49:48:30](https://youtu.be/5JTBS09e7ew?t=179310) | No step output in CloudWatch Logs because the instance role had no CloudWatch permissions; he adds a second managed policy |
| [49:53:45](https://youtu.be/5JTBS09e7ew?t=179625) | With logs working, the stream shows the real error ("Could not locate Gemfile"). The Automation page showed nothing |
| [49:55:03](https://youtu.be/5JTBS09e7ew?t=179703) | Editing a document creates a new version; at [49:57:27](https://youtu.be/5JTBS09e7ew?t=179847) he runs it alone with Run Command (version, instance, output log group) |
| [49:59:20](https://youtu.be/5JTBS09e7ew?t=179960) | Debug method: open Session Manager and paste the exact command the document runs. The session user is `ssm-user` |
| [50:09:40](https://youtu.be/5JTBS09e7ew?t=180580) | Last step starts the agent with `amazon-cloudwatch-agent-ctl` and a config file; all steps pass |
| [50:19:15](https://youtu.be/5JTBS09e7ew?t=181155) | The log group he made in CloudFormation is unused; the agent created the one named in its config |
| [50:22:38](https://youtu.be/5JTBS09e7ew?t=181358) | Export to S3 denied until the bucket policy names the CloudWatch Logs service principal |
| [50:30:50](https://youtu.be/5JTBS09e7ew?t=181850) | The agent-created log group never expires and has to be deleted by hand |

## Lecture index

| Time | Topic |
|------|-------|
| [46:57:20](https://youtu.be/5JTBS09e7ew?t=169040) | How often services send metrics; EC2 basic versus detailed monitoring |
| [46:58:05](https://youtu.be/5JTBS09e7ew?t=169085) | Installing the agent with Run Command; the IAM role |
| [46:59:35](https://youtu.be/5JTBS09e7ew?t=169175) | Host-level metrics versus agent metrics |
| [47:01:03](https://youtu.be/5JTBS09e7ew?t=169263) | Custom metrics and high-resolution metrics |
| [47:01:53](https://youtu.be/5JTBS09e7ew?t=169313) | Log collection with the agent |

## Notes

### How often metrics arrive

| Source | Interval | Note |
|--------|----------|------|
| EC2, basic monitoring | 5 minutes | The default |
| EC2, detailed monitoring | 1 minute | Switched on per instance |
| Most other services | 1 minute | A few are 3 or 5 minutes |
| Custom metric, standard resolution | 1 minute | |
| Custom metric, high resolution | 1 second | Read back in periods of 1, 5, 10 or 30 seconds, or multiples of 60 |

He stresses that the "EC2 is 5 minutes, 1 with detailed monitoring" fact is asked in exams.

### Default EC2 metrics versus agent metrics

| No agent (seen from the hypervisor) | Needs the agent (seen from inside the OS) |
|-------------------------------------|-------------------------------------------|
| CPU utilisation | Memory utilisation |
| Network in and out | Disk space used |
| Disk read and write activity | Swap (Linux) and page file (Windows) |
| Status checks | Log files |

### Installing the agent

| Step | Detail |
|------|--------|
| 1. Role | Instance profile role with `AmazonSSMManagedInstanceCore` (so Systems Manager can reach it) and `CloudWatchAgentServerPolicy` (so the agent can send metrics and logs) |
| 2. Install | Run Command, document `AWS-ConfigureAWSPackage`, Action `Install`, Name `AmazonCloudWatchAgent`, Version `latest` |
| 3. Targets | Instances picked by hand, by tag, or by resource group |
| 4. Configure | Put the JSON config on the instance, load it, start the agent |
| 5. Change | Edit the config, then restart the agent |

```bash
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl \
  -a fetch-config -m ec2 -s -c file:/path/to/amazon-cloudwatch-agent.json

# -a fetch-config = load this config, -m ec2 = running on EC2, -s = restart the agent

# -a status / -a stop / -a start for the rest
```

### The agent configuration file

| Section | Holds | Keys he shows |
|---------|-------|---------------|
| `agent` | Global settings | `metrics_collection_interval` (60), `run_as_user` |
| `logs` | Files to tail and where to send them | `file_path`, `log_group_name`, `log_stream_name` |
| `metrics` | OS metrics to collect | `mem`, `swap`, `disk`, `cpu` |

The captions do not show his file. This is the same shape with the values he reads out:

```json
{
  "agent": { "metrics_collection_interval": 60, "run_as_user": "ec2-user" },
  "logs": { "logs_collected": { "files": { "collect_list": [
    { "file_path": "/home/ec2-user/app/log/puma.stdout.log",
      "log_group_name": "/examples/agent-app/puma.stdout.log",
      "log_stream_name": "{instance_id}" } ] } } },
  "metrics": {
    "append_dimensions": { "InstanceId": "${aws:InstanceId}" },
    "metrics_collected": {
      "mem":  { "measurement": ["mem_used_percent"] },
      "swap": { "measurement": ["swap_used_percent"] },
      "disk": { "measurement": ["used_percent", "inodes_free"], "resources": ["*"] } } }
}
```

- The agent creates the log group and stream if they are missing. A group it creates has no retention period.
- The log file does not need to exist when the agent starts. In the lab the agent started first and picked the file up once the app wrote it.

### Custom metrics

```bash
aws cloudwatch put-metric-data --namespace "MyApp" --metric-name QueueDepth \
  --unit Count --value 42 --dimensions Service=checkout
```

- Parts: namespace, metric name, unit, value, dimensions (name and value pairs that identify what is measured).
- Also available in every SDK. He says anyone doing SysOps should know this command.

### Run Command and Automation, as used in the lab

| | Command document | Automation runbook |
|---|------------------|--------------------|
| Runs | Shell commands inside an instance, through the SSM Agent | A list of steps, each of which can call a Command document |
| Start it from | Run Command | Automation, "Execute automation" |
| In the lab | One per install step, action `aws:runShellScript` | Chains them with `aws:runCommand` steps |
| What bit him | Commands needed `sudo su - ec2-user -c`; 60-second timeout | Instance IDs must be a `StringList` |

- A step's output reaches CloudWatch Logs only when `CloudWatchOutputConfig` is set and the instance role may write logs.
- One missing CloudWatch policy on the role blocks both the Run Command output and the agent itself.

### Exporting a log group to S3

- Log group, Actions, "Export data to Amazon S3". Choose the bucket and a time range.
- The bucket policy must allow the service principal `logs.<region>.amazonaws.com` to call `s3:GetBucketAcl` and `s3:PutObject`.
- Files arrive gzip-compressed. It is a batch export, not a live feed.

### Not in the video

These agent points are tested and the lab does not reach them.

| Topic | Fact |
|-------|------|
| Config wizard | `amazon-cloudwatch-agent-config-wizard` asks questions, writes `config.json`, and can save it to Parameter Store |
| Parameter Store | Keep one config as a parameter whose name starts with `AmazonCloudWatch-`. Load it with `-c ssm:<parameter-name>` or the `AmazonCloudWatch-ManageAgent` document |
| Two policies | `CloudWatchAgentServerPolicy`: run the agent and read the parameter. `CloudWatchAgentAdminPolicy`: also write the parameter; give it only to the instance where you build the config |
| Namespace | Agent metrics appear under `CWAgent`, not `AWS/EC2`. Typical names: `mem_used_percent`, `disk_used_percent` |
| High resolution | `put-metric-data --storage-resolution 1`. Alarms on such metrics can use 10- or 30-second periods |
| Metric retention | Under 1 minute: 3 hours. 1 minute: 15 days. 5 minutes: 63 days. 1 hour: 15 months |
| Nothing arrives | Read `/opt/aws/amazon-cloudwatch-agent/logs/amazon-cloudwatch-agent.log`, then check the role and the route to CloudWatch |
| On premises | The same agent runs on your own servers with `-m onPremise` |

## Memory hooks

- The hypervisor sees the box from outside: CPU, network, disk activity. How full the box is (memory, disk space) needs someone inside: the agent.
- EC2 is the odd one out: 5 minutes, or 1 with detailed monitoring.
- Three verbs, three tools: install with `AWS-ConfigureAWSPackage`, describe in the JSON file, load with `fetch-config`.
- "Package" installs the agent. "ManageAgent" configures it.

## Exam traps

- "Alarm on memory or free disk space of an EC2 instance": install the CloudWatch agent. These are not default metrics.
- "EC2 metrics are needed every minute": enable detailed monitoring. "Every second": publish a high-resolution custom metric.
- "Agent is installed but nothing reaches CloudWatch": the instance role lacks `CloudWatchAgentServerPolicy`.
- "Install and configure the agent on a whole fleet": Run Command by tag with `AWS-ConfigureAWSPackage`, config held in Parameter Store.
- "Send an application's log files from EC2 to CloudWatch Logs": the `logs` section of the agent config.
- "Track a business number such as orders per minute": `put-metric-data` custom metric.

## Outdated or wrong in the video

- **"The CloudWatch agent document in Run Command is only for restarting the agent."** `AmazonCloudWatch-ManageAgent` also configures it: it loads a config from Parameter Store and can start, stop or report status. Very sure.
- **"Attach the CloudWatchAgentServerRole."** That is only the name AWS suggests for a role you create yourself. The AWS managed policy to attach is `CloudWatchAgentServerPolicy`. Very sure.
- **"CPU and memory usage are not collected by default"** (said while reading the config). CPU utilisation is a default EC2 metric. Memory is not. Very sure.
- **Two status checks.** Nitro instances now show a third check, for attached EBS volumes. Fairly sure.
- **Security group open to the whole internet and system gem folders handed to `ec2-user`.** He says himself this is not for production. Very sure.

Related: [[FCC 23.1 - CloudWatch Logs and Logs Insights]] · [[FCC 23.3 - EventBridge and CloudWatch alarms]] · [[FCC 22 - AWS Systems Manager]] · [[FCC 02 - Amazon EC2]] · [[FCC 29 - AWS CloudFormation]] · [[FCC 00 - Watch guide]]
