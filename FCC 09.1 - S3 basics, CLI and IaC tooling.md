# FCC 09.1 - S3 basics, CLI and IaC tooling

> Video section [07:42:14](https://youtu.be/5JTBS09e7ew?t=27734) to 11:56:20 (part 1 of 5 of the S3 chapter) · 4h14m total · 2 min lecture, the rest labs
> Fast path: read this note and skip the whole section. It is general scripting practice (bash, PowerShell, Ruby, Java, Terraform, CDK, Pulumi), and almost none of it is exam content.

## Summary

- S3 is serverless object storage. An object is a key, a value (the bytes), optional metadata and an optional version ID. Buckets hold objects.
- Bucket names share one global namespace, but each bucket lives in the Region you create it in.
- "Folders" are not real. They are prefixes in the object key.
- There are two CLIs: `aws s3` for everyday file commands and `aws s3api` for the full low-level API.
- A bucket must be empty before it can be deleted.

## Labs and demos

| Time | Len | What it shows | Takeaway | Verdict |
|------|-----|---------------|----------|---------|
| [07:44:03](https://youtu.be/5JTBS09e7ew?t=27843) | 54m | Creates a bucket in the console, uploads a file, changes its storage class, then repeats everything in CloudShell with `aws s3` and `aws s3api` | Default encryption (SSE-S3) cannot be turned off. Storage class is set per object, not per bucket. `aws s3 cp` uploads, downloads and copies; with `s3api` those are `put-object`, `get-object` and `copy-object`. | Skim |
| [08:37:51](https://youtu.be/5JTBS09e7ew?t=31071) | 72m | Writes bash scripts to create and delete buckets, list newest buckets, sync random files, put one object and bulk-delete objects with `jq` | Outside us-east-1, `s3api create-bucket` fails without `--create-bucket-configuration LocationConstraint=<region>`. `delete-objects` wants a JSON file passed as `file://`. | Skip |
| [09:50:20](https://youtu.be/5JTBS09e7ew?t=35420) | 25m | Installs PowerShell and the `AWS.Tools.S3` module, then writes a script using `New-S3Bucket` and `Write-S3Object` | Works on the second run. He says PowerShell rarely matters for AWS. | Skip |
| [10:15:45](https://youtu.be/5JTBS09e7ew?t=36945) | 27m | Ruby SDK script that creates a bucket and uploads random files; then sets up a Java Maven project | The Ruby script works. The Java part stops after project setup with no code written. | Skip |
| [10:42:42](https://youtu.be/5JTBS09e7ew?t=38562) | 18m | CloudFormation: a three-line `AWS::S3::Bucket` template, deployed with `aws cloudformation deploy --no-execute-changeset`, reviewed and executed in the console, then deleted | A change set shows what will be replaced before anything happens. `delete-stack` aimed at the wrong Region reports nothing and deletes nothing. | Skim |
| [11:01:08](https://youtu.be/5JTBS09e7ew?t=39668) | 14m | Terraform: install, `aws_s3_bucket`, then `init`, `plan`, `apply`, `destroy` | State lives in a local `terraform.tfstate` file, which can hold secrets and must not be lost or committed. | Skip |
| [11:14:42](https://youtu.be/5JTBS09e7ew?t=40482) | 22m | AWS CDK in TypeScript: `cdk init`, an `s3.Bucket` construct, `cdk bootstrap`, `synth`, `deploy`, `destroy` | CDK generates CloudFormation. It needs a one-time `cdk bootstrap` per account and Region. Ten minutes go on a "no stack found" error caused by passing the wrong scope. | Skip |
| [11:36:44](https://youtu.be/5JTBS09e7ew?t=41804) | 20m | Pulumi with Python in GitHub Codespaces: `pulumi new aws-python`, `pulumi up`, `pulumi destroy` | The generated starter already creates a bucket. State is stored in Pulumi's cloud service. | Skip |

## Lecture index

| Time | Topic |
|------|-------|
| [07:42:14](https://youtu.be/5JTBS09e7ew?t=27734) | What object storage is, objects and buckets (2 min) |

## Notes

### The two CLIs

| | `aws s3` | `aws s3api` |
|---|----------|-------------|
| Level | High-level file commands | One command per API action |
| Commands | `ls`, `cp`, `mv`, `rm`, `sync`, `mb`, `rb`, `presign` | `create-bucket`, `put-object`, `get-object`, `list-objects-v2`, `delete-objects` and every other S3 action |
| Output | Plain text | JSON, filterable with `--query` |
| Bulk copy | `sync` and `cp --recursive` | None; one object per call |

```bash
aws s3 mb s3://my-bucket-ab123
aws s3 cp hello.txt s3://my-bucket-ab123/
aws s3 sync ./images s3://my-bucket-ab123/images
aws s3 rm s3://my-bucket-ab123 --recursive
aws s3 rb s3://my-bucket-ab123

aws s3api create-bucket --bucket my-bucket-ab123 --region ca-central-1 \
  --create-bucket-configuration LocationConstraint=ca-central-1
aws s3api list-buckets --query "Buckets[].Name" --output text
```

### Infrastructure as code tools compared

| Tool | Language | Where state is kept | Preview step |
|------|----------|---------------------|--------------|
| CloudFormation | YAML or JSON | Managed by AWS, free | Change set |
| Terraform | HCL | A state file you manage | `terraform plan` |
| CDK | TypeScript, Python, Java, .NET, Go | CloudFormation, which it generates | `cdk diff` or `cdk synth` |
| Pulumi | General programming languages | Pulumi's service by default | `pulumi preview` |

## Memory hooks

- `s3` is the remote control, `s3api` is the circuit board.
- A bucket name is like a domain name: globally unique, lowercase, no underscores.
- Change set, plan, diff, preview: four tools, one idea, which is "show me before you do it".

## Exam traps

- "Cannot delete the bucket" usually means it still contains objects or object versions.
- `IllegalLocationConstraintException` on `create-bucket` means the Region and the location constraint do not match.
- The S3 console says "Global", but data never leaves the bucket's Region unless you replicate it.
- Changing an object's storage class makes a new copy of the object.

## Outdated or wrong in the video

- **"An object can be 0 bytes to 5 TB."** AWS raised the maximum object size to 50 TB in late 2025. Older questions still assume 5 TB. Fairly sure.
- **"I thought you could only have 100 buckets."** The default quota is now 10,000 general purpose buckets per account. Fairly sure.

Related: [[FCC 09.2 - S3 buckets and objects]] · [[FCC 28.2 - AWS CLI]] · [[FCC 29 - AWS CloudFormation]] · [[FCC 00 - Watch guide]]
