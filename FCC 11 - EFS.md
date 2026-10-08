# FCC 11 - EFS

> Video chapter [26:12:24](https://youtu.be/5JTBS09e7ew?t=94344) to 26:37:17 · 25 min total · 5 min lecture, 20 min lab
> Fast path: read this note, then watch the last 7 minutes of the lab from [26:30:18](https://youtu.be/5JTBS09e7ew?t=95418) (mount, write a file, read it from a second instance). The lecture leaves out storage classes and throughput modes, which the exam does ask about; they are in the table "Not in the video" below.

## Summary

- EFS is a managed NFS file system that many Linux instances can mount at the same time, across Availability Zones.
- It grows and shrinks on its own. You pay for the storage used, with nothing to provision.
- Instances reach it through a **mount target**: a network interface in a subnet, one per AZ, protected by a security group that allows NFS (TCP 2049).
- Install `amazon-efs-utils` to get the mount helper, which adds TLS and IAM support.
- Linux only. Windows instances cannot mount EFS.

## Labs and demos

| Time | Len | What it shows | Takeaway | Verdict |
|------|-----|---------------|----------|---------|
| [26:17:23](https://youtu.be/5JTBS09e7ew?t=94643) | 20m | Builds a file system, mount target and security group with CloudFormation, then mounts it from two EC2 instances through Session Manager | The mount fails until he installs `amazon-efs-utils` and creates the mount directory. A file written from instance 1 is visible from instance 2. | Skim |

### Useful moments inside the lab

| Time | What you learn |
|------|----------------|
| [26:22:07](https://youtu.be/5JTBS09e7ew?t=94927) | `AWS::EFS::FileSystem` properties: encryption and performance mode |
| [26:27:14](https://youtu.be/5JTBS09e7ew?t=95234) | Deploy fails with "unresolved" security group: the mount target needs `!GetAtt Sg.GroupId` in a list, not `!Ref` |
| [26:30:18](https://youtu.be/5JTBS09e7ew?t=95418) | Launches an instance in the same AZ as the mount target; the console's Attach button gives the mount commands |
| [26:32:20](https://youtu.be/5JTBS09e7ew?t=95540) | Mount helper missing on Amazon Linux 2023: `sudo yum install -y amazon-efs-utils`, then `sudo mkdir /mnt/efs` |
| [26:35:22](https://youtu.be/5JTBS09e7ew?t=95722) | Second instance mounts the same file system and sees `hello.txt` |

## Lecture index

| Time | Topic |
|------|-------|
| [26:12:24](https://youtu.be/5JTBS09e7ew?t=94344) | What EFS is, mount targets, pricing |
| [26:13:43](https://youtu.be/5JTBS09e7ew?t=94423) | EFS client (`amazon-efs-utils`), mount helper, NFS mount options |

## Notes

### Core facts

| Item | Detail |
|------|--------|
| Protocol | NFS v4.0 and v4.1 |
| Port | TCP 2049, opened in the mount target's security group |
| Scale | Petabytes; grows and shrinks automatically |
| Mount target | One per AZ, in a subnet you choose; gives a DNS name and an IP |
| Clients | EC2 Linux, ECS, EKS, Fargate, Lambda, on premises over VPN or Direct Connect |
| Not supported | Windows instances |

### Mounting

```bash
sudo yum install -y amazon-efs-utils
sudo mkdir /mnt/efs
sudo mount -t efs -o tls fs-0123456789abcdef0:/ /mnt/efs     # mount helper
sudo mount -t nfs4 -o nfsvers=4.1,hard,timeo=600,retrans=2,noresvport \
  fs-0123456789abcdef0.efs.us-east-1.amazonaws.com:/ /mnt/efs  # plain NFS client
```

To mount on every boot, add a line to `/etc/fstab`:

```text
fs-0123456789abcdef0:/ /mnt/efs efs _netdev,tls 0 0
```

`_netdev` tells Linux to wait for the network before mounting.

### Not in the video

These points are tested but not covered in this chapter.

| Topic | Options | Remember |
|-------|---------|----------|
| File system type | Regional, One Zone | One Zone is cheaper, with data in a single AZ |
| Storage classes | Standard, Infrequent Access, Archive | Lifecycle management moves files that have not been read for N days |
| Throughput mode | Elastic, Provisioned, Bursting | Bursting ties throughput to the amount stored and uses burst credits |
| Performance mode | General Purpose, Max I/O | General Purpose has the lowest latency and is the default |
| Access points | Entry points that enforce a POSIX user and a root directory | Useful for containers and Lambda |
| Protection | AWS Backup, EFS replication to another Region | Encryption at rest must be chosen at creation |

## Memory hooks

- EBS is one instance's hard disk. EFS is the office shared drive. S3 is a warehouse with a web address.
- EFS = **E**lastic, **F**or Linux, **S**hared.
- Port 2049 is NFS. If a mount hangs, look at the security group first.

## Exam traps

- "Shared file storage for many Linux instances in several AZs": EFS. For Windows the answer is FSx for Windows File Server.
- "Mount times out": the mount target's security group does not allow TCP 2049 from the instance, or there is no mount target in that AZ.
- "BurstCreditBalance is falling and throughput drops": switch from Bursting to Elastic or Provisioned throughput.
- "Reduce EFS cost for files nobody reads": lifecycle management to Infrequent Access.
- "Encrypt data in transit to EFS": mount with the helper and the `tls` option.

## Outdated or wrong in the video

- **Security group open to the whole internet.** He says himself it is not best practice. Allow port 2049 only from the instances' security group. Very sure.
- **"30 cents per GB a month."** That is the EFS Standard price in some Regions; Infrequent Access and Archive cost far less. Treat it as a rough figure. Fairly sure.

Related: [[FCC 05 - EBS]] · [[FCC 12 - FSx]] · [[FCC 40 - AWS Backup]] · [[FCC 29 - AWS CloudFormation]] · [[FCC 00 - Watch guide]]
