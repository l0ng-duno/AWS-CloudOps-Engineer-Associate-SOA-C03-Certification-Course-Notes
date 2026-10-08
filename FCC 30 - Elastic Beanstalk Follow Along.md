# FCC 30 - Elastic Beanstalk Follow Along

> Video chapter [60:40:12](https://youtu.be/5JTBS09e7ew?t=218412) to 61:58:19 · 1h18m total · no lecture, 78 min follow-along
> Fast path: read this note and skip the video. It is a lab reused from his Developer Associate course, built on tooling that has since been retired (Cloud9, Amazon Linux 1, Node 10). The only part worth seeing is the blue/green URL swap from [61:36:07](https://youtu.be/5JTBS09e7ew?t=221767) (2 min). The deployment theory the exam asks about is never taught here; it is in "Not in the video".

## Summary

- EB CLI work flow: `eb init` makes the application, `eb create` makes an environment, `eb deploy` ships a new version.
- The EB CLI deploys the last Git commit. Uncommitted changes, including new `.ebextensions` files, are not deployed.
- Settings live in `.ebextensions/*.config` under `option_settings`. The deployment policy is one of those settings.
- All at once is fast but interrupts the app. Immutable is slow but safe. Blue/green means: clone the environment, deploy to the clone, swap the two URLs, terminate the old one.
- Two IAM roles matter: the service role (used by Beanstalk) and the EC2 instance profile (used by the instances). Both real failures in this lab were role problems.

## Labs and demos

| Time | Len | What it shows | Takeaway | Verdict |
|------|-----|---------------|----------|---------|
| [60:40:12](https://youtu.be/5JTBS09e7ew?t=218412) | 19m | Creates a Cloud9 environment and a small Node.js Express app, then opens port 8080 with the CLI to preview it | Nothing about Beanstalk yet. He reads the security group ID from instance metadata and adds an inbound rule for his own IP with `/32`. | Skip |
| [60:58:58](https://youtu.be/5JTBS09e7ew?t=219538) | 8m | `git init`, `.gitignore`, first commit, then installs the EB CLI from the AWS GitHub installer | The EB CLI is a separate tool from the AWS CLI and must be added to `PATH`. | Skip |
| [61:07:12](https://youtu.be/5JTBS09e7ew?t=220032) | 7m | `eb init` with CodeCommit, then two `.ebextensions` files for environment variables and the Node start command | `eb init` writes `.elasticbeanstalk/config.yml`. Config files must be committed and pushed or the environment build fails. | Skip |
| [61:14:18](https://youtu.be/5JTBS09e7ew?t=220458) | 9m | `eb create --single` to launch a one-instance environment | Health goes yellow, then red: Beanstalk cannot assume `aws-elasticbeanstalk-service-role` because the roles were never created. He launches a sample app from the console to make the roles, terminates everything and runs `eb create` again. About 5 minutes lost. | Skip |
| [61:23:19](https://youtu.be/5JTBS09e7ew?t=220999) | 7m | Switches the deployment policy to immutable with a config file and deploys | First deploy fails on a typo ("invalid key"), cancelled with `eb abort`. The second prints "Immutable deployment policy enabled". He then deletes the file to return to all at once. | Skip |
| [61:30:36](https://youtu.be/5JTBS09e7ew?t=221436) | 9m | Blue/green: `eb clone`, deploy version 2 to the clone, `eb swap`, `eb terminate` the old environment | After the swap each environment holds the other's CNAME. The swap finishes in seconds. Watch [61:36:07](https://youtu.be/5JTBS09e7ew?t=221767) to 61:38:13 only. | Skim |
| [61:39:27](https://youtu.be/5JTBS09e7ew?t=221967) | 19m | Docker twice: first with a `Dockerfile` in the repo, then with an image pushed to ECR and a `Dockerrun.aws.json` | `docker push` fails until the ECR repository exists. The environment comes up degraded because the instances cannot pull from ECR; attaching `AmazonEC2ContainerRegistryReadOnly` to the instance role and redeploying fixes it. | Skip |

### Useful moments inside the follow-along

| Time | What you learn |
|------|----------------|
| [60:49:09](https://youtu.be/5JTBS09e7ew?t=218949) | Instance metadata gives the MAC address, then the security group IDs of that network interface |
| [60:51:10](https://youtu.be/5JTBS09e7ew?t=219070) | `aws ec2 authorize-security-group-ingress` for port 8080 from one IP |
| [61:07:31](https://youtu.be/5JTBS09e7ew?t=220051) | The `eb init` prompts: Region, application name, platform, CodeCommit, branch, SSH |
| [61:10:34](https://youtu.be/5JTBS09e7ew?t=220234) | Creating `.ebextensions` and the `option_settings` layout |
| [61:14:18](https://youtu.be/5JTBS09e7ew?t=220458) | `--single` means no load balancer; the instance gets an Elastic IP |
| [61:16:44](https://youtu.be/5JTBS09e7ew?t=220604) | Yellow then red health: "unable to assume role" and his console workaround |
| [61:21:48](https://youtu.be/5JTBS09e7ew?t=220908) | `eb status`, `eb logs` and `eb events` mirror what the console shows |
| [61:26:59](https://youtu.be/5JTBS09e7ew?t=221219) | `eb abort`, then the deploy that reports the immutable policy |
| [61:31:02](https://youtu.be/5JTBS09e7ew?t=221462) | Blue/green explained in one minute, then `eb clone` |
| [61:34:06](https://youtu.be/5JTBS09e7ew?t=221646) | `eb deploy <environment-name>` targets the clone instead of the default environment |
| [61:36:07](https://youtu.be/5JTBS09e7ew?t=221767) | `eb swap`: fails without `--destination_name`, then the CNAMEs change places |
| [61:47:26](https://youtu.be/5JTBS09e7ew?t=222446) | ECR repository must be created before `docker push` |
| [61:48:28](https://youtu.be/5JTBS09e7ew?t=222508) | `Dockerrun.aws.json`: version 1 for a single container |
| [61:54:36](https://youtu.be/5JTBS09e7ew?t=222876) | Degraded health because the instance profile cannot read ECR, and the fix |

## Lecture index

There are no slides. These are the only spoken explanations.

| Time | Topic |
|------|-------|
| [61:28:00](https://youtu.be/5JTBS09e7ew?t=221280) | Why immutable is slower than all at once but causes no interruption |
| [61:31:02](https://youtu.be/5JTBS09e7ew?t=221462) | What blue/green deployment is |
| [61:40:16](https://youtu.be/5JTBS09e7ew?t=222016) | Why package the app in Docker: you ship your own runtime version |

## Notes

### EB CLI commands he uses

| Command | What it does |
|---------|--------------|
| `eb init` | Creates the application and `.elasticbeanstalk/config.yml` |
| `eb create --single` | New environment with one instance and no load balancer |
| `eb status` | Health colour, status and CNAME |
| `eb logs`, `eb events` | Instance logs, environment event history |
| `eb deploy [env]` | Deploys the last commit to the default or the named environment |
| `eb abort` | Cancels a deployment in progress |
| `eb clone` | Copies an environment with its settings |
| `eb swap <env> --destination_name <other>` | Swaps the CNAMEs of two environments |
| `eb terminate <env>` | Deletes an environment |

Health colours seen in the lab: grey while launching, green when healthy, yellow for a warning, red when degraded.

### Config files as typed in the video

Environment variables (`.ebextensions/001-envar.config`):

```yaml
option_settings:
  aws:elasticbeanstalk:application:environment:
    PORT: 8081
    NODE_ENV: production
```

Deployment policy (`.ebextensions/000-deploy.config`):

```yaml
option_settings:
  aws:elasticbeanstalk:command:
    DeploymentPolicy: Immutable
    HealthCheckSuccessThreshold: Warning
    IgnoreHealthCheck: true
    Timeout: 600
```

Config files are applied in alphabetical order, which is why they carry number prefixes.

### Docker on Beanstalk

| | `Dockerfile` in the repo | `Dockerrun.aws.json` pointing at ECR |
|---|--------------------------|--------------------------------------|
| Who builds the image | Beanstalk, on the instance | You, then you push it to a registry |
| What the repo holds | Source code and the `Dockerfile` | Only the JSON file and `.ebextensions` |
| Extra permission | None | Instance profile needs `AmazonEC2ContainerRegistryReadOnly` |

```bash
docker build -t study-sync .
aws ecr get-login-password | docker login --username AWS \
  --password-stdin 123456789012.dkr.ecr.us-east-1.amazonaws.com
docker tag <image-id> 123456789012.dkr.ecr.us-east-1.amazonaws.com/study-sync
docker push 123456789012.dkr.ecr.us-east-1.amazonaws.com/study-sync
```

```json
{ "AWSEBDockerrunVersion": "1",
  "Image": { "Name": "123456789012.dkr.ecr.us-east-1.amazonaws.com/study-sync:latest" },
  "Ports": [ { "ContainerPort": "8080" } ] }
```

### Not in the video

The lab only uses all at once and immutable. The exam expects all of these.

| Deployment policy | How it works | During the deployment | Rollback |
|-------------------|--------------|-----------------------|----------|
| All at once | New version on every instance at the same time | Short outage | Redeploy the old version |
| Rolling | One batch of instances at a time | No outage, capacity reduced by one batch | Redeploy the old version |
| Rolling with additional batch | Launches an extra batch first, then rolls | No outage, full capacity | Redeploy the old version |
| Immutable | New instances in a temporary Auto Scaling group; they join the original group once healthy | No outage, capacity doubles for a while | Terminate the new instances; fast |
| Traffic splitting | Sends a set percentage of traffic to new instances for a test period | Canary test; needs an Application Load Balancer | Traffic returns to the old instances |
| Blue/green | A second environment, then "Swap environment URLs" | Not a policy setting; the switch is a DNS change | Swap back |

| Topic | Remember |
|-------|----------|
| Environment tiers | Web server tier answers HTTP requests. Worker tier reads jobs from an SQS queue; `cron.yaml` schedules periodic jobs. |
| `.ebextensions` keys | `option_settings`, `packages`, `files`, `commands`, `container_commands`, `Resources` (extra CloudFormation resources). Files are YAML or JSON and end in `.config`. |
| Database | An RDS instance created inside an environment is deleted with it. Create it outside for production and for blue/green. |
| Cost | Beanstalk is free. You pay for the EC2, load balancer and other resources it creates. |

## Memory hooks

- init, create, deploy = application, environment, version.
- Immutable replaces the instances. Blue/green replaces the whole environment and swaps the name on the door.
- "With additional batch" means "rolling without losing capacity".

## Exam traps

- "Fastest deployment, a short outage is acceptable": all at once.
- "Keep full capacity during the deployment at the lowest extra cost": rolling with additional batch.
- "No downtime and the quickest, safest rollback": immutable.
- "Test the new version on 10% of users first": traffic splitting.
- "Switch to a new platform version and be able to go straight back": blue/green with Swap environment URLs.
- "Environment is degraded and cannot pull the image from ECR": add `AmazonEC2ContainerRegistryReadOnly` to the instance profile.

## Outdated or wrong in the video

- **Cloud9 as the workspace.** Cloud9 has been closed to new customers since July 2024. Use CloudShell or your own machine. Very sure.
- **"If Amazon Linux 1 is here, absolutely use it."** Amazon Linux 1 reached end of life in December 2023 and its Beanstalk platforms are retired. Very sure.
- **`NodeCommand` and `NodeVersion` in `aws:elasticbeanstalk:container:nodejs`, with Node 10.** Those options only worked on the old Amazon Linux 1 Node.js platform. Current platforms use a `Procfile` and the `engines` field in `package.json`. Fairly sure.
- **"`eb create` is supposed to create the roles", and his fix of launching a sample app from the console.** Beanstalk no longer creates the `aws-elasticbeanstalk-ec2-role` instance profile for you. In a new account you create it in IAM and attach `AWSElasticBeanstalkWebTier`. Very sure.
- **CodeCommit.** It was closed to new customers in July 2024 and returned to general availability in late 2025, so this part works again. Fairly sure.
- **"Version two of `Dockerrun.aws.json` is for multicontainer."** Version 2 is for the ECS-managed Docker platform. On the plain Docker platform, several containers are described with a `docker-compose.yml` file. Fairly sure.
- **Plain `curl` to `169.254.169.254`.** This is IMDSv1. Instances that require IMDSv2 need a token first (see FCC 02). Very sure.

Related: [[FCC 29 - AWS CloudFormation]] · [[FCC 20 - ASG]] · [[FCC 19 - ELB]] · [[FCC 21 - Identity and Access Management IAM]] · [[FCC 31 - SQS]] · [[FCC 00 - Watch guide]]
