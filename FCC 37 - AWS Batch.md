# FCC 37 - AWS Batch

> Video chapter [64:25:26](https://youtu.be/5JTBS09e7ew?t=231926) to 64:53:15 · 28 min total · 1 min lecture, 27 min lab
> Fast path: read this note and skip the video. The lab never gets a job to run and he gives up. The cause of the failure and the order that works are below.

## Summary

- AWS Batch plans, schedules and runs batch computing jobs as containers. It starts the compute when jobs arrive and can use Spot to save money.
- Parts: job definition (the template), job (one run), job queue (where jobs wait), compute environment (what they run on), scheduler.
- Jobs run on EC2, Fargate or EKS.
- The scheduler is first in, first out by default.
- Supports array jobs, multi-node parallel jobs, GPU jobs and dependencies between jobs.

## Labs and demos

| Time | Len | What it shows | Takeaway | Verdict |
|------|-----|---------------|----------|---------|
| [64:26:40](https://youtu.be/5JTBS09e7ew?t=232000) | 26m | Tries to run a tiny Python container (it prints the square of 3) as a Batch job on Fargate, using the CLI | Fails. About 5 minutes go on Docker build and output problems before the image reaches ECR. He then registers a job definition, creates a Fargate compute environment and a job queue. `submit-job` is rejected: the queue's compute environment "cannot run jobs with capability EC2". The console form shows no queue to pick either. After 5 more minutes he stops and blames the service. | Skip |

### Useful moments inside the Batch lab

| Time | What you learn |
|------|----------------|
| [64:36:50](https://youtu.be/5JTBS09e7ew?t=232610) | `register-job-definition` fails with "vCPUs is required"; a CLI example with `vcpus` and `memory` gets it registered |
| [64:39:45](https://youtu.be/5JTBS09e7ew?t=232785) | `submit-job` fails because there is no job queue yet |
| [64:43:10](https://youtu.be/5JTBS09e7ew?t=232990) | A queue needs a compute environment, and that has to exist first |
| [64:44:33](https://youtu.be/5JTBS09e7ew?t=233073) | Creates a Fargate compute environment in the console, maximum 1 vCPU |
| [64:47:37](https://youtu.be/5JTBS09e7ew?t=233257) | Queue created; `submit-job` rejected with the "capability EC2" error |
| [64:52:35](https://youtu.be/5JTBS09e7ew?t=233555) | His summary of the order he intended |

## Lecture index

| Time | Topic |
|------|-------|
| [64:25:26](https://youtu.be/5JTBS09e7ew?t=231926) | What Batch is, components, job types, and his warning that the lab does not succeed |

## Notes

### Building blocks

| Part | What it is | Remember |
|------|-----------|----------|
| Job definition | Template: container image, vCPUs, memory, command | Platform is EC2 unless you say FARGATE |
| Job | One submitted run of a definition | Needs a name, a queue and a definition |
| Job queue | Holds jobs until capacity is free | Has a state, a priority and an ordered list of compute environments |
| Compute environment | The capacity jobs run on | Managed: EC2, Spot, Fargate or Fargate Spot. Unmanaged: EC2 only, run by you. |
| Scheduler | Picks the next job | FIFO by default; a scheduling policy is optional |

Rules he reads from the documentation:
- All compute environments in one queue must be the same family: EC2 and Spot, or Fargate and Fargate Spot. No mixing.
- A compute environment must be in the VALID state before a queue can use it.
- Batch uses the service-linked role `AWSServiceRoleForBatch`. It did not exist in his account until the first compute environment was created.
- `create-job-queue` wants `--compute-environment-order order=1,computeEnvironment=<ARN>`.

### Why the lab failed

He never finds the cause. It is this:
- His job definition came from an EC2-style example. A job definition defaults to the EC2 platform.
- His queue pointed at a Fargate compute environment. Batch refuses that mismatch, which is what the error says.
- The fix is to register the job definition for Fargate: `--platform-capabilities FARGATE`, vCPU and memory given as `resourceRequirements`, and an execution role.

### The order that works

```bash
# 1. compute environment
aws batch create-compute-environment --compute-environment-name my-ce --type MANAGED \
  --compute-resources type=FARGATE,maxvCpus=1,subnets=subnet-123,securityGroupIds=sg-123

# 2. queue that points at it
aws batch create-job-queue --job-queue-name my-job-queue --state ENABLED --priority 10 \
  --compute-environment-order order=1,computeEnvironment=<compute-environment-arn>

# 3. job definition, with the platform stated
aws batch register-job-definition --job-definition-name square --type container \
  --platform-capabilities FARGATE --container-properties file://container.json

# 4. run it
aws batch submit-job --job-name my-job --job-queue my-job-queue --job-definition square
```

### Not in the video

| Topic | Remember |
|-------|----------|
| Job states | SUBMITTED, PENDING, RUNNABLE, STARTING, RUNNING, then SUCCEEDED or FAILED |
| Stuck in RUNNABLE | The compute environment cannot supply capacity: maximum vCPUs too low, the job asks for more than any allowed instance type has, no network path to pull the image, or a role problem |
| Logs | Job output goes to CloudWatch Logs, log group `/aws/batch/job` |
| Retries and timeouts | Set in the job definition |
| Alerts | Job state changes are EventBridge events; route them to SNS |

## Memory hooks

- The definition is the recipe, the job is the order, the queue is the order rail, the compute environment is the kitchen.
- Build from the bottom: compute environment, queue, definition, then submit.
- Lambda stops at 15 minutes. Batch does not.

## Exam traps

- "Long-running, compute-heavy jobs in containers at the lowest cost": Batch on Spot.
- "The job runs for hours": not Lambda, which is limited to 15 minutes.
- "Job stays in RUNNABLE": the compute environment cannot start capacity. Check maximum vCPUs, instance types, networking and roles.
- "Run thousands of similar jobs from one submission": array job.
- "Tightly coupled job across several instances": multi-node parallel job, which needs EC2, not Fargate.
- S3 Batch Operations is a different thing: it acts on S3 objects.

## Outdated or wrong in the video

- **"AWS Batch is the problem, not me", and "there was no information on how to get it to work".** The error names the cause. His job definition was for EC2 and his queue was Fargate. Registering the definition with `--platform-capabilities FARGATE` and Fargate-style resource settings fixes it. Very sure.
- **Components list in the lecture.** He names jobs, job definitions, job queues and the scheduler, and leaves out the compute environment, which the lab shows is required. Very sure.

Related: [[FCC 02 - Amazon EC2]] · [[FCC 20 - ASG]] · [[FCC 23.1 - CloudWatch Logs and Logs Insights]] · [[FCC 23.3 - EventBridge and CloudWatch alarms]] · [[FCC 00 - Watch guide]]
