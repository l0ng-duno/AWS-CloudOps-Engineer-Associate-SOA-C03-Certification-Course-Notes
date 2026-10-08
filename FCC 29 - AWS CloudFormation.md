# FCC 29 - AWS CloudFormation

> Video chapter [57:56:14](https://youtu.be/5JTBS09e7ew?t=208574) to 60:40:12 · 2h44m total · about 45 min lecture, 2h labs
> Fast path: read this note. Watch the retain-on-delete lab (4 min) and two 2-minute clips: reading a change set at [58:23:00](https://youtu.be/5JTBS09e7ew?t=210180) and the stack policy denial at [59:58:49](https://youtu.be/5JTBS09e7ew?t=215929). Most lab time is syntax errors. The video skips StackSets, termination protection, dynamic references and `continue-update-rollback`; see "Not in the video".

## Summary

- A template (YAML or JSON) is deployed as a stack. Only `Resources` is required. AWS keeps the state, so there is no state file to lose.
- Update directly or through a change set (a preview). Each change is no interruption, some interruption, or replacement with a new physical ID.
- A failed create or update rolls back by default. `UPDATE_ROLLBACK_FAILED` and `DELETE_FAILED` are the stuck states to recognise.
- Protection: `DeletionPolicy` and `UpdateReplacePolicy` keep a resource or its data; a stack policy blocks updates to chosen resources, but not stack deletion.
- Drift detection reports resources changed or deleted outside CloudFormation. `cfn-init` configures an instance and `cfn-signal` tells the stack it is ready.

## Labs and demos

| Time | Len | What it shows | Takeaway | Verdict |
|------|-----|---------------|----------|---------|
| [58:05:55](https://youtu.be/5JTBS09e7ew?t=209155) | 7m | Launches the RDS Quick Start | The link opens the create-stack page with a template loaded and a long parameter form. Submit fails at once on two empty parameters. He gives up. | Skip |
| [58:17:00](https://youtu.be/5JTBS09e7ew?t=209820) | 9m | Stack updates through change sets, on a bucket and an EC2 instance | Read two columns: Action (Add, Modify, Remove) and Replacement (True, False, Conditional). Adding tags: Modify, False. Changing instance type: Modify, Conditional. Without `--no-execute-changeset` the update runs with no review. | Skim: [58:23:00](https://youtu.be/5JTBS09e7ew?t=210180), 2 min |
| [58:28:39](https://youtu.be/5JTBS09e7ew?t=210519) | 10m | Nested stack: child template in S3, parent uses `AWS::CloudFormation::Stack` | Fails twice: `s3://...` is rejected; `TemplateURL` must be the `https://` S3 address. A private bucket in the same account needs no bucket policy. Deleting the parent deletes the child. | Skip |
| [58:41:25](https://youtu.be/5JTBS09e7ew?t=211285) | 8m | Drift detection on a load balancer stack | Edited load balancer attributes and an added listener rule are not detected. A changed listener port shows `MODIFIED` after a delay. A terminated instance shows `DELETED` only after detection is run on that resource. | Skip |
| [58:54:15](https://youtu.be/5JTBS09e7ew?t=212055) | 4m | Pseudo parameters as bucket tags | `!Ref AWS::Region`, `AWS::StackName` and `AWS::AccountId` resolve with no declaration. | Skip |
| [59:05:35](https://youtu.be/5JTBS09e7ew?t=212735) | 18m | `Ref`, `GetAtt`, `Join` and `Sub` in outputs | About 6 minutes lost on `Sub`: a `!GetAtt` typed inside the string is printed literally. The two-argument form fixes it (see below). An output must be a string, not a list. | Skip |
| [59:25:40](https://youtu.be/5JTBS09e7ew?t=213940) | 10m | IaC generator: scan the account, build a template from an existing instance, import it into a stack | One deletion policy applies to every resource, so he edits the template to `Retain` the VPC and subnet. Delete hits `DELETE_FAILED` on the attached root volume; CloudTrail has the clearer error; a second delete works. | Skip |
| [59:43:31](https://youtu.be/5JTBS09e7ew?t=215011) | 9m | First template, in JSON, uploaded in the console, then converted to YAML | Three errors: `BucketName` not under `Properties`, `Type` inside `Properties`, misspelt `AWSTemplateFormatVersion`. The beginner lab, placed out of order. | Skip |
| [59:53:42](https://youtu.be/5JTBS09e7ew?t=215622) | 7m | Stack policy that denies updates to an instance | `deploy` has no stack policy option, so he runs `set-stack-policy` afterwards. The change set is still created; executing it fails with "Action denied by stack policy". Deleting the stack still works. | Skim: [59:58:49](https://youtu.be/5JTBS09e7ew?t=215929), 2 min |
| [60:01:03](https://youtu.be/5JTBS09e7ew?t=216063) | 6m | `DependsOn` forces creation order of EC2 instances | `DependsOn` sits beside `Type`, not under `Properties`. Events confirm the forced order. One failure: a t4g (ARM) type with an x86 AMI. | Skip |
| [60:07:16](https://youtu.be/5JTBS09e7ew?t=216436) | 9m | `CreationPolicy` on an Auto Scaling group | Adapts a wait-condition template from a lab that is not in this video. One rollback: `Timeout` must be ISO 8601 (`PT5M`), not `300`. `cfn-signal --resource` names the group. | Skip |
| [60:15:49](https://youtu.be/5JTBS09e7ew?t=216949) | 11m | Cross-stack reference: bucket stack exports its name, main stack imports it into a tag | Deploy the exporter first; delete the importer first. Two short-form functions cannot follow each other, so write `Fn::ImportValue: !Sub ...`. The ARM/x86 mismatch again. | Skip |
| [60:27:13](https://youtu.be/5JTBS09e7ew?t=217633) | 4m | Stack delete fails on a non-empty bucket, then succeeds with "retain resources" | `DELETE_FAILED`. Deleting again offers a tick box to retain the failing resource. The stack goes; the bucket stays for you to clean up. | **Watch** |
| [60:30:46](https://youtu.be/5JTBS09e7ew?t=217846) | 4m | Imports an existing bucket into a new stack from the console | Create stack, "With existing resources": template, bucket name as identifier, execute the import change set. The resource needs a `DeletionPolicy` and its template properties must match reality. | Skip |
| [60:34:20](https://youtu.be/5JTBS09e7ew?t=218060) | 6m | Application Composer | Paste a template and see it as a diagram. A visualiser only. | Skip |

## Lecture index

| Time | Topic |
|------|-------|
| [57:56:16](https://youtu.be/5JTBS09e7ew?t=208576) | Infrastructure as code, stacks, JSON and YAML |
| [57:59:10](https://youtu.be/5JTBS09e7ew?t=208750) | Template anatomy (told twice) |
| [58:04:37](https://youtu.be/5JTBS09e7ew?t=209077) | Quick Starts |
| [58:12:44](https://youtu.be/5JTBS09e7ew?t=209564) | Stack status codes |
| [58:15:24](https://youtu.be/5JTBS09e7ew?t=209724) | Stack updates: direct or change set, three update behaviours |
| [58:26:07](https://youtu.be/5JTBS09e7ew?t=210367) | Stack policies |
| [58:27:21](https://youtu.be/5JTBS09e7ew?t=210441) | Nested stacks |
| [58:38:18](https://youtu.be/5JTBS09e7ew?t=211098) | Rollbacks |
| [58:39:45](https://youtu.be/5JTBS09e7ew?t=211185) | Drift detection |
| [58:48:57](https://youtu.be/5JTBS09e7ew?t=211737) | Change sets |
| [58:49:50](https://youtu.be/5JTBS09e7ew?t=211790) | CDK |
| [58:53:13](https://youtu.be/5JTBS09e7ew?t=211993) | Pseudo parameters |
| [58:58:27](https://youtu.be/5JTBS09e7ew?t=212307) | Resource attributes |
| [59:01:15](https://youtu.be/5JTBS09e7ew?t=212475) | Intrinsic functions |
| [59:23:29](https://youtu.be/5JTBS09e7ew?t=213809) | Wait conditions |
| [59:24:49](https://youtu.be/5JTBS09e7ew?t=213889) | IaC generator |
| [59:35:39](https://youtu.be/5JTBS09e7ew?t=214539) | Helper scripts |
| [59:37:15](https://youtu.be/5JTBS09e7ew?t=214635) | `cfn-init` |
| [59:38:32](https://youtu.be/5JTBS09e7ew?t=214712) | `cfn-hup` |
| [59:41:02](https://youtu.be/5JTBS09e7ew?t=214862) | Custom resources |
| [59:42:15](https://youtu.be/5JTBS09e7ew?t=214935) | CloudFormation registry |
| [59:52:50](https://youtu.be/5JTBS09e7ew?t=215570) | Preventing stack updates |

## Notes

### Template anatomy

| Section | Purpose |
|---------|---------|
| `AWSTemplateFormatVersion` | Always `2010-09-09` |
| `Description` | Free text |
| `Metadata` | Extra information about the template |
| `Parameters` | Inputs at deploy time, with `Type`, `Default`, `AllowedValues` |
| `Mappings` | Lookup table, e.g. Region to AMI ID, read with `Fn::FindInMap` |
| `Conditions` | If/else: decide whether a resource is created or a property set |
| `Transform` | Macros that rewrite the template, e.g. SAM |
| `Resources` | The only required section; at least one resource |
| `Outputs` | Values shown after deployment; can be exported to other stacks |

```yaml
Parameters:
  InstanceType: { Type: String, Default: t3.micro, AllowedValues: [t3.micro, t3.nano] }
Mappings:
  RegionMap:
    us-east-1: { AMI: ami-0aaa }
Resources:
  WebServer:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: !Ref InstanceType
      ImageId: !FindInMap [RegionMap, !Ref "AWS::Region", AMI]
Outputs:
  PublicIp:
    Value: !GetAtt WebServer.PublicIp
    Export:
      Name: !Sub "${AWS::StackName}-PublicIp"
```

### Updates and change sets

| Update behaviour | Effect | Physical ID |
|------------------|--------|-------------|
| No interruption | Changed in place, keeps running | Same |
| Some interruption | Brief outage, e.g. stop and start | Same |
| Replacement | New resource created, old one deleted | New |

- **Direct update**: applied at once. **Change set**: a preview that you execute or delete.
- `aws cloudformation deploy` creates a change set and runs it straight away unless you add `--no-execute-changeset`.
- A change set does not name the three behaviours. Infer them from Action and Replacement.

```bash
aws cloudformation deploy --template-file template.yaml --stack-name my-stack \
  --capabilities CAPABILITY_NAMED_IAM --no-execute-changeset
```

### Stack status and rollback

| Operation | In progress | Success | Failure |
|-----------|-------------|---------|---------|
| Review | `REVIEW_IN_PROGRESS` (change set waiting on a new stack) | | |
| Create | `CREATE_IN_PROGRESS` | `CREATE_COMPLETE` | `ROLLBACK_IN_PROGRESS`, then `ROLLBACK_COMPLETE` or `ROLLBACK_FAILED` |
| Update | `UPDATE_IN_PROGRESS`, `UPDATE_COMPLETE_CLEANUP_IN_PROGRESS` | `UPDATE_COMPLETE` | `UPDATE_ROLLBACK_IN_PROGRESS`, then `UPDATE_ROLLBACK_COMPLETE` or `UPDATE_ROLLBACK_FAILED` |
| Delete | `DELETE_IN_PROGRESS` | `DELETE_COMPLETE` | `DELETE_FAILED` |

- Rollback is on by default and returns the stack to its last good state.
- A stack whose creation rolled back (`ROLLBACK_COMPLETE`) cannot be updated. Delete it and deploy again, as he does in several labs.
- A rollback can fail too, for example when a resource was deleted by hand.
- `DELETE_FAILED`: retry the delete and retain the resource that blocks it.

### Resource attributes

These sit beside `Type` and `Properties`, not inside `Properties`.

| Attribute | What it does |
|-----------|--------------|
| `CreationPolicy` | Resource stays in progress until it receives N success signals or the timeout passes |
| `DeletionPolicy` | On stack delete: `Delete`, `Retain` or `Snapshot` (databases and volumes) |
| `UpdateReplacePolicy` | Same choices, for the old resource when an update replaces it |
| `UpdatePolicy` | How updates roll out to an Auto Scaling group, ElastiCache replication group, OpenSearch domain or Lambda alias |
| `DependsOn` | Forces creation order when CloudFormation cannot work it out |
| `Metadata` | Free data on a resource; `AWS::CloudFormation::Init` is the special key that `cfn-init` reads |

### Stack policy

- A JSON document attached to the stack. It looks like IAM but is not IAM.
- Resource is `LogicalResourceId/<name in the template>`.
- Actions: `Update:Modify`, `Update:Replace`, `Update:Delete`, `Update:*`.
- Set at creation with `create-stack`, or later with `set-stack-policy`.

```json
{ "Statement": [
  { "Effect": "Deny", "Action": "Update:Replace", "Principal": "*",
    "Resource": "LogicalResourceId/ProductionDatabase" },
  { "Effect": "Allow", "Action": "Update:*", "Principal": "*", "Resource": "*" } ] }
```

### Intrinsic functions and pseudo parameters

| Function | Returns |
|----------|---------|
| `Ref` | A parameter's value, or a resource's main identifier (ID, name or ARN; check the docs per resource) |
| `Fn::GetAtt` | Another attribute, e.g. `!GetAtt Sg.GroupId` |
| `Fn::Sub` | A string with `${Name}` placeholders filled in |
| `Fn::Join`, `Fn::Split`, `Fn::Select` | Build, cut or pick from lists |
| `Fn::FindInMap` | A value from `Mappings` |
| `Fn::ImportValue` | A value exported by another stack |
| `Fn::GetAZs`, `Fn::Cidr`, `Fn::Base64` | Availability Zones of a Region, CIDR blocks, encoded user data |
| `Fn::If`, `Fn::Equals`, `Fn::And`, `Fn::Or`, `Fn::Not` | Condition logic |
| `Fn::ForEach`, `Fn::ToJsonString`, `Fn::Length` | Newer; need `Transform: AWS::LanguageExtensions` |

- Short form is `!Ref X`; long form is `Ref: X` or `Fn::GetAtt: [...]`.
- Pseudo parameters need no declaration: `AWS::AccountId`, `AWS::Region`, `AWS::StackName`, `AWS::StackId`, `AWS::Partition`, `AWS::URLSuffix`, `AWS::NotificationARNs`, `AWS::NoValue`. Most often used to build ARNs.

```yaml
Value: !Sub
  - "arn:aws:ec2:${AWS::Region}:${AWS::AccountId}:vpc/${VpcId}"
  - VpcId: !GetAtt MyInstance.VpcId
```

### Cross-stack references and nested stacks

| | Cross-stack reference | Nested stack |
|---|-----------------------|--------------|
| Purpose | Share a value between independent stacks | Reuse a template as a component |
| How | `Outputs` with `Export: Name:` in one stack, `Fn::ImportValue` in the other | Resource of type `AWS::CloudFormation::Stack` with `TemplateURL` |
| Lifecycle | Separate; the exporter cannot be deleted while its export is imported | Created, updated and deleted through the root stack only |

Export names must be unique in a Region.

### Drift detection

- Drift: the real configuration differs from what the stack expects, because of manual changes.
- Resource status: `IN_SYNC`, `MODIFIED`, `DELETED`, `NOT_CHECKED`. Stack status: `DRIFTED` or `IN_SYNC`.
- Detection only reports. You fix drift yourself.
- It compares only properties that the template sets. Extra resources created by hand are invisible to it.
- Detection on a parent does not cover nested stacks. Run it on the nested stack.

### Helper scripts, creation policies and wait conditions

Python scripts in the `aws-cfn-bootstrap` package, pre-installed on Amazon Linux.

| Script | Job |
|--------|-----|
| `cfn-init` | Reads `AWS::CloudFormation::Init` metadata: packages, groups, users, sources, files, commands, services. Called from user data. |
| `cfn-signal` | Sends success or failure to a `CreationPolicy` or wait condition |
| `cfn-get-metadata` | Prints a resource's metadata |
| `cfn-hup` | Daemon that checks for metadata changes (every 15 minutes) and runs hooks, so a stack update can reconfigure a running instance |

| | `CreationPolicy` | Wait condition |
|---|------------------|----------------|
| Pieces | One attribute on the resource | `WaitConditionHandle` (a presigned URL) plus `WaitCondition` with `Count` and `Timeout` |
| Use for | EC2 instances and Auto Scaling groups (AWS recommendation) | Configuration that happens outside the stack |

```yaml
WebAsg:
  Type: AWS::AutoScaling::AutoScalingGroup
  CreationPolicy:
    ResourceSignal: { Count: 2, Timeout: PT5M }
    AutoScalingCreationPolicy: { MinSuccessfulInstancesPercent: 100 }
```

```bash
/opt/aws/bin/cfn-signal -e $? --stack ${AWS::StackName} --resource WebAsg --region ${AWS::Region}
```

### Other features

| Feature | One line |
|---------|----------|
| Custom resource | `Custom::Name` with a `ServiceToken` pointing to a Lambda function or SNS topic, which must handle Create, Update and Delete requests. For anything CloudFormation cannot manage. |
| Registry | Public and private extensions: resource types, modules, hooks. Supports drift detection; built with the `cfn` CLI. |
| IaC generator | Scans existing resources and writes a template for those you select |
| Resource import | Brings an existing resource under a stack without recreating it |
| CDK | Imperative: write infrastructure in a programming language, which compiles to CloudFormation templates (declarative) |
| Quick Starts | Ready-made templates with a reference architecture and a deployment guide |

### Not in the video

| Topic | Remember |
|-------|----------|
| StackSets | One template deployed to many accounts and Regions from an administrator account. Service-managed permissions use AWS Organizations and can auto-deploy to new accounts; self-managed permissions need an administration role and an execution role. |
| Termination protection | Blocks deletion of the whole stack. Off by default. |
| Dynamic references | `{{resolve:ssm:/name}}`, `{{resolve:ssm-secure:/name}}`, `{{resolve:secretsmanager:secret-id:SecretString:key}}`. Keeps secrets out of templates. |
| `UPDATE_ROLLBACK_FAILED` | Fix the cause, then run `continue-update-rollback`, with `--resources-to-skip` for resources that cannot be recovered. He only mentions calling AWS Support. |
| Stack policy defaults | Once a policy is set, every update is denied unless a statement allows it. It cannot be removed, only replaced, or overridden for one update. |
| Capabilities | Templates that create IAM resources need `CAPABILITY_IAM` or `CAPABILITY_NAMED_IAM`; macros need `CAPABILITY_AUTO_EXPAND`. Otherwise: `InsufficientCapabilitiesException`. |
| Instance logs | `/var/log/cfn-init.log` and `/var/log/cloud-init-output.log` when a signal never arrives. |

## Memory hooks

- Cross-stack shares a value. Nested reuses a template. StackSets spread a stack over accounts and Regions.
- `init` installs, `signal` reports, `hup` watches.
- Three locks: stack policy against updates, termination protection against stack delete, `DeletionPolicy` for the resource when the stack does go.
- `Ref` gives the main ID. `GetAtt` gives everything else.

## Exam traps

- "See what an update will replace before applying it": change set.
- "Keep the database, or its data, when the stack is deleted": `DeletionPolicy: Retain` or `Snapshot`. When an update replaces it: `UpdateReplacePolicy`.
- "Stack shows `CREATE_COMPLETE` before the software is installed": `CreationPolicy` with `cfn-signal`.
- "Stack delete fails on an S3 bucket": the bucket is not empty. Empty it and retry, or retain it.
- "Someone changed a security group in the console": drift detection finds it; it does not revert it.
- "Stop an update replacing the production database": stack policy denying `Update:Replace`. It does not stop stack deletion.

## Outdated or wrong in the video

- **"Metadata holds the description and template version."** Those are their own top-level sections. `Metadata` is separate. Very sure.
- **"t4g.micro is not available in ca-central-1."** His later labs show the real cause: t4g is ARM and his AMI was x86_64. Fairly sure.
- **Changing the instance type means replacement.** For an EBS-backed instance it is "some interruption": stop, resize, start, same instance ID. `Conditional` means replacement depends on the case. Very sure.
- **"Use the ignore rollback flag."** The options are `--disable-rollback` or `--on-failure DO_NOTHING`. Very sure.
- **"Drift detection is done in the UI, not programmatically."** `aws cloudformation detect-stack-drift` exists, and AWS Config has a managed rule for stack drift. Very sure.
- **Drift detection is shown as report-only.** Since November 2025, drift-aware change sets can preview and revert drift. Very sure.
- **A nested stack can take a template body.** Inside a template only `TemplateURL` works. Fairly sure.
- **"Import won't work with EC2 instances."** It does; he imports one with the IaC generator earlier in the chapter. Very sure.
- **`cfn-hup` "watches instance metadata".** It polls the resource's `Metadata` in the stack, not the EC2 metadata service. `cfn-init` is also separate from cloud-init. Very sure.
- **Old names.** Application Composer is now Infrastructure Composer. Quick Starts are now Partner Solutions in the AWS Solutions Library. CDK also supports Go. Fairly sure.

Related: [[FCC 09.1 - S3 basics, CLI and IaC tooling]] · [[FCC 20 - ASG]] · [[FCC 25 - AWS Config]] · [[FCC 30 - Elastic Beanstalk Follow Along]] · [[FCC 33 - Service Catalog]] · [[FCC 00 - Watch guide]]
