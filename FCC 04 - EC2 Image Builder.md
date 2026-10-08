# FCC 04 - EC2 Image Builder

> Video chapter [03:03:53](https://youtu.be/5JTBS09e7ew?t=11033) to 04:30:23 · 1h26m total · 6 min lecture, 80 min labs
> Fast path: read this note and skip both labs. The second lab runs for an hour and ends in failure.

## Summary

- EC2 Image Builder is a managed service that builds AMIs or container images automatically, on a schedule or on demand.
- You define a pipeline made of a recipe, a workflow, an infrastructure configuration and distribution settings.
- The service itself is free. You pay for the build instance (m5.large unless you change it) and for the stored images.
- Rebuilding on a schedule keeps images patched, because every build reinstalls the latest packages.

## Labs and demos

| Time | Len | What it shows | Takeaway | Verdict |
|------|-----|---------------|----------|---------|
| [03:10:19](https://youtu.be/5JTBS09e7ew?t=11419) | 17m | Builds an AMI pipeline in the console: AL2023 base, user data that installs Apache, Java and Python components, a reboot test. Runs it and launches an instance from the result. | It works. The build is slow because the test stage launches a second instance. He spends several minutes wrongly concluding the build costs nothing. | Skim |
| [03:27:24](https://youtu.be/5JTBS09e7ew?t=12444) | 63m | Tries to build a container image of a Python app and push it to ECR. Six recipe versions, each failing at the build step. | Only a "hello world" container builds. He never gets the app files into the image and gives up at [04:28:03](https://youtu.be/5JTBS09e7ew?t=16083). Useful moments are listed below. | Skip |

### Useful moments inside the failed lab

| Time | What you learn |
|------|----------------|
| [03:40:53](https://youtu.be/5JTBS09e7ew?t=13253) | The build instance's role is `EC2InstanceProfileForImageBuilder`; he adds S3 read access to it |
| [03:44:00](https://youtu.be/5JTBS09e7ew?t=13440) | Proof the build runs on an m5.large EC2 instance, so it is not free |
| [03:49:08](https://youtu.be/5JTBS09e7ew?t=13748) | Creating a second infrastructure configuration to switch the build to t3.medium |
| [03:56:18](https://youtu.be/5JTBS09e7ew?t=14178) | The "Application logs" tab, which is far more useful than the step output |
| [04:15:46](https://youtu.be/5JTBS09e7ew?t=15346) | Writing a custom component that uses the `S3Download` action |

## Lecture index

| Time | Topic |
|------|-------|
| [03:03:53](https://youtu.be/5JTBS09e7ew?t=11033) | Pipeline parts, workflows, infrastructure configuration, distribution, components (6 min) |

## Notes

### Parts of a pipeline

| Part | What it defines |
|------|-----------------|
| Image recipe / container recipe | Base image, components, and user data or a Dockerfile |
| Components | Reusable YAML build or test steps (AWS provides many, such as installing the CloudWatch agent, the AWS CLI or Java) |
| Image workflow | The ordered build steps and test steps |
| Infrastructure configuration | The build instance: type, instance profile, VPC, subnet, security group, logging, SNS topic |
| Distribution settings | Where the result goes: Regions and accounts for an AMI, an ECR repository for a container |
| Schedule | Cron expression, schedule builder, or manual; EventBridge can also trigger it |
| Lifecycle policy | Rules that delete old images |

### Build stages

| Stage | What happens |
|-------|--------------|
| Build | Launches the build instance, waits for the SSM Agent, applies components, runs a sanitize script, creates the image |
| Test | Launches an instance from the new image and runs the test components |

## Memory hooks

- Recipe = what goes in; infrastructure = which oven; distribution = where the cake is delivered.
- Image Builder is the factory, the AMI is the product.

## Exam traps

- "Automate building and patching golden AMIs with the least effort" points to EC2 Image Builder.
- Unexpected charges come from the build instance and from stored AMIs, snapshots and ECR images.
- The build instance needs an instance profile with SSM access, or the build cannot start.
- Distribution settings are how one pipeline delivers an AMI to several Regions or accounts.

## Outdated or wrong in the video

- **"It seems to be free" (first lab).** Wrong, and he corrects it in the second lab: the service has no fee, the build instance and storage do. Very sure.
- **"The default security group doesn't allow traffic out to the internet."** A default security group allows all outbound traffic, so this was not the cause of his failure. Very sure.
- **Why the container lab fails.** He suspects networking and file paths. A likelier cause is that `aws s3 sync` inside a Dockerfile runs in the container being built, which has neither the AWS CLI nor credentials. He never confirms either explanation. Fairly sure.

Related: [[01 - EC2 & EBS]] · [[FCC 03 - Amazon Machine Images AMIs]] · [[FCC 22 - AWS Systems Manager]] · [[Amazon ECR]] · [[FCC 00 - Watch guide]]
