# FCC 22 - AWS Systems Manager

> Video chapter [42:41:46](https://youtu.be/5JTBS09e7ew?t=153706) to 44:03:12 · 1h21m total · about 47 min lecture, 34 min labs
> Fast path: read this note and watch nothing. The Systems Manager part is 40 minutes of slides with no demo at all. The last 42 minutes are about Secrets Manager, and all three labs are CloudFormation templates for secrets. The slides skip several points the exam tests; learn the "Not in the video" table as well.

## Summary

- Systems Manager (SSM) is a toolbox for running fleets of EC2 instances and on-premises servers: connect, run commands, patch, automate, keep configuration.
- An instance becomes a managed node when it runs the SSM Agent and its role has the policy `AmazonSSMManagedInstanceCore`.
- Session Manager is a shell with no SSH keys, no open inbound ports and no bastion. Run Command runs a script on many nodes at once. State Manager repeats a document on a schedule. Patch Manager patches by baseline. Automation runs runbooks against AWS APIs.
- Parameter Store keeps configuration and secrets in a hierarchy; the standard tier is free.
- Secrets Manager costs $0.40 per secret per month, always encrypts, and rotates database passwords for you.

## Labs and demos

There is no Systems Manager lab in this chapter. The three labs are Secrets Manager.

| Time | Len | What it shows | Takeaway | Verdict |
|------|-----|---------------|----------|---------|
| [43:28:42](https://youtu.be/5JTBS09e7ew?t=156522) | 15m | Creates a secret with CloudFormation, first with a generated password, then with a value typed into the template. Reads it back from a Ruby script. | `GenerateSecretString` puts a random password into your JSON template under the key you name. The console shows sample code for each language. The value arrives as a JSON string in `SecretString`, so parse it. About 2 minutes are lost deploying under the wrong stack name. An old secret cannot be deleted at once. | Skip |
| [43:43:14](https://youtu.be/5JTBS09e7ew?t=157394) | 6m | Passes a secret into the template as a parameter with `NoEcho: true` | The stack's Parameters tab shows `****`. The value is still readable in Secrets Manager. His advice: do not pass secrets as parameters, generate them. | Skip |
| [43:49:16](https://youtu.be/5JTBS09e7ew?t=157756) | 14m | One template with a generated secret, an RDS instance that reads it, and a `SecretTargetAttachment` | After the deploy the secret also holds engine, host, port, database name and instance identifier, added by the attachment. Rotation stays off because it needs a Lambda function. Most of the time is copy-and-paste trouble in the template. He does not test the login. | Skip |

## Lecture index

| Time | Topic |
|------|-------|
| [42:41:46](https://youtu.be/5JTBS09e7ew?t=153706) | What Systems Manager is, list of capabilities |
| [42:43:38](https://youtu.be/5JTBS09e7ew?t=153818) | SSM Agent |
| [42:44:54](https://youtu.be/5JTBS09e7ew?t=153894) | Policy for the instance role |
| [42:45:59](https://youtu.be/5JTBS09e7ew?t=153959) | Explorer |
| [42:47:25](https://youtu.be/5JTBS09e7ew?t=154045) | OpsCenter |
| [42:49:09](https://youtu.be/5JTBS09e7ew?t=154149) | OpsItems: status, priority, related resources |
| [42:50:59](https://youtu.be/5JTBS09e7ew?t=154259) | Remediation and runbooks |
| [42:53:08](https://youtu.be/5JTBS09e7ew?t=154388) | Tags and Resource Groups |
| [42:54:22](https://youtu.be/5JTBS09e7ew?t=154462) | AppConfig |
| [42:55:50](https://youtu.be/5JTBS09e7ew?t=154550) | Automation and the approval use case |
| [42:57:22](https://youtu.be/5JTBS09e7ew?t=154642) | Automation documents, queue limits, action types |
| [43:01:06](https://youtu.be/5JTBS09e7ew?t=154866) | Change Calendar |
| [43:01:42](https://youtu.be/5JTBS09e7ew?t=154902) | Maintenance Windows |
| [43:02:12](https://youtu.be/5JTBS09e7ew?t=154932) | Compliance and Inventory |
| [43:03:15](https://youtu.be/5JTBS09e7ew?t=154995) | Hybrid activations |
| [43:04:06](https://youtu.be/5JTBS09e7ew?t=155046) | Session Manager |
| [43:05:48](https://youtu.be/5JTBS09e7ew?t=155148) | Run Command |
| [43:07:34](https://youtu.be/5JTBS09e7ew?t=155254) | State Manager |
| [43:09:17](https://youtu.be/5JTBS09e7ew?t=155357) | Patch Manager |
| [43:10:25](https://youtu.be/5JTBS09e7ew?t=155425) | Distributor |
| [43:13:50](https://youtu.be/5JTBS09e7ew?t=155630) | SSM documents |
| [43:16:09](https://youtu.be/5JTBS09e7ew?t=155769) | Parameter Store: tiers, policies, hierarchy |
| [43:21:27](https://youtu.be/5JTBS09e7ew?t=156087) | Secrets Manager: secret types, pricing, rotation, CLI |
| [43:26:39](https://youtu.be/5JTBS09e7ew?t=156399) | Secrets Manager architecture diagram |

## Notes

### What a managed node needs

| Requirement | Detail |
|-------------|--------|
| SSM Agent | Preinstalled on Amazon Linux, Ubuntu Server, Windows Server and ECS-optimised AMIs. Otherwise download the package, install it with the package manager and start the service. |
| IAM role | An instance profile whose role has the AWS managed policy `AmazonSSMManagedInstanceCore`. |

### Capabilities

| Capability | What it does | Remember |
|------------|--------------|----------|
| Session Manager | Shell in the browser or CLI, Linux and Windows | No inbound ports, SSH keys or bastion. Access through IAM. Sessions logged to S3 or CloudWatch Logs, API calls in CloudTrail. Port forwarding. |
| Run Command | Runs a command document on many nodes at once | Shell or PowerShell scripts, installing agents, Windows updates, Ansible or Chef. For ad hoc work. |
| State Manager | Applies a command or automation document to targets on a schedule; this link is an association | It only re-runs the document. The document decides the state. |
| Patch Manager | Patches operating systems and applications | A patch baseline holds auto-approval rules (N days after release) and approved and rejected lists. Schedule it as a maintenance window task. Target by tags. |
| Maintenance Windows | A schedule for disruptive work such as patching | You register targets and tasks |
| Automation | Runbooks for multi-step tasks on AWS resources | See below |
| Change Calendar | Says when automations may run | Open by default: blocked only during events. Closed by default: allowed only during events. |
| Parameter Store | Configuration and secrets | See below |
| Documents | YAML or JSON definitions used by the other capabilities | Types: Command, Automation, Package, Session, Policy, Change Calendar |
| Inventory | Collects software and configuration data from nodes | Can be stored in S3 and queried |
| Compliance | Shows patch and configuration compliance across the fleet | |
| Hybrid Activations | Registers on-premises servers and VMs in other clouds | An activation gives a code and an ID, used like an access key pair |
| Distributor | Packages and installs software, including AWS agents such as the CloudWatch agent | Once or on a schedule (it uses State Manager). Packages can be shared with other accounts. |
| OpsCenter | One list of OpsItems: operational work to investigate and resolve | Created by hand or by EventBridge, Config and CloudWatch. Status Open, In progress, Resolved. Priority 1 to 5. A runbook can be attached for one-click remediation. |
| Explorer | Dashboard of OpsItems and patch compliance across accounts and Regions | Reports export as CSV to S3 |
| Resource Groups | Resources grouped by shared tags | |
| AppConfig | Validated, gradual rollout of application configuration | Source: S3, SSM document or parameter. Validator: JSON Schema or Lambda. Strategy: linear or exponential, with a bake time. |

### Automation

| Part | Detail |
|------|--------|
| Runbook | An Automation document, `schemaVersion: '0.3'` |
| `assumeRole` | The IAM role the automation runs as |
| `parameters` | Input variables |
| `mainSteps` | The steps. Each has `name`, `action` and `inputs`, plus optional `maxAttempts`, `timeoutSeconds`, `onFailure`, `nextStep` |
| Ready-made runbooks | Many from AWS. His example: stop an instance with approval. SNS emails the approvers, then the instance stops. |
| Notifications | Through EventBridge |

Action types to recognise: `aws:approve` (wait for a person), `aws:executeAwsApi`, `aws:executeScript`, `aws:runCommand`, `aws:invokeLambdaFunction`, `aws:executeAutomation`, `aws:executeStateMachine`, `aws:branch`, `aws:sleep`, `aws:pause`, `aws:waitForAwsResourceProperty`, `aws:assertAwsResourceProperty`, `aws:changeInstanceState`, `aws:runInstances`, `aws:createImage`, `aws:copyImage`, `aws:deleteImage`, `aws:createStack`, `aws:deleteStack`, `aws:createTags`.

### Parameter Store

| | Standard | Advanced |
|---|----------|----------|
| Parameters per Region | 10,000 | 100,000 |
| Largest value | 4 KB | 8 KB |
| Parameter policies | No | Yes |
| Cost | Free | Charged per parameter |
| Change tier | Can become advanced | Cannot go back to standard |

- Types: `String`, `StringList` (comma-separated), `SecureString` (encrypted with KMS). Encryption is optional.
- A name with slashes forms a hierarchy, e.g. `/app/prod/db-url`. One call returns a whole branch.
- Every change creates a new version.
- Parameter policies: `Expiration` (delete at a date), `ExpirationNotification` (warn before it), `NoChangeNotification` (warn when nobody changed the value in N days).

```bash
aws ssm put-parameter --name /planets/vulcan/population --value 6000000 --type String
aws ssm get-parameters-by-path --path /planets/vulcan
```

### Secrets Manager

| | Parameter Store | Secrets Manager |
|---|-----------------|-----------------|
| Cost | Standard tier free | $0.40 per secret per month, $0.05 per 10,000 API calls |
| Encryption | Optional | Always, with the default KMS key or your own |
| Rotation | Build it yourself with Lambda | Built in, run by a Lambda function |
| Secret types | Any string | RDS, Redshift, DocumentDB, other database, other (key/value or JSON) |

- Flow: Secrets Manager changes the password in the database and stores the new one. The application on EC2 fetches it with the SDK each time it connects. A developer fetches it with the CLI.
- `aws secretsmanager describe-secret --secret-id x` shows metadata and version IDs.
- `aws secretsmanager get-secret-value --secret-id x` returns `SecretString`. The default version stage is `AWSCURRENT`.
- Access to secrets is recorded in CloudTrail.
- A deleted secret is kept for a recovery window of 7 to 30 days.

```yaml
MySecret:
  Type: AWS::SecretsManager::Secret
  Properties:
    GenerateSecretString:
      SecretStringTemplate: '{"username": "testuser"}'
      GenerateStringKey: password
MyDb:
  Type: AWS::RDS::DBInstance
  Properties:
    MasterUsername: !Sub '{{resolve:secretsmanager:${MySecret}:SecretString:username}}'
    MasterUserPassword: !Sub '{{resolve:secretsmanager:${MySecret}:SecretString:password}}'
Attachment:
  Type: AWS::SecretsManager::SecretTargetAttachment
  Properties:
    SecretId: !Ref MySecret
    TargetId: !Ref MyDb
    TargetType: AWS::RDS::DBInstance
```

`AWS::SecretsManager::RotationSchedule` and `AWS::SecretsManager::ResourcePolicy` are separate resources.

### Not in the video

| Topic | Remember |
|-------|----------|
| Network path | The agent needs outbound HTTPS (443) to Systems Manager. In a private subnet with no NAT, add interface VPC endpoints `ssm`, `ssmmessages` and `ec2messages`. |
| Node is not listed as managed | Check three things: agent running, instance profile with `AmazonSSMManagedInstanceCore`, network path. |
| Session Manager | The user needs `ssm:StartSession`. The CLI needs the Session Manager plugin. The security group needs no port 22. |
| Run Command | Target by tags or resource group. Rate control: concurrency and error threshold. Output to S3 or CloudWatch Logs. |
| Patch Manager | The tag key `Patch Group` maps nodes to a baseline. The document `AWS-RunPatchBaseline` runs with operation `Scan` or `Install`. |
| Command or Automation | A Command document runs inside the node through the agent. An Automation runbook runs in the service and calls AWS APIs. |
| Automation uses | Remediation action for AWS Config rules, target of EventBridge rules, patching a golden AMI. |
| Reading parameters | `--with-decryption` for a `SecureString`. CloudFormation uses `{{resolve:ssm:name}}`. Public parameters return the latest AMI ID. |
| Hybrid nodes | Their IDs start with `mi-`. EC2 instances keep `i-`. |

## Memory hooks

- Session Manager: a person at a shell. Run Command: one script, many nodes, now. State Manager: the same script, again and again. Automation: steps against AWS APIs.
- A node needs ARN: Agent, Role, Network.
- Parameter Store is the free notice board. Secrets Manager is the paid safe that changes its own combination.
- Change Calendar answers "may we?". A maintenance window answers "when?".

## Exam traps

- "Shell access to instances in a private subnet, no bastion, no SSH keys, with an audit trail": Session Manager.
- "Instance does not appear in Systems Manager": agent, instance profile, route to the SSM endpoints.
- "Run a script on hundreds of instances, a few at a time, and stop when too many fail": Run Command with rate control.
- "Keep an agent installed and configured on every instance, new ones included": a State Manager association.
- "Patch production on Sunday night and development on another day": patch baselines, patch groups and maintenance windows.
- "Rotate a database password automatically": Secrets Manager. "Cheapest place for configuration values": Parameter Store.

## Outdated or wrong in the video

- **Change Calendar types explained backwards.** Open by default means actions run except during events. To freeze changes on a demo day, use open by default with the day as an event; his slide was right and his correction wrong. Very sure.
- **"Distributor takes MSI, deb or npm files."** It is `.rpm`, the Red Hat package format, not npm. Very sure.
- **"Activations expire after 30 days, so you keep renewing."** The expiry (30 days at most) is the deadline for registering servers with that code. Registered servers stay registered. Fairly sure.
- **"25 concurrent automations, queue of 1,000."** Now 100 concurrent and a queue of 5,000. Fairly sure.
- **"`aws:executeScript` is probably a bash script."** It runs Python or PowerShell. Very sure.
- **List of AMIs with the agent preinstalled.** It is old. Today: Amazon Linux 2 and 2023, Ubuntu Server 18.04 and later, Windows Server 2016 to 2025, macOS, SLES 15, and the ECS and EKS optimised AMIs. `AmazonEC2RoleforSSM` is deprecated. Very sure.
- **"API calls cost half a cent per 10,000."** Five cents ($0.05). Very sure.
- **"Key/value secrets cannot rotate" and "up to 365 days".** Any secret can rotate with your own Lambda function; AWS only supplies ready-made functions for databases. The interval can be as short as 4 hours and longer than a year. Very sure.
- **"A CloudWatch Events rule triggers the rotation Lambda."** Secrets Manager schedules and invokes the function itself. Fairly sure.
- **Hand-built RDS secret.** RDS can now create and rotate the master password itself (`ManageMasterUserPassword: true`), with no Lambda. Very sure.

Related: [[FCC 02 - Amazon EC2]] · [[FCC 21 - Identity and Access Management IAM]] · [[FCC 23.2 - CloudWatch metrics and the CloudWatch agent]] · [[FCC 25 - AWS Config]] · [[FCC 29 - AWS CloudFormation]] · [[FCC 00 - Watch guide]]
