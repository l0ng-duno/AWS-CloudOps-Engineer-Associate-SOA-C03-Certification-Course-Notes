# FCC 28.2 - AWS CLI

> Video section [54:26:00](https://youtu.be/5JTBS09e7ew?t=195960) to 57:56:14 (part 2 of 2 of the AWS API chapter) · 3h30m total · about 75 min lecture, 2h15m labs
> Fast path: read this note; nothing has to be watched. If you want one demo, take the second half of the pagination lab (4 min from [54:54:45](https://youtu.be/5JTBS09e7ew?t=197685)). jq, jp, yq, base64, aliases, wizards, Docker and auto-prompt fill about 80 minutes and are work tools, not exam material.

## Summary

- `--filters` runs on the server and shrinks the response. `--query` (JMESPath) runs on your machine after the data has arrived. Use both together.
- Pagination: `--page-size` changes how many items each API call asks for, and the output stays complete. `--max-items` caps the output and returns a `NextToken` for `--starting-token`.
- Credentials are searched in a fixed order: command-line options, environment variables, the `credentials` and `config` files, container credentials, and the EC2 instance profile last.
- A profile in `~/.aws/config` can assume a role by itself: `role_arn` plus `source_profile` or `credential_source`, and `mfa_serial` if the role demands MFA.
- `aws <service> wait <condition>` holds a script until a resource is ready. Failed calls are retried automatically with exponential backoff (`retry_mode`, `max_attempts`).

## Labs and demos

| Time | Len | What it shows | Takeaway | Verdict |
|------|-----|---------------|----------|---------|
| [54:27:21](https://youtu.be/5JTBS09e7ew?t=196041) | 5m | Tours the developer tools page, the `aws-cli` GitHub repo and the v2 command reference | Each command page has options, examples and sometimes sample output. Check that the page says v2. | Skip |
| [54:35:16](https://youtu.be/5JTBS09e7ew?t=196516) | 4m | Prints `$?` after `aws sts get-caller-identity`, with good keys and then with a junk profile | 0 on success. Junk keys give 254, not the 253 he expected. He blames the docs, but 254 is correct. | Skip |
| [54:41:26](https://youtu.be/5JTBS09e7ew?t=196886) | 3m | Runs `aws configure wizard` and `aws iam wizard new-role`, then shows his tutorial on custom wizards | A wizard asks questions and builds the resource. He says it is not on the exam. | Skip |
| [54:45:20](https://youtu.be/5JTBS09e7ew?t=197120) | 4m | Previews `describe-vpcs` as json, table, text and yaml in the auto-prompt output panel | JSON is the default. `--query 'Vpcs[].State' --output text` gives bare values for scripts. | Skip |
| [54:50:40](https://youtu.be/5JTBS09e7ew?t=197440) | 8m | `describe-subnets` with `--max-items`, `--page-size` and `--starting-token` | `--page-size 1` is rejected (minimum 5 here). `--page-size 5` still prints all 7 subnets, in two calls. `--max-items 1` prints one plus a `NextToken`, which fetches the second. The first 4 min are setup. | Skim: from [54:54:45](https://youtu.be/5JTBS09e7ew?t=197685) (4 min) |
| [55:03:43](https://youtu.be/5JTBS09e7ew?t=198223) | 16m | Launches an instance, keeps its ID in a variable, then blocks on `aws ec2 wait instance-status-ok` | About 12 min of errors: no default VPC in ca-central-1, a missing space before `\`, a trailing space after `\`, a missing `--region`. The wait prints nothing and returns after a couple of minutes. | Skim: from [55:17:10](https://youtu.be/5JTBS09e7ew?t=199030) (3 min) |
| [55:23:49](https://youtu.be/5JTBS09e7ew?t=199429) | 4m | Saves a long `describe-subnets` command as the alias `dsub` | `aws dsub` works at once, with no reload. He prefers bash scripts and says it is not on the exam. | Skip |
| [55:29:27](https://youtu.be/5JTBS09e7ew?t=199767) | 4m | Filters `describe-subnets` by Availability Zone with `--filters` | `Name=availability-zone,Values=us-east-1a,us-east-1b` returns subnets in either zone. Filter names are on each command's reference page. | Skip |
| [55:35:08](https://youtu.be/5JTBS09e7ew?t=200108) | 5m | Tries `--query` on `describe-vpcs`, then asks ChatGPT for a query that nests subnets under VPCs | A multi-select such as `Vpcs[].[VpcId,CidrBlock]` works. The nested query fails; he blames JMESPath, but `describe-vpcs` returns no subnets at all. | Skip |
| [55:45:36](https://youtu.be/5JTBS09e7ew?t=200736) | 16m | Works through his jq cheat sheet on `describe-vpcs` output | See the jq row in the last table. Indexes start at 0; he calls `[1]` "first" and corrects it at [55:57:56](https://youtu.be/5JTBS09e7ew?t=201476). | Skip |
| [56:03:47](https://youtu.be/5JTBS09e7ew?t=201827) | 11m | Installs `jp` (the brew build fails with "permission denied", the binary download works) and tries JMESPath | His `[?IsDefault==false]` filter returns nothing and he gives up: the literal needs backticks. His final expression works unchanged in `--query`. | Skip |
| [56:18:17](https://youtu.be/5JTBS09e7ew?t=202697) | 3m | Pipes `describe-vpcs` JSON into yq to get YAML and XML | `yq -p json -o yaml` converts. CSV fails, and one of his jq expressions is a syntax error in yq. | Skip |
| [56:26:29](https://youtu.be/5JTBS09e7ew?t=203189) | 6m | Base64-encodes a string in Ruby, in Python and with the `base64` command | `echo "text" \| base64` gives a different result and he cannot say why. Cause: `echo` adds a newline, so use `echo -n`. | Skip |
| [56:38:20](https://youtu.be/5JTBS09e7ew?t=203900) | 12m | Generates a `run-instances` skeleton, converts it to YAML, edits it and launches with `--cli-input-yaml` | Other flags on the skeleton command are ignored. The whole skeleton fails with "Parameter validation failed"; a file with only `ImageId` and `InstanceType` works. | Skip |
| [56:54:50](https://youtu.be/5JTBS09e7ew?t=204890) | 23m | Chains commands: default VPC ID into a variable, a CIDR edited with `sed`, `create-subnet`, then one line with `xargs` | About half is lost to guessing a filter value, CIDR arithmetic and two missing `--region` flags. The patterns are under "Chaining commands". | Skim: xargs at [57:16:50](https://youtu.be/5JTBS09e7ew?t=206210) (1 min) |
| [57:33:40](https://youtu.be/5JTBS09e7ew?t=207220) | 7m | Runs the CLI from the `amazon/aws-cli` Docker image, then with `~/.aws` mounted, then from ECR Public | The first run pulls the image. `-v ~/.aws:/root/.aws` passes the credentials files in, and `s3 ls` lists his buckets. | Skip |

### Useful moments inside the chaining lab

| Time | What you learn |
|------|----------------|
| [56:55:25](https://youtu.be/5JTBS09e7ew?t=204925) | The pattern `export VPC_ID=$(aws ec2 describe-vpcs ...)` |
| [57:00:20](https://youtu.be/5JTBS09e7ew?t=205220) | The `is-default` filter wants `true`. `yes` and `Yes` return nothing, with no error. |
| [57:04:36](https://youtu.be/5JTBS09e7ew?t=205476) | `sed` rewrites the VPC CIDR into a subnet CIDR; he warns that `sed` syntax differs between distributions |
| [57:10:42](https://youtu.be/5JTBS09e7ew?t=205842) | `create-subnet` fails until the tag value is quoted, then with "VPC does not exist" until `--region` is added |
| [57:16:50](https://youtu.be/5JTBS09e7ew?t=206210) | `xargs -I {}` drops the first command's output into `Values={}` of the second |

## Lecture index

*Bold topics are the exam-relevant ones.*

| Time | Topic |
|------|-------|
| [54:26:00](https://youtu.be/5JTBS09e7ew?t=195960) | CLI command reference, v1 and v2 pages |
| [54:32:40](https://youtu.be/5JTBS09e7ew?t=196360) | Return codes |
| [54:39:21](https://youtu.be/5JTBS09e7ew?t=196761) | Wizards |
| [54:43:58](https://youtu.be/5JTBS09e7ew?t=197038) | **Output formats** |
| [54:49:17](https://youtu.be/5JTBS09e7ew?t=197357) | **Pagination flags** |
| [54:58:32](https://youtu.be/5JTBS09e7ew?t=197912) | Command flags in shell programs, then in the AWS CLI ([55:00:56](https://youtu.be/5JTBS09e7ew?t=198056)) |
| [55:02:35](https://youtu.be/5JTBS09e7ew?t=198155) | **The `wait` subcommand** |
| [55:19:47](https://youtu.be/5JTBS09e7ew?t=199187) | Aliases |
| [55:28:11](https://youtu.be/5JTBS09e7ew?t=199691) | **Server-side filters** |
| [55:33:49](https://youtu.be/5JTBS09e7ew?t=200029) | **Client-side `--query`** |
| [55:40:07](https://youtu.be/5JTBS09e7ew?t=200407) | JSON, then YAML ([55:41:45](https://youtu.be/5JTBS09e7ew?t=200505)) |
| [55:44:18](https://youtu.be/5JTBS09e7ew?t=200658) | jq |
| [56:02:05](https://youtu.be/5JTBS09e7ew?t=201725) | jp |
| [56:15:00](https://youtu.be/5JTBS09e7ew?t=202500) | yq |
| [56:21:35](https://youtu.be/5JTBS09e7ew?t=202895) | File parameters, `file://` and `fileb://` |
| [56:24:04](https://youtu.be/5JTBS09e7ew?t=203044) | Base64 encoding |
| [56:32:51](https://youtu.be/5JTBS09e7ew?t=203571) | Quoting strings |
| [56:34:16](https://youtu.be/5JTBS09e7ew?t=203656) | Parameter types |
| [56:36:06](https://youtu.be/5JTBS09e7ew?t=203766) | `--generate-cli-skeleton`, then `--cli-input-json` ([56:37:23](https://youtu.be/5JTBS09e7ew?t=203843)) |
| [56:50:05](https://youtu.be/5JTBS09e7ew?t=204605) | Shorthand syntax |
| [56:51:07](https://youtu.be/5JTBS09e7ew?t=204667) | PowerShell file encoding |
| [56:52:04](https://youtu.be/5JTBS09e7ew?t=204724) | Chaining with environment variables, then with xargs ([56:53:12](https://youtu.be/5JTBS09e7ew?t=204792)) |
| [57:18:00](https://youtu.be/5JTBS09e7ew?t=206280) | **`config` and `credentials` files** |
| [57:19:50](https://youtu.be/5JTBS09e7ew?t=206390) | **Named profiles** |
| [57:21:17](https://youtu.be/5JTBS09e7ew?t=206477) | `aws configure` commands, then single sign-on ([57:23:10](https://youtu.be/5JTBS09e7ew?t=206590)) |
| [57:23:33](https://youtu.be/5JTBS09e7ew?t=206613) | **Retries** |
| [57:25:23](https://youtu.be/5JTBS09e7ew?t=206723) | **Credentials on EC2** |
| [57:26:30](https://youtu.be/5JTBS09e7ew?t=206790) | Proxies |
| [57:27:33](https://youtu.be/5JTBS09e7ew?t=206853) | **IAM role in a profile** |
| [57:30:14](https://youtu.be/5JTBS09e7ew?t=207014) | `credential_process` |
| [57:31:06](https://youtu.be/5JTBS09e7ew?t=207066) | Docker image |
| [57:40:45](https://youtu.be/5JTBS09e7ew?t=207645) | Installing, then versions ([57:41:55](https://youtu.be/5JTBS09e7ew?t=207715)) |
| [57:43:05](https://youtu.be/5JTBS09e7ew?t=207785) | **MFA with the CLI** |
| [57:44:41](https://youtu.be/5JTBS09e7ew?t=207881) | **Environment variables and precedence** |
| [57:47:58](https://youtu.be/5JTBS09e7ew?t=208078) | Autocompletion tools, then auto-prompt ([57:49:22](https://youtu.be/5JTBS09e7ew?t=208162)) |

## Notes

### Command shape and flags

```text
aws <command> <subcommand> [options]     aws ec2 describe-subnets --region us-east-1 --output table
```

- No short flags: `-o` does not exist. The order of flags does not matter.
- `--flag value` or `--flag=value`. The `=` is required when the value starts with a hyphen.
- Quote values that contain spaces or JSON. On Linux: single quotes outside, double quotes inside.
- Global options include `--profile`, `--region`, `--output`, `--query`, `--no-paginate`, `--endpoint-url` and `--debug`.

### Return codes (`echo $?` in bash)

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | An S3 transfer failed (`aws s3` commands only) |
| 2 | Command could not be parsed; for `aws s3`, files were skipped |
| 130 | Interrupted with Ctrl+C |
| 252 | Invalid syntax or unknown parameter |
| 253 | Environment or configuration is invalid, e.g. no credentials |
| 254 | The request was sent and the service returned an error |
| 255 | Any other failure |

### Output formats

| `--output` | Use |
|------------|-----|
| `json` | The default. For jq and for saving. |
| `yaml`, `yaml-stream` | Easier to read. The stream variant prints large results as they arrive. |
| `text` | Tab-separated, no quotes. For one value into a variable. |
| `table` | ASCII table for people, not for scripts |

Set the default with `output =` in `config` or `AWS_DEFAULT_OUTPUT`.

### Pagination

| Flag | Effect |
|------|--------|
| none | The CLI keeps calling the API and prints all pages joined together |
| `--page-size N` | Items requested per API call. Same output, more and smaller calls. Use it when a big listing times out. |
| `--max-items N` | Stops after N items and prints a `NextToken` |
| `--starting-token T` | Continues from that `NextToken` |
| `--no-paginate` | One API call only, so only the first page |

- High-level `aws s3 ls` has only `--page-size`. Low-level `aws s3api list-objects-v2` has all of them.
- Limits differ per API. More calls can cost more on services billed per request.

### `--filters` versus `--query`

| | `--filters` | `--query` |
|---|-------------|-----------|
| Runs on | The server | Your machine, after the response arrives |
| Syntax | `Name=...,Values=...`, names defined per API | JMESPath |
| Available on | Some commands; may be spelled `--filter` or `--filter-expression` | Every command |
| Less data sent | Yes | No |
| Job | Choose which resources come back | Choose which fields are shown |

```bash
aws ec2 describe-instances \
  --filters "Name=instance-state-name,Values=running" "Name=instance-type,Values=t3.micro" \
  --query 'Reservations[].Instances[].[InstanceId,PrivateIpAddress]' --output table
aws ec2 describe-vpcs --query 'Vpcs[?IsDefault==`true`].VpcId' --output text
```

- Several values in one filter mean OR. Several filters mean AND. Names and values are case-sensitive.
- Literal values in a JMESPath filter go in backticks, with the whole query in single quotes.

| JMESPath | Result |
|----------|--------|
| `Vpcs[0].VpcId` | One field of the first item |
| `Vpcs[].VpcId` | That field from every item |
| `Vpcs[].[VpcId,CidrBlock]` | Several fields per item |
| `Vpcs[].{ID:VpcId,CIDR:CidrBlock}` | Several fields with your own names |
| `Subnets[].SubnetId \| [0:5]` | Pipe into a second expression; here the first five |

### Waiters

```bash
ID=$(aws ec2 run-instances --image-id ami-0abc1234 --instance-type t3.micro \
       --query 'Instances[0].InstanceId' --output text)
aws ec2 wait instance-status-ok --instance-ids "$ID"    # silent; returns when ready
```

- Other EC2 waiters: `instance-running`, `instance-stopped`, `vpc-available`, `image-available`.
- AMI IDs differ per Region. When a resource "does not exist", check `--region` first.

### Passing values in

| Need | Syntax |
|------|--------|
| Parameter from a file | `file://policy.json` (relative), `file:///tmp/policy.json` (absolute), `file://~/policy.json` (home) |
| Binary file | `fileb://data.bin` |
| JSON inline | `--tags '[{"Key":"env","Value":"dev"}]'` |
| Shorthand for the same | `--tags Key=env,Value=dev`; list items are separated by spaces |
| Template of all parameters | `aws ec2 run-instances --generate-cli-skeleton > in.json` (no API call is made) |
| Run from the template | `--cli-input-json file://in.json` or `--cli-input-yaml file://in.yaml` |

- Delete the keys you do not set from a skeleton. Flags on the command line override the file.
- Parameter types: string, integer, boolean, timestamp (ISO 8601 or Unix epoch), list, blob.
- A few flags take a plain path with no `file://`. Read the reference page.
- PowerShell writes UTF-16 by default. Add `-Encoding ascii` to files that the CLI or Linux will read.

### Chaining commands

```bash
# 1. Command substitution into a variable
VPC_ID=$(aws ec2 describe-vpcs --filters Name=is-default,Values=true \
           --query 'Vpcs[0].VpcId' --output text)
aws ec2 create-subnet --vpc-id "$VPC_ID" --cidr-block 172.31.96.0/20

# 2. One line with xargs: the CLI reads arguments, not stdin, so a plain pipe is not enough
aws ec2 describe-vpcs --query 'Vpcs[0].VpcId' --output text |
  xargs -I {} aws ec2 describe-subnets --filters "Name=vpc-id,Values={}"
```

### The `config` and `credentials` files (in `~/.aws/`)

```text
# ~/.aws/credentials
[default]
aws_access_key_id = AKIA...
aws_secret_access_key = ...

# ~/.aws/config
[default]
region = us-east-1
output = json
retry_mode = standard
max_attempts = 5

[profile prod]                      # the word "profile" is used only in config
role_arn = arn:aws:iam::222222222222:role/Admin
source_profile = default            # or: credential_source = Ec2InstanceMetadata
mfa_serial = arn:aws:iam::111111111111:mfa/long
```

- Choose a profile with `--profile prod` or `export AWS_PROFILE=prod`. Otherwise `default` is used.
- Keys belong in `credentials`, everything else in `config`. If both hold the same setting, `credentials` wins.
- **Role profile**: the CLI calls STS AssumeRole for you and uses the temporary credentials. `source_profile` names a profile that has keys. `credential_source` (`Environment`, `Ec2InstanceMetadata`, `EcsContainer`) replaces it when no keys are in a file.
- **MFA**: with `mfa_serial` the CLI prompts "Enter MFA code". The role's trust policy enforces it with the `aws:MultiFactorAuthPresent` condition.
- `credential_process = /path/script` runs a program that prints credentials as JSON. AWS advises against it unless needed.

| Command | Does |
|---------|------|
| `aws configure` | Prompts for key, secret, Region and output |
| `aws configure set region us-west-2 --profile dev` | Writes one value; an empty string unsets it |
| `aws configure get region` | Prints one value |
| `aws configure list` | Shows the active settings and where each came from |
| `aws configure list-profiles` | Lists profile names |
| `aws configure import --csv file://creds.csv` | Imports the key file downloaded from the console |
| `aws configure sso` | Creates an IAM Identity Center profile; then `aws sso login --profile name` |

### Where the CLI looks for credentials (first match wins)

| Order | Source |
|-------|--------|
| 1 | Command-line options such as `--profile` |
| 2 | Environment variables |
| 3 | `credentials` and `config` files: keys, role profiles, SSO, `credential_process` |
| 4 | Container credentials (ECS task role) |
| 5 | EC2 instance profile, read from instance metadata |

- On an instance with a role attached, the CLI works with no files at all (step 5).
- Behind a corporate proxy, set `HTTPS_PROXY`. On EC2 also set `NO_PROXY=169.254.169.254` so metadata calls stay direct.
- Access keys, STS calls by hand, Signature Version 4 and endpoints are in [[FCC 28.1 - AWS API, STS and service endpoints]].

### Retries

| `retry_mode` | Total attempts by default | Notes |
|--------------|---------------------------|-------|
| `legacy` | 5 | Old behaviour, backoff factor 2 |
| `standard` | 3 | Default in CLI v2. Backoff factor 2, capped at 20 seconds, retries more error types. |
| `adaptive` | Same as standard | Adds client-side rate limiting; he calls it experimental |

Set `retry_mode` and `max_attempts` in `config`, or `AWS_RETRY_MODE` and `AWS_MAX_ATTEMPTS`. `max_attempts` includes the first call.

### Environment variables

| Variable | Sets |
|----------|------|
| `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_SESSION_TOKEN` | Credentials; the token is needed for temporary ones |
| `AWS_REGION`, `AWS_DEFAULT_REGION` | Region |
| `AWS_PROFILE` | Profile |
| `AWS_DEFAULT_OUTPUT` | Output format |
| `AWS_CONFIG_FILE`, `AWS_SHARED_CREDENTIALS_FILE` | Location of the two files |
| `AWS_RETRY_MODE`, `AWS_MAX_ATTEMPTS` | Retries |
| `AWS_EC2_METADATA_DISABLED` | Stops the CLI asking instance metadata for credentials |
| `AWS_PAGER` | Program that displays output; set it to empty in scripts |

### Work tools, not exam material

| Tool | What to keep |
|------|--------------|
| jq | `aws ec2 describe-vpcs \| jq '.Vpcs[] \| {id: .VpcId, cidr: .CidrBlock}'`. Leading `.` is the root, `[]` iterates, `[0]` is the first item, `?` hides errors. |
| yq | `yq -p json -o yaml in.json > out.yaml`; also reads and writes XML |
| base64 | `echo -n 'text' \| base64` and `base64 --decode`. Three bytes become four characters: `Man` is `TWFu`. |
| Aliases | `~/.aws/cli/alias`: first line `[toplevel]`, then `dsub = ec2 describe-subnets --output table` (no `aws`). Start the value with `!` to run a bash function that takes arguments. |
| Wizards | `aws iam wizard new-role`, `aws configure wizard`; also dynamodb, events, lambda. They need a default Region. |
| Docker | `docker run --rm -it -v ~/.aws:/root/.aws amazon/aws-cli s3 ls`. Add `-v $(pwd):/aws` to read or write local files. |
| Auto-prompt | `export AWS_CLI_AUTO_PROMPT=on-partial`. F3 docs panel, F5 output preview, Ctrl+R history. The panels work badly in CloudShell and Gitpod. |
| Install and version | Use the AWS installer, not brew or another package manager. `aws --version` should print `aws-cli/2.x`. |

### Not in the video

| Point | Detail |
|-------|--------|
| Waiter timeout | A waiter polls at a fixed interval and gives up with return code 255. `ec2 wait instance-running` checks every 15 seconds, 40 times. |
| `--query` and API calls | `--query` does not reduce the number of API calls. Use `--filters` or `--max-items` for that. |
| `--dry-run` | EC2 commands accept it to test permissions without making the change |

## Memory hooks

- Filter is the shop assistant who fetches only what you asked for. Query is you picking things out of the bag at home.
- `--page-size` is the size of each bite. `--max-items` is the size of the meal.
- Credential order runs from your keyboard outwards: flag, environment, file, container, instance.
- 252, 253, 254: you typed it wrong, your setup is wrong, AWS said no.

## Exam traps

- "Listing thousands of resources times out": lower `--page-size`. It does not change what is returned.
- "Return only the first N results": `--max-items`, then `--starting-token` with the `NextToken`.
- "Reduce the data the API returns": `--filters`. `--query` only trims what is displayed.
- "The instance role is correct but the CLI acts as another identity": keys in environment variables or `~/.aws/credentials` override the instance profile.
- "A script must continue only when the instance or stack is ready": `aws ec2 wait ...` or `aws cloudformation wait stack-create-complete`.
- "Use a role in another account without calling STS by hand": a profile with `role_arn` and `source_profile`.

## Outdated or wrong in the video

- **"The config files are in TOML format."** They are INI-style files. Very sure.
- **Junk keys "should" return 253.** 253 means credentials or configuration are missing. Keys that exist but are invalid reach the service, which rejects them, so 254 is right. Very sure.
- **"On EC2 you will find a credentials file with `credential_source`."** No file is created. The CLI falls back to instance metadata by itself. `credential_source` is written only when a profile must assume a second role. Very sure.
- **`AWS_PAGER` "is for pagination".** It names the program (`less` by default) that v2 sends output to. It has nothing to do with API pages. Very sure.
- **"The CLI is written in Python, so have Python 3."** v2 bundles its own Python; only v1 uses the system one. Very sure.
- **"Some commands only support some output formats."** `--output` is a global option. The exception is the high-level `aws s3` commands, which print their own text. Fairly sure.
- **He wonders why v1 still exists.** v1 entered maintenance mode on 15 July 2026 and reaches end of support on 15 July 2027. Fairly sure.
- **Sign-in options.** Since CLI 2.32 (November 2025), `aws login` signs the CLI in with your console sign-in and issues short-lived credentials. Fairly sure.

Related: [[FCC 28.1 - AWS API, STS and service endpoints]] · [[FCC 21 - Identity and Access Management IAM]] · [[FCC 02 - Amazon EC2]] · [[FCC 09.1 - S3 basics, CLI and IaC tooling]] · [[FCC 29 - AWS CloudFormation]] · [[FCC 00 - Watch guide]]
