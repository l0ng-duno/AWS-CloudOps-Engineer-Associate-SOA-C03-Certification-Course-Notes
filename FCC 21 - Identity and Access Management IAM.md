# FCC 21 - Identity and Access Management IAM

> Video chapter [41:25:03](https://youtu.be/5JTBS09e7ew?t=149103) to 42:41:46 · 1h17m total · about 8 min lecture, 69 min labs
> Fast path: read this note and skip the chapter. The lectures are 8 minutes of basics. The big lab (44 min) ends unsolved; the answer he never finds is in "S3 actions and the ARN they need" below. Optional: 3 minutes from [42:11:02](https://youtu.be/5JTBS09e7ew?t=151862) to see bucket ARN versus object ARN in the visual editor. Roles, trust policies and policy evaluation are missing; see "Not in the video".

## Summary

- IAM has three identities (users, groups, roles) and policies. A policy is a JSON document that allows or denies actions on resources.
- Three policy types: AWS managed (read-only, written by AWS), customer managed (yours, reusable, versioned), inline (embedded in one identity).
- Policy elements: `Version`, `Statement`, `Sid`, `Effect`, `Action`, `Resource`, `Principal`, `Condition`.
- Each action needs the right resource type. Bucket actions take the bucket ARN, object actions take `bucket/*`, and listing all buckets takes `*`.
- Numbers: 2 access keys per user, 5 versions per managed policy.

## Labs and demos

| Time | Len | What it shows | Takeaway | Verdict |
|------|-----|---------------|----------|---------|
| [41:28:50](https://youtu.be/5JTBS09e7ew?t=149330) | 18m | Attaches an AWS managed, a customer managed and an inline policy to one user in the console. Then rebuilds this as a CloudFormation template. | The console part (first 4 minutes) is the whole lesson. The stack fails once with "extraneous key not permitted" on the user name property; he deletes the property without finding the cause. Result: one AWS managed policy and three inline ones. He never gets a customer managed policy out of CloudFormation. | Skip |
| [41:49:22](https://youtu.be/5JTBS09e7ew?t=150562) | 44m | Strips a user down to `iam:*` and `sts:*`, then writes a customer managed policy in YAML, converts it with `yq` and pushes it with the CLI, to practise least privilege on S3 | The first 25 minutes work and are in "Useful moments". From 42:14:06 to 42:30:19 (16 min) `aws s3 ls` keeps returning AccessDenied and he gives up. Cause: wrong action, see below. | Skim: only [42:11:02](https://youtu.be/5JTBS09e7ew?t=151862) to 42:14:06 (3 min) |
| [42:33:38](https://youtu.be/5JTBS09e7ew?t=153218) | 1m | Opens IAM, Account settings, Password policy | Reads out the options. Listed below. | Skip |
| [42:36:46](https://youtu.be/5JTBS09e7ew?t=153406) | 5m | Adds an authenticator app as MFA device to an IAM user, then removes it | Scan the QR code from inside the authenticator app (the phone's camera app fails for him), then enter two codes in a row. "Resync" repairs a device whose codes stop being accepted. Root MFA is set under the account menu, Security credentials. | Skip |

### Useful moments inside the policy lab

| Time | What you learn |
|------|----------------|
| [41:53:36](https://youtu.be/5JTBS09e7ew?t=150816) | Policy skeleton typed out: `Version`, `Statement`, `Sid`, `Effect`, `Action`, `Resource` |
| [41:58:41](https://youtu.be/5JTBS09e7ew?t=151121) | `create-policy` fails three times with `MalformedPolicyDocument`: `Resources` instead of `Resource`, a top-level array instead of an object ([41:59:46](https://youtu.be/5JTBS09e7ew?t=151186)), spaces in the `Sid` ([42:01:50](https://youtu.be/5JTBS09e7ew?t=151310)) |
| [42:02:51](https://youtu.be/5JTBS09e7ew?t=151371) | There is no `update-policy`. You change a managed policy with `create-policy-version` |
| [42:04:55](https://youtu.be/5JTBS09e7ew?t=151495) | Right after `attach-user-policy` the call is still denied, then works: IAM changes take a few seconds |
| [42:11:02](https://youtu.be/5JTBS09e7ew?t=151862) | Visual editor: `ListBucket` asks for a bucket ARN, `GetObject` asks for an object ARN |
| [42:20:20](https://youtu.be/5JTBS09e7ew?t=152420) | IAM policy simulator says `s3:ListBucket` is allowed, which is true and not what `aws s3 ls` needs |
| [42:22:23](https://youtu.be/5JTBS09e7ew?t=152543) | `ListAllMyBuckets` appears on screen and he passes over it. Then the console refuses to save: a managed policy holds at most 5 versions |
| [42:30:35](https://youtu.be/5JTBS09e7ew?t=153035) | Adding a condition to a statement, e.g. `StringEquals` on `s3:prefix` |

## Lecture index

| Time | Topic |
|------|-------|
| [41:25:03](https://youtu.be/5JTBS09e7ew?t=149103) | Users, groups, roles, policies and how they combine |
| [41:27:40](https://youtu.be/5JTBS09e7ew?t=149260) | AWS managed, customer managed and inline policies |
| [41:46:53](https://youtu.be/5JTBS09e7ew?t=150413) | Anatomy of a policy document |
| [42:33:29](https://youtu.be/5JTBS09e7ew?t=153209) | Password policy |
| [42:35:10](https://youtu.be/5JTBS09e7ew?t=153310) | Access keys |
| [42:35:55](https://youtu.be/5JTBS09e7ew?t=153355) | MFA |

## Notes

### Identities

| Identity | What it is | Remember |
|----------|------------|----------|
| User | A person or application with long-term credentials: console password, access keys | Up to 2 access keys |
| Group | A set of users that share permissions | Attach policies to the group, not to each user |
| Role | Permissions with no long-term credentials. A user, a service or another account assumes it and gets temporary credentials | Reaches EC2 through an instance profile |

### Policy types

| Type | Written by | Editable | Reusable | Typical use |
|------|-----------|----------|----------|-------------|
| AWS managed | AWS | No | Yes | Job functions: `AdministratorAccess`, `PowerUserAccess`, `ReadOnlyAccess`, `ViewOnlyAccess`, `Billing` |
| Customer managed | You | Yes, as versions | Yes | Your own least-privilege policies |
| Inline | You | Yes | No | A one-off permission for one user, group or role |

- `ReadOnlyAccess` can read data, such as S3 objects. `ViewOnlyAccess` only lists resources and their metadata.
- `PowerUserAccess` is for developers: almost everything except managing IAM.

### Policy document elements

| Element | Meaning | Note |
|---------|---------|------|
| `Version` | Policy language version | Always `2012-10-17` |
| `Statement` | One statement, or an array of them | The document itself is an object, not an array |
| `Sid` | Optional label for a statement | Letters and digits only, unique in the policy |
| `Effect` | `Allow` or `Deny` | |
| `Action` | API calls, e.g. `s3:GetObject` or `s3:*` | |
| `Resource` | ARNs the actions apply to | |
| `Principal` | Who the statement is about: account, user, role, federated user | Only in resource-based policies, such as a bucket policy |
| `Condition` | Extra test that must be true | Keys are service-specific (`s3:prefix`) or global (`aws:...`) |

### S3 actions and the ARN they need

| Command | Action | Resource |
|---------|--------|----------|
| `aws s3 ls` | `s3:ListAllMyBuckets` | `*` |
| `aws s3 ls s3://my-bucket` | `s3:ListBucket` | `arn:aws:s3:::my-bucket` |
| `aws s3 cp s3://my-bucket/file .` | `s3:GetObject` | `arn:aws:s3:::my-bucket/*` |

```json
{ "Version": "2012-10-17",
  "Statement": [
    { "Sid": "ListBuckets", "Effect": "Allow",
      "Action": "s3:ListAllMyBuckets", "Resource": "*" },
    { "Sid": "ListOneBucket", "Effect": "Allow",
      "Action": "s3:ListBucket", "Resource": "arn:aws:s3:::my-bucket",
      "Condition": { "StringEquals": { "s3:prefix": "hello/" } } },
    { "Sid": "ReadObjects", "Effect": "Allow",
      "Action": "s3:GetObject", "Resource": "arn:aws:s3:::my-bucket/*" } ] }
```

### Managed policies from the CLI

```bash
aws sts get-caller-identity                    # which identity am I using?
aws iam create-policy --policy-name my-policy --policy-document file://policy.json
aws iam attach-user-policy --user-name alice --policy-arn <policy-arn>
aws iam create-policy-version --policy-arn <policy-arn> \
  --policy-document "$(yq -o json policy.yaml)" --set-as-default
aws iam delete-policy-version --policy-arn <policy-arn> --version-id v1
```

- A sixth version is refused. Delete an old one first. The default version cannot be deleted.
- The policy simulator (`policysim.aws.amazon.com`) tests whether a user's policies allow an action, without running it.
- CloudFormation makes you tick "I acknowledge" (`CAPABILITY_IAM`) before it creates IAM resources.

| CloudFormation | Creates |
|----------------|---------|
| `AWS::IAM::User` with `ManagedPolicyArns` | A user with existing managed policies attached |
| `AWS::IAM::User` with `Policies` | Inline policies on that user |
| `AWS::IAM::Policy`, `AWS::IAM::UserPolicy` | Inline policies; `!Ref` on a user returns its name |
| `AWS::IAM::ManagedPolicy` | A customer managed policy (the one he was looking for) |

### Passwords, access keys and MFA

| Item | Facts |
|------|-------|
| Password policy | Account-wide, under Account settings. Minimum length, required character types, expiry after N days, admin reset after expiry, users may change their own password, no reuse. |
| Access keys | Access key ID plus secret access key, for the CLI and SDKs. Maximum 2 per user. A key can be made inactive. |
| MFA | Set per user: authenticator app, security key or hardware token. To force its use, allow actions only when the request was made with MFA (`aws:MultiFactorAuthPresent`). |

### Not in the video

| Topic | Remember |
|-------|----------|
| Evaluation order | An explicit `Deny` beats every `Allow`. No matching `Allow` means denied. |
| Trust policy | The resource policy on a role. It says who may assume the role (`sts:AssumeRole`). |
| `iam:PassRole` | Needed to hand a role to a service, such as launching EC2 with an instance profile. |
| Permissions boundary, SCP | They cap what an identity or account can do. They never grant anything. |
| Credential report | CSV of all users: password age, key age, MFA on or off. |
| IAM Access Analyzer | Finds resources shared outside the account and validates policies. |
| Finding a denied call | CloudTrail records it with an `AccessDenied` error code. |

## Memory hooks

- Managed is a badge many people can wear. Inline is a tattoo on one identity.
- PARC: Principal, Action, Resource, Condition, plus an Effect.
- The shelf versus the books: `my-bucket` to list, `my-bucket/*` to read.
- 2 keys, 5 versions.

## Exam traps

- "`GetObject` on `bucket/*` is allowed but listing the bucket is denied": add `s3:ListBucket` on the bucket ARN without `/*`.
- "One policy allows, another denies": the explicit deny wins.
- "Application on EC2 needs AWS access": a role through an instance profile, never access keys on the instance.
- "Rotate an access key without downtime": create the second key, switch the application, deactivate the old key, then delete it.
- "Saving a managed policy fails with a limit error": it already has 5 versions.
- "Only allow sensitive actions after MFA": a condition on `aws:MultiFactorAuthPresent`.

## Outdated or wrong in the video

- **"You attach a role to a user or group."** You attach policies to users, groups and roles. A role is assumed, by users, services or other accounts. His later remark that roles are only for services is also too narrow. Very sure.
- **`Principal` shown as a normal part of any policy.** A policy attached to a user, group or role has no `Principal`. It belongs in resource-based policies and trust policies. Very sure.
- **The unsolved AccessDenied.** `aws s3 ls` calls `ListBuckets`, which needs `s3:ListAllMyBuckets` on `*`. `s3:ListBucket` lists the objects in one bucket, not "some or all buckets" as he reads it. Very sure.
- **"There is a limit on the number of statements."** The limit is size: 6,144 characters for a managed policy. Very sure.
- **"An administrator cannot enforce MFA."** There is no switch, but a policy that denies everything except MFA set-up until MFA is present does enforce it. Very sure.
- **Root user left without MFA.** AWS now requires MFA for the root user of every account type; the last group, member accounts, was added in 2025. Fairly sure.

Related: [[FCC 28.1 - AWS API, STS and service endpoints]] · [[FCC 09.4 - S3 security and encryption]] · [[FCC 24 - AWS CloudTrail]] · [[FCC 29 - AWS CloudFormation]] · [[FCC 22 - AWS Systems Manager]] · [[FCC 00 - Watch guide]]
