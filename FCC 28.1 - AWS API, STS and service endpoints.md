# FCC 28.1 - AWS API, STS and service endpoints

> Video section [52:39:39](https://youtu.be/5JTBS09e7ew?t=189579) to 54:26:00 (part 1 of 2 of the AWS API chapter) · 1h46m total · about 35 min lecture, 1h11m labs and demos
> Fast path: read this note, then watch 5 minutes of the STS lab from [53:29:17](https://youtu.be/5JTBS09e7ew?t=192557) (trust policy fix, `assume-role` working, temporary credentials in use). For SOA-C03 what counts is STS and role assumption, access key hygiene, throttling and retries, and endpoint types. Smithy and hand-signing requests are background: skip them.

## Summary

- Every AWS operation is a signed HTTPS call to a service endpoint. The console, CLI and SDKs all call the same API.
- An access key is a key ID plus a secret. It is long-lived and carries all of its user's permissions. Two per user at most.
- STS gives temporary credentials: key ID, secret, **session token** and an expiry. `AssumeRole` is the call to know.
- Role assumption has two sides. The role's trust policy must trust the caller. If the role is in another account, the caller's own policy must also allow `sts:AssumeRole`.
- Failed calls are retried with exponential backoff. Endpoints are Regional, global, FIPS, dual-stack or feature-specific.

## Labs and demos

| Time | Len | What it shows | Takeaway | Verdict |
|------|-----|---------------|----------|---------|
| [52:56:38](https://youtu.be/5JTBS09e7ew?t=190598) | 46m | A new IAM user with no permissions assumes a role with `aws sts assume-role` and reads an S3 bucket. CloudFormation builds the role and bucket. | Works in the end. The first 20 minutes are setup. Then 11 minutes of AccessDenied: allowing the user `sts:AssumeRole` does not help, because the trust policy names `s3.amazonaws.com` instead of the user. Watch 5 minutes from [53:29:17](https://youtu.be/5JTBS09e7ew?t=192557). | Skim |
| [53:57:36](https://youtu.be/5JTBS09e7ew?t=194256) | 20m | Points the CLI at LocalStack, a local AWS emulator in Docker, with `--endpoint-url`. Repeats it with an environment variable and with boto3. | Buckets are created in the emulator, not in AWS. The first 9 minutes are installing and starting LocalStack. LocalStack is not on the exam. | Skip |
| [54:19:53](https://youtu.be/5JTBS09e7ew?t=195593) | 6m | Looks up DynamoDB `CreateTable` in the API reference, CLI reference, CloudFormation, boto3 and the IAM policy editor | The same action and parameter names appear in all five. | Skip |

### Useful moments inside the STS lab

| Time | What you learn |
|------|----------------|
| [53:07:44](https://youtu.be/5JTBS09e7ew?t=191264) | `aws sts get-caller-identity` shows which identity a profile uses |
| [53:18:00](https://youtu.be/5JTBS09e7ew?t=191880) | Builds the `aws sts assume-role` command. It returns AccessDenied a minute later. |
| [53:21:07](https://youtu.be/5JTBS09e7ew?t=192067) | `aws iam put-user-policy` adds an inline policy allowing `sts:AssumeRole` on the role. Still denied at [53:25:12](https://youtu.be/5JTBS09e7ew?t=192312). |
| [53:29:17](https://youtu.be/5JTBS09e7ew?t=192557) | The fix: the trust policy principal becomes `"AWS": "<user ARN>"`. `assume-role` returns credentials at [53:30:18](https://youtu.be/5JTBS09e7ew?t=192618). |
| [53:31:18](https://youtu.be/5JTBS09e7ew?t=192678) | The three values go into `~/.aws/credentials` as a new profile, with `aws_session_token`. At [53:33:21](https://youtu.be/5JTBS09e7ew?t=192801) `get-caller-identity` shows the role session. |
| [53:39:27](https://youtu.be/5JTBS09e7ew?t=193167) | `aws iam delete-user` refuses until the inline policy and access key are deleted |

## Lecture index

| Time | Topic |
|------|-------|
| [52:39:39](https://youtu.be/5JTBS09e7ew?t=189579) | What an API is; console, CLI and SDK all call it |
| [52:43:17](https://youtu.be/5JTBS09e7ew?t=189797) | What a CLI is: terminal, console, shell (belongs with part 2) |
| [52:46:36](https://youtu.be/5JTBS09e7ew?t=189996) | Access keys, credentials file, environment variables |
| [52:51:16](https://youtu.be/5JTBS09e7ew?t=190276) | API retries and exponential backoff |
| [52:53:02](https://youtu.be/5JTBS09e7ew?t=190382) | Smithy (background) |
| [52:54:21](https://youtu.be/5JTBS09e7ew?t=190461) | Security Token Service |
| [53:42:10](https://youtu.be/5JTBS09e7ew?t=193330) | Signing requests, then Signature Version 4 at [53:43:59](https://youtu.be/5JTBS09e7ew?t=193439) (background) |
| [53:46:01](https://youtu.be/5JTBS09e7ew?t=193561) | Service endpoints: format and types |
| [53:48:46](https://youtu.be/5JTBS09e7ew?t=193726) | Regional endpoints, then global at [53:49:59](https://youtu.be/5JTBS09e7ew?t=193799) |
| [53:50:57](https://youtu.be/5JTBS09e7ew?t=193857) | FIPS endpoints |
| [53:52:37](https://youtu.be/5JTBS09e7ew?t=193957) | Troubleshooting endpoints: telnet, nslookup, ping |
| [53:54:40](https://youtu.be/5JTBS09e7ew?t=194080) | Dual-stack endpoints, then feature-specific at [53:56:31](https://youtu.be/5JTBS09e7ew?t=194191) |
| [54:17:12](https://youtu.be/5JTBS09e7ew?t=195432) | API actions and how they are named |

## Notes

### Access keys

| Point | Detail |
|-------|--------|
| Made of | Access key ID and secret access key. Treat both as secret. |
| Limit | 2 per IAM user. Deactivate or delete one to make another. |
| Permissions | Exactly those of the user. For narrow access, create a separate "machine user". |
| Never | Share a key or commit it to Git |
| File | `~/.aws/credentials`: a `[default]` profile plus named profiles. `aws configure` writes it. |
| Environment | `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_SESSION_TOKEN`. The CLI and SDKs read them automatically. |

Region and output format live in a second file, `~/.aws/config`. The CLI itself is covered in part 2.

### Retries and exponential backoff

- Calls fail for network reasons, so plan to send them again.
- Wait twice as long before each retry: 1, 2, 4, 8, 16, 32 seconds.
- The CLI and SDKs have this built in and let you tune it. Raw HTTP callers must code it themselves.

### STS: temporary credentials

| Action | Use |
|--------|-----|
| `AssumeRole` | A user or role takes on another role, in the same or another account |
| `AssumeRoleWithWebIdentity` | Sign-in through a web identity provider. The request is not signed. |
| `GetCallerIdentity` | "Who am I?" Needs no permissions. |

A response contains `AccessKeyId`, `SecretAccessKey`, `SessionToken` and `Expiration`.

| Policy | Where | Says |
|--------|-------|------|
| Trust policy | On the role | Who may assume it (`Principal`) |
| Identity policy | On the caller | Allow `sts:AssumeRole` on the role ARN |
| Permissions policy | On the role | What the session may do |

```bash
aws sts assume-role --role-arn arn:aws:iam::111122223333:role/MyRole \
  --role-session-name s3-sts-fun --profile sts
```

```json
{ "Version": "2012-10-17", "Statement": [ { "Effect": "Allow",
    "Principal": { "AWS": "arn:aws:iam::111122223333:user/sts-machine-user" },
    "Action": "sts:AssumeRole" } ] }
```

- `"Service": "s3.amazonaws.com"` as the principal lets the S3 service assume the role. It does nothing for a user.
- His advice: give a user only the right to assume roles.
- `aws s3 ls` needs `s3:ListAllMyBuckets` on `"Resource": "*"`. The lab role covered one bucket, so `aws s3 ls` was denied while `aws s3 ls s3://<bucket>` worked.

### Signing (background)

- A signature proves who sent the request and that it was not altered. The CLI and SDKs sign for you.
- Signature Version 4 derives the signing key from the secret access key. The signature goes in the `Authorization` header or the query string, as in a presigned URL.
- Not signed: anonymous reads of public S3 objects, and `AssumeRoleWithWebIdentity`.

### Service endpoints

Format: `https://<service>.<region>.amazonaws.com`, for example `https://cloudformation.us-east-2.amazonaws.com`.

| Type | Example | Notes |
|------|---------|-------|
| Regional | `ec2.us-east-1.amazonaws.com` | Most services, including S3 and STS. `ec2.amazonaws.com` routes to us-east-1. |
| Global | `iam.amazonaws.com` | IAM, Route 53, CloudFront, Global Accelerator |
| FIPS | `ec2-fips.us-east-1.amazonaws.com` | FIPS 140 validated cryptography, for US and Canadian government work. Some services and Regions only. |
| Dual-stack | `s3.dualstack.us-east-1.amazonaws.com`, `ec2.us-east-1.api.aws` | Answers on IPv4 and IPv6 |
| Feature-specific | `s3-control`, `route53resolver`, `runtime.sagemaker` | A large feature with its own endpoint |

Types combine, for example Regional plus FIPS plus dual-stack.

| Override | How |
|----------|-----|
| One command | `--endpoint-url https://kms-fips.us-east-1.amazonaws.com`, a global option on every command |
| All services | `AWS_ENDPOINT_URL` |
| One service | `AWS_ENDPOINT_URL_S3` |
| FIPS switch | `AWS_USE_FIPS_ENDPOINT=true` |

| Tool | Checks | Example |
|------|--------|---------|
| `nslookup` | DNS: does the name resolve | `nslookup s3.amazonaws.com` |
| `telnet` | TCP: does port 443 accept a connection | `telnet s3.amazonaws.com 443` |
| `ping` | ICMP: does the host answer | `ping s3.amazonaws.com` |

### API actions

| Place | How `RunInstances` appears |
|-------|----------------------------|
| Raw API | `?Action=RunInstances` |
| CLI | `aws ec2 run-instances` |
| SDK (Python, Ruby) | `run_instances` |
| IAM policy | `ec2:RunInstances` |

- "Action" and "operation" mean the same thing. Common verbs: Describe, Create, Update, Delete.
- CloudFormation properties mirror the API parameters, e.g. `BillingMode`.
- The CLI adds helpers with no API action behind them: `wait` and `wizard`.
- One console click can call several actions.

### Not in the video

He explains retries only as a cure for network faults. The exam asks about throttling.

| Topic | Fact |
|-------|------|
| Throttling | AWS rate-limits API calls. `ThrottlingException`, `RequestLimitExceeded` or "Rate exceeded" means retry with exponential backoff and jitter (a random spread). |
| What is retried | Throttling, HTTP 500, 502, 503, 504 and timeouts. AccessDenied and validation errors are not. |
| Tuning | `AWS_RETRY_MODE` (`legacy`, `standard`, `adaptive`) and `AWS_MAX_ATTEMPTS`. CLI v2 defaults to `standard` with 3 attempts. |
| Session length | `AssumeRole` lasts 1 hour by default. Minimum 15 minutes, maximum set on the role (up to 12 hours). |

## Memory hooks

- Long-term key: two parts, no expiry. STS credentials: three parts and a clock.
- AssumeRole is a handshake. The role says "I trust you"; your own policy says "you may ask".
- Backoff doubles: 1, 2, 4, 8.
- Endpoint = service, Region, `amazonaws.com`. No Region means global. `-fips` for government. `dualstack` for IPv6.

## Exam traps

- "AccessDenied on `sts:AssumeRole` although the user's policy allows it": the role's trust policy does not name the caller.
- "Temporary credentials are rejected as invalid": the session token is missing, or they have expired.
- "A script fails now and then with ThrottlingException or Rate exceeded": retry with exponential backoff and jitter.
- "Access key pushed to a public repository": deactivate or delete it at once, create a new one, then check CloudTrail for what it did.
- "API calls must use FIPS validated cryptography": use the FIPS endpoints.
- "An application on EC2 needs AWS access": an IAM role through an instance profile, never stored access keys.

## Outdated or wrong in the video

- **"STS is a global service with a single endpoint, `sts.amazonaws.com`."** STS also has Regional endpoints, `sts.<region>.amazonaws.com`, and AWS recommends them. He lists STS as a Regional service himself later. Very sure.
- **"You send your credentials and get back a token for the Authorization header."** There is no round trip. The signature is calculated locally from the secret access key. Very sure.
- **"The credentials file is a TOML file" and "Python is required for the CLI."** The file is INI-style text. CLI v2 ships with its own Python. Very sure.
- **"FIPS is a method of using keys with envelope encryption."** FIPS 140 is a standard for validating cryptographic modules, as he says later. He cites 140-2; AWS now refers to 140-3. Very sure.
- **"TLS version two, or an older one might be supported."** AWS API endpoints require TLS 1.2 or later and accept TLS 1.3. Very sure.
- **"nslookup shows the path."** It only resolves a name to an address. Very sure.
- **`BatchExecuteStatement` is missing from the IAM editor and he guesses the UI is incomplete.** IAM actions do not always match API names. That call is authorised by the `dynamodb:PartiQL*` actions. Very sure.

Related: [[FCC 28.2 - AWS CLI]] · [[FCC 21 - Identity and Access Management IAM]] · [[FCC 24 - AWS CloudTrail]] · [[FCC 29 - AWS CloudFormation]] · [[FCC 15.2 - VPC gateways, IPv6, endpoints and flow logs]] · [[FCC 00 - Watch guide]]
