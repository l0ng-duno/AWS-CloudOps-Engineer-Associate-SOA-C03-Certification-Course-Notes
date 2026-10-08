# FCC 38 - AWS Transfer Family

> Video chapter [64:53:15](https://youtu.be/5JTBS09e7ew?t=233595) to 66:10:28 · 1h17m total · about 2 min lecture, 75 min lab
> Fast path: read this note and skip the video. The lab is one 75-minute build in which about 15 minutes concern Transfer Family; the rest is typing a VPC and IAM role by hand, clicking through Windows Server, and teardown. For the exam you need the "what is it" table and the endpoint and identity options.

## Summary

- Transfer Family is a managed file transfer server. Partners keep using SFTP, FTPS, FTP or AS2, and the files land in S3 or EFS.
- You run no servers. You choose protocols, an endpoint type, an identity provider and a storage back end.
- Each user (or Active Directory group) is mapped to an IAM role and a home directory. The role decides what the user can reach.
- Managed workflows run steps on every uploaded file: copy, tag, delete, decrypt, or a custom Lambda step.
- The lab logs in with an Active Directory user. It works, but it is the hardest way to do it.

## Labs and demos

| Time | Len | What it shows | Takeaway | Verdict |
|------|-----|---------------|----------|---------|
| [64:55:40](https://youtu.be/5JTBS09e7ew?t=233740) | 75m | An internet-facing SFTP server whose users live in a Simple AD directory, with S3 as storage. He follows his co-founder's written steps, mostly by CLI in CloudShell. | An upload from FileZilla reaches the bucket at the end. Around 45 minutes go on a VPC, IAM role and security groups typed one command at a time, with many typos. Around 15 minutes go on Windows Server clicks. The Transfer Family lessons are in the table below. | Skip |

### Useful moments inside the lab

| Time | What you learn |
|------|----------------|
| [64:58:57](https://youtu.be/5JTBS09e7ew?t=233937) | VPC, four subnets, internet gateway and routes by CLI (10 min). Nothing new; `--availability-zone-id use1-az1` is not the same flag as `--availability-zone us-east-1a`. |
| [65:10:12](https://youtu.be/5JTBS09e7ew?t=234612) | Simple AD options: size, DNS name, NetBIOS name, admin password, two subnets. Then `aws ds create-directory`; three minutes are lost because the shorthand needs commas: `VpcId=...,SubnetIds=a,b`. |
| [65:16:26](https://youtu.be/5JTBS09e7ew?t=234986) | `aws transfer create-server` with endpoint type `VPC` and identity provider `AWS_DIRECTORY_SERVICE` |
| [65:19:29](https://youtu.be/5JTBS09e7ew?t=235169) | Server is online. To give it fixed public IPs he stops it, allocates two Elastic IPs, attaches them with `update-server`, and starts it again. He also notices SFTP can listen on port 2222. |
| [65:23:35](https://youtu.be/5JTBS09e7ew?t=235415) | IAM role and instance profile for the Windows instance (8 min, mostly typos). Needs `AmazonSSMManagedInstanceCore` and `AmazonSSMDirectoryServiceAccess`. |
| [65:31:51](https://youtu.be/5JTBS09e7ew?t=235911) | Security groups: RDP 3389 from his own IP only (`/32`); port 22 from anywhere on the VPC's default group, which the server endpoint uses |
| [65:40:57](https://youtu.be/5JTBS09e7ew?t=236457) | Launches Windows Server 2022 with "Domain join directory" set, so it joins the directory at boot |
| [65:43:02](https://youtu.be/5JTBS09e7ew?t=236582) | RDP login as the directory admin: `AD\Administrator`. A forward slash fails; it must be a backslash. |
| [65:46:04](https://youtu.be/5JTBS09e7ew?t=236764) | Installs the Remote Server Administration Tools (AD DS tools, DNS tools) to manage the directory |
| [65:48:05](https://youtu.be/5JTBS09e7ew?t=236885) | Creates group `FTP Users` and user `test.a`, then adds the user to the group (4 min, slowed by an Edge pop-up) |
| [65:53:10](https://youtu.be/5JTBS09e7ew?t=237190) | Reads the group's SID in PowerShell with `wmic group where "name like 'FTP Users'"` |
| [65:55:11](https://youtu.be/5JTBS09e7ew?t=237311) | S3 bucket, plus an IAM policy and role for the Transfer service, made in the console |
| [65:59:19](https://youtu.be/5JTBS09e7ew?t=237559) | On the server, "Add access": paste the SID, pick the role, pick the bucket as home directory |
| [66:02:23](https://youtu.be/5JTBS09e7ew?t=237743) | FileZilla: user `test.a` alone does not connect; `test.a@ad.exampro.co` on port 22 does. Uploads one image. |
| [66:05:25](https://youtu.be/5JTBS09e7ew?t=237925) | Teardown (5 min). The VPC cannot be deleted until the directory has finished deleting, because its network interfaces are still there. |

## Lecture index

| Time | Topic |
|------|-------|
| [64:53:15](https://youtu.be/5JTBS09e7ew?t=233595) | The four protocols and their ports (the first sentence is a few seconds before the chapter mark) |
| [64:54:25](https://youtu.be/5JTBS09e7ew?t=233665) | Managed File Transfer Workflows; warning that the lab is hard |

## Notes

### What it is, when it is the answer, what it is confused with

| Service | Use it when |
|---------|-------------|
| Transfer Family | Outside parties must push or pull files with SFTP, FTPS, FTP or AS2, and you want the data in S3 or EFS |
| DataSync | You move or sync large data sets between storage systems yourself, on a schedule |
| Storage Gateway (File Gateway) | On-premises servers need an NFS or SMB share backed by S3 |
| Snow Family | The data is too big for the network; a device is shipped |
| S3 presigned URL | One person needs one object for a short time |

### Protocols

| Protocol | What it is | On Transfer Family |
|----------|------------|--------------------|
| SFTP | File transfer over SSH | Port 22. The only protocol allowed on a public endpoint. |
| FTPS | FTP with TLS | Port 21, data ports 8192 to 8200. VPC endpoint only. Needs an ACM certificate. |
| FTP | Old, no encryption | VPC internal endpoint only, never internet-facing |
| AS2 | Business-to-business document exchange over HTTP | VPC endpoint only |

### How the lab fits together

```text
SFTP client --port 22--> Elastic IPs --> Transfer server (VPC endpoint, two public subnets)
                                           | checks the user in the directory (two private subnets)
                                           | finds the user's AD group by SID
                                           | assumes the IAM role mapped to that group
                                           v
                                        S3 bucket (home directory)
```

```bash
aws transfer create-server --endpoint-type VPC \
  --identity-provider-type AWS_DIRECTORY_SERVICE \
  --identity-provider-details DirectoryId=d-0123456789 \
  --endpoint-details VpcId=vpc-0abc,SubnetIds=subnet-pub1,subnet-pub2
aws transfer stop-server --server-id s-0123456789abcdef0
aws transfer update-server --server-id s-0123456789abcdef0 \
  --endpoint-details AddressAllocationIds=eipalloc-0a,eipalloc-0b
aws transfer start-server --server-id s-0123456789abcdef0
```

- The server must be stopped before Elastic IPs are attached.
- A user must belong directly to exactly one group that has access.
- The IAM role must trust `transfer.amazonaws.com`.
- AWS documents `Get-ADGroup` in PowerShell for reading the SID; he uses the older `wmic`.

### Not in the video

| Choice | Options | Remember |
|--------|---------|----------|
| Identity provider | Service managed (SSH keys, SFTP only); AWS Directory Service; custom (Lambda or API Gateway) | Service managed is the simple default. He mentions at the end that a Lambda would have been easier. |
| Endpoint type | Public; VPC internal; VPC internet-facing | Public has no security group and its IPs can change. VPC internet-facing uses Elastic IPs, so partners can allow-list them. |
| Storage | S3 or EFS | Chosen per server |
| Logging | CloudWatch Logs | Shows who uploaded what |

## Memory hooks

- Transfer Family is "an SFTP server with an S3 bucket behind it".
- Fixed IP needed? VPC endpoint plus Elastic IPs. Public endpoint means no security group.
- Plain FTP stays inside: internal only.
- User → role → home directory. The role is the permission.

## Exam traps

- "Partners must keep using SFTP, files must end up in S3, no servers to manage": Transfer Family.
- "The partner's firewall needs fixed IP addresses to allow": VPC endpoint, internet-facing, with Elastic IPs.
- "Only certain source IPs may connect": VPC endpoint with a security group. A public endpoint cannot do this.
- "Log in with existing corporate Active Directory accounts": Directory Service identity provider (AWS Managed Microsoft AD, or AD Connector for an on-premises directory).
- "Decrypt, scan or tag each file after upload": a managed workflow, with a Lambda step for custom work.
- "Sync a file server to S3 every night": DataSync, not Transfer Family.

## Outdated or wrong in the video

- **"FTPS is port 990."** That is implicit FTPS in general. A Transfer Family FTPS server listens on port 21, with data on ports 8192 to 8200. Very sure.
- **Simple AD as the identity provider.** It works in his demo, but the AWS documentation says Transfer Family does not support Simple AD. The supported choices are AWS Managed Microsoft AD and AD Connector. Fairly sure.
- **"Simple AD is only in us-east-1."** It is offered in six Regions: N. Virginia, Oregon, Ireland, Singapore, Sydney and Tokyo. Very sure.
- **"You can create two subnets with the same CIDR and it won't complain."** A VPC rejects a subnet whose range overlaps an existing one. Very sure.

Related: [[FCC 39 - AWS Directory Service]] · [[FCC 09.2 - S3 buckets and objects]] · [[FCC 11 - EFS]] · [[FCC 13 - Storage Gateway]] · [[FCC 15.1 - VPC core, NACLs, security groups and route tables]] · [[FCC 00 - Watch guide]]
