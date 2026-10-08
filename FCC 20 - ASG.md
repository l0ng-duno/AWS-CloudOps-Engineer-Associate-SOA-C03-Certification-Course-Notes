# FCC 20 - ASG

> Video chapter [40:31:14](https://youtu.be/5JTBS09e7ew?t=145874) to 41:25:03 · 54 min total · about 26 min lecture, 28 min lab
> Fast path: read this note, then watch 3 minutes of the lab from [40:59:36](https://youtu.be/5JTBS09e7ew?t=147576) to 41:02:40. It shows that a new launch template version does not touch the running instance, and how the group replaces an instance he terminates. Scaling policies are only slides; there is no scaling lab.

## Summary

- An Auto Scaling group (ASG) keeps the number of EC2 instances between **min** and **max**, aiming at **desired capacity**. It launches them from a **launch template**.
- It replaces unhealthy instances. Health check type is **EC2** (status checks only) or **ELB** (also the load balancer's target health).
- Dynamic scaling policies: **target tracking** (first choice), **step** and **simple**. All are driven by CloudWatch alarms. **Predictive** scaling forecasts from history.
- A new launch template version only affects instances launched later. Roll it out with an **instance refresh**.
- Operations tools: lifecycle hooks, standby, suspend processes, scale-in protection, and `describe-scaling-activities` to see why a launch failed.

## Labs and demos

| Time | Len | What it shows | Takeaway | Verdict |
|------|-----|---------------|----------|---------|
| [40:36:09](https://youtu.be/5JTBS09e7ew?t=146169) | 28m | Builds a launch template and an ASG (min 1, max 1, desired 1) with CloudFormation, with no load balancer. Then adds user data as version 2 of the template, terminates the instance and watches the group replace it. | The first 18 minutes are template writing and six failed attempts (script and parameter slips, then three stack failures on the launch template's network interface). After that: the instance is "healthy" though no web server runs, the template update leaves the old instance alone, and a terminated instance is replaced within a couple of minutes. | Skim |

### Useful moments inside the ASG lab

| Time | What you learn |
|------|----------------|
| [40:37:52](https://youtu.be/5JTBS09e7ew?t=146272) | Launch templates replace launch configurations |
| [40:46:21](https://youtu.be/5JTBS09e7ew?t=146781) | The ASG resource: launch template and version, min and max size, list of subnets in `VPCZoneIdentifier` |
| [40:53:30](https://youtu.be/5JTBS09e7ew?t=147210) | Fix for "when a network interface is provided, the security groups must be part of it": put the security group inside the network interface only, not at the top level as well |
| [40:54:31](https://youtu.be/5JTBS09e7ew?t=147271) | Console tour: launch template versions, the group's desired, min and max capacity |
| [40:56:32](https://youtu.be/5JTBS09e7ew?t=147392) | The instance is healthy but serves no page. The EC2 health check does not test the application; a load balancer health check would. |
| [40:58:34](https://youtu.be/5JTBS09e7ew?t=147514) | Deploying the change modifies the group in place and creates launch template version 2 |
| [40:59:36](https://youtu.be/5JTBS09e7ew?t=147576) | The running instance is still version 1. New versions apply only to new instances. |
| [41:00:38](https://youtu.be/5JTBS09e7ew?t=147638) | He terminates the instance. The group marks it unhealthy and launches a replacement from version 2. |

## Lecture index

| Time | Topic |
|------|-------|
| [40:31:17](https://youtu.be/5JTBS09e7ew?t=145877) | What an ASG is, ways to scale, architecture diagram |
| [40:33:38](https://youtu.be/5JTBS09e7ew?t=146018) | Capacity settings: min, max, desired |
| [40:34:52](https://youtu.be/5JTBS09e7ew?t=146092) | Health check replacement: EC2 and ELB |
| [41:04:11](https://youtu.be/5JTBS09e7ew?t=147851) | ELB integration |
| [41:04:58](https://youtu.be/5JTBS09e7ew?t=147898) | Dynamic scaling policies and adjustment types |
| [41:06:06](https://youtu.be/5JTBS09e7ew?t=147966) | Simple scaling |
| [41:07:52](https://youtu.be/5JTBS09e7ew?t=148072) | Step scaling |
| [41:09:01](https://youtu.be/5JTBS09e7ew?t=148141) | Target tracking |
| [41:10:15](https://youtu.be/5JTBS09e7ew?t=148215) | Predictive scaling |
| [41:11:31](https://youtu.be/5JTBS09e7ew?t=148291) | Termination policies |
| [41:12:25](https://youtu.be/5JTBS09e7ew?t=148345) | Instance refresh |
| [41:13:38](https://youtu.be/5JTBS09e7ew?t=148418) | Scale-in protection |
| [41:14:15](https://youtu.be/5JTBS09e7ew?t=148455) | Launch templates and launch configurations |
| [41:16:15](https://youtu.be/5JTBS09e7ew?t=148575) | Lifecycle hooks |
| [41:18:52](https://youtu.be/5JTBS09e7ew?t=148732) | Warm pools |
| [41:20:10](https://youtu.be/5JTBS09e7ew?t=148810) | Attaching and detaching instances |
| [41:21:35](https://youtu.be/5JTBS09e7ew?t=148895) | Standby |
| [41:22:38](https://youtu.be/5JTBS09e7ew?t=148958) | Suspending processes |
| [41:23:32](https://youtu.be/5JTBS09e7ew?t=149012) | Troubleshooting launch failures |

## Notes

### Capacity

| Setting | Meaning |
|---------|---------|
| Min size | Never fewer than this. The group always launches instances to reach it. |
| Max size | Never more than this |
| Desired capacity | The number the group tries to hold right now. Scaling policies move it between min and max. |

```bash
aws autoscaling update-auto-scaling-group --auto-scaling-group-name my-asg --min-size 1 --max-size 4
aws autoscaling set-desired-capacity --auto-scaling-group-name my-asg --desired-capacity 2
```

- Changing these numbers by hand is **manual scaling**.
- Min = max = 1 gives a self-healing single instance. There is a short outage while the replacement boots.
- ASGs scale EC2 only, including the EC2 instances under ECS and EKS. Fargate does not use them.

### Health checks

| Type | Instance is unhealthy when | Note |
|------|----------------------------|------|
| EC2 | An EC2 status check fails, or the instance is stopped or terminated | Says nothing about the application |
| ELB | The above, or the load balancer's health check on the target fails | The check (path, port, success code) is configured on the target group. The ASG only chooses the type. |

- An unhealthy instance is terminated and a new one is launched.
- A load balancer is attached through its **target group**, not directly. Only a Classic Load Balancer is attached by name.

```bash
aws autoscaling attach-load-balancer-target-groups --auto-scaling-group-name my-asg \
  --target-group-arns <target-group-arn>
```

### Scaling policies

| Policy | How it works | CloudWatch alarms |
|--------|--------------|-------------------|
| Target tracking | Holds a metric at a target value, e.g. average CPU at 50% | Creates and manages them for you |
| Step | Adjusts by different amounts depending on how far the metric is past the threshold | You create them |
| Simple | One fixed adjustment per alarm, then waits for the cooldown | You create them. AWS recommends step or target tracking instead. |
| Predictive | Forecasts load from history and scales ahead of it | None |

| Adjustment type (simple and step) | Example |
|-----------------------------------|---------|
| `ChangeInCapacity` | Add 2, or remove 1 (`-1`) |
| `ExactCapacity` | Set capacity to exactly 4 |
| `PercentChangeInCapacity` | Add 30% |

- **Scale out** adds instances. **Scale in** removes them. Simple and step need one policy and one alarm for each direction.
- Step example from the slide: 0 to 15 above the threshold adds 10%, 15 to 25 adds 20%, above 25 adds 30%.
- Target tracking metrics: `ASGAverageCPUUtilization`, `ASGAverageNetworkIn`, `ASGAverageNetworkOut`, `ALBRequestCountPerTarget`, or a custom metric.

```bash
aws autoscaling put-scaling-policy --auto-scaling-group-name my-asg --policy-name cpu50 \
  --policy-type TargetTrackingScaling --target-tracking-configuration file://config.json
```

```json
{ "PredefinedMetricSpecification": { "PredefinedMetricType": "ASGAverageCPUUtilization" },
  "TargetValue": 50.0 }
```

| Predictive scaling | Number |
|--------------------|--------|
| History needed before it can start | 24 hours |
| History it uses | Last 14 days |
| Forecast | Next 48 hours, hour by hour |
| Forecast refreshed | Every 6 hours |
| Modes | `ForecastOnly` (look first), then `ForecastAndScale` |

### Launch templates

- Hold the AMI, instance type, key pair, security groups, user data, network interfaces and block devices. The AMI can be an SSM parameter instead of a fixed ID.
- **Versioned.** The group uses a specific version, `$Latest` or `$Default`.
- A new version changes nothing on running instances.
- Launch configurations are the legacy equivalent: no versions, cannot be edited.

### Managing instances in the group

| Feature | What it does | Key facts |
|---------|--------------|-----------|
| Instance refresh | Rolling replacement after a configuration change | `start-instance-refresh`. Preferences: minimum healthy percentage, instance warm-up, checkpoints (pause at a percentage), skip matching, auto rollback. |
| Termination policy | Chooses which instance goes first on scale-in | `Default`, `OldestInstance`, `NewestInstance`, `OldestLaunchTemplate`, `OldestLaunchConfiguration`, `ClosestToNextInstanceHour`, `AllocationStrategy`, or a Lambda function |
| Scale-in protection | Scale-in will not choose a protected instance | Off by default. Can be changed at any time. |
| Warm pool | Pre-initialised instances kept ready for slow-booting applications | Kept `Stopped`, `Running` or `Hibernated`. Size = max prepared capacity (or max size) minus desired capacity. |
| Detach | Removes an instance from the group; it keeps running on its own | Add `--should-decrement-desired-capacity`, or a replacement is launched |
| Attach | Adds an existing instance | It must be running, its AMI must still exist, it must not be in another group, and it must be in one of the group's zones |
| Standby | Takes an in-service instance out of rotation to troubleshoot or patch, then returns it | Still a member of the group, gets no load balancer traffic. `enter-standby`, `exit-standby`. |
| Suspend processes | Stops the group doing chosen things while you investigate | `Launch`, `Terminate`, `HealthCheck`, `ReplaceUnhealthy`, `AZRebalance`, `AlarmNotification`, `ScheduledActions`, `AddToLoadBalancer`, `InstanceRefresh`. `suspend-processes`, `resume-processes`. |

### Lifecycle hooks

```text
scale out:  Pending → Pending:Wait → Pending:Proceed → InService
scale in:   Terminating → Terminating:Wait → Terminating:Proceed → Terminated
```

| Item | Detail |
|------|--------|
| Hook on launch | `autoscaling:EC2_INSTANCE_LAUNCHING`: install or configure software before traffic arrives |
| Hook on terminate | `autoscaling:EC2_INSTANCE_TERMINATING`: copy logs off before the instance goes |
| The wait ends when | The heartbeat timeout expires (default 1 hour), or something calls `complete-lifecycle-action` |
| Who does the work | An SNS topic or SQS queue (notification target plus IAM role), an EventBridge rule that invokes Lambda, or a user data script on the instance |

### Troubleshooting launch failures

```bash
aws autoscaling describe-scaling-activities --auto-scaling-group-name my-asg
```

| Area | Causes he lists |
|------|-----------------|
| Launch settings | Key pair or security group does not exist, invalid block device mapping, instance type not offered in that zone, placement group not valid for the instance type |
| AMI | ID does not exist, AMI still pending, architecture does not match the instance type |
| Load balancer | Not found, or not in the same VPC as the instances |
| Capacity | Maximum number of instances already running |

### Not in the video

| Topic | Fact |
|-------|------|
| Scheduled scaling | Changes min, max or desired at set times (one-off or cron). For load you can predict by the clock. |
| Default health check type | EC2, even with a load balancer attached. You must set ELB yourself. |
| Health check grace period | Time after launch before health checks count. 300 s by default in the console, 0 from the CLI. Too short and instances are killed while still booting. |
| Cooldown | 300 s by default. Applies to simple scaling. Step and target tracking use instance warm-up instead. |
| Default termination policy | First the zone with the most instances, then the oldest launch template or configuration, then the instance closest to its next billing hour |

## Memory hooks

- Min is the floor, max is the ceiling, desired is where you stand.
- Target tracking is a thermostat: "keep it at 50". Step is "the hotter it gets, the more fans". Simple is one fan, then a rest.
- Scale **out** = more boxes side by side. Scale **in** = fewer.
- Detach = leaves the team. Standby = on the bench, still on the team.

## Exam traps

- "Instances are launched and terminated again in a loop": the health check grace period is shorter than the boot time, or the ELB health check fails.
- "The application crashed but the instance is not replaced": health check type is EC2. Change it to ELB.
- "The launch template was updated but instances still run the old AMI": existing instances are not changed. Start an instance refresh.
- "Copy logs to S3 before an instance is terminated": lifecycle hook on terminating.
- "Investigate one instance without it receiving traffic or being replaced": put it in standby.
- "Keep average CPU near 50% with the least effort": target tracking. "Load rises every weekday at 09:00": scheduled scaling.

## Outdated or wrong in the video

- **Scale in and scale out are swapped on the simple scaling slide.** He calls removing an instance "scaling out". Scale out adds, scale in removes. Very sure.
- **"The subnets have to be the same as the load balancer's."** Only the Availability Zones must match. The usual design is a load balancer in public subnets and instances in private subnets of the same zones. Very sure.
- **"Attaching a load balancer means the group uses its health checks instead of EC2's."** The type stays EC2 until you set it to ELB, and then both kinds of check count. Very sure.
- **"Scale-in protection is just termination protection."** It only stops scale-in from choosing that instance. It does not stop manual termination or replacement of an unhealthy instance. Very sure.
- **"Creating a launch template with the same name pushes a new version."** The CLI call fails for an existing name; use `create-launch-template-version`. A CloudFormation update does create a new version. Very sure.
- **"Scaling activities are not in the console."** They are on the group's Activity tab, under Activity history. Very sure.
- **Launch configurations.** His diagram still uses one. Accounts created on or after 1 October 2024 cannot create them at all. Very sure.

Related: [[FCC 19 - ELB]] · [[FCC 02 - Amazon EC2]] · [[FCC 03 - Amazon Machine Images AMIs]] · [[FCC 23.3 - EventBridge and CloudWatch alarms]] · [[FCC 29 - AWS CloudFormation]] · [[FCC 00 - Watch guide]]
