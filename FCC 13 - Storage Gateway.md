# FCC 13 - Storage Gateway

> Video chapter [26:48:28](https://youtu.be/5JTBS09e7ew?t=96508) to 27:08:51 · 20 min total · 9 min lecture, 11 min lab
> Fast path: read this note. Watch the lab only if you want to see a file written to an NFS share turn into an S3 object (about 4 minutes from [26:57:49](https://youtu.be/5JTBS09e7ew?t=97069)).

## Summary

- Storage Gateway is a virtual appliance you run on premises. Local apps talk to it with ordinary storage protocols, and it stores the data in AWS.
- **File Gateway**: a shared folder (NFS or SMB) whose files become S3 objects.
- **Volume Gateway**: a remote disk over iSCSI. **Cached** keeps the data in S3 with a local cache; **stored** keeps the data on premises and backs it up to AWS.
- **Tape Gateway**: a virtual tape library for existing backup software, archived to Glacier.
- The gateway runs on VMware ESXi, Hyper-V, KVM, a hardware appliance, or an EC2 instance.

## Labs and demos

| Time | Len | What it shows | Takeaway | Verdict |
|------|-----|---------------|----------|---------|
| [26:51:55](https://youtu.be/5JTBS09e7ew?t=96715) | 11m | Creates an S3 File Gateway on an EC2 instance from the console, adds an NFS file share backed by a new bucket, and mounts it from a second instance | A file created on the share appears in S3 after a short delay. A file uploaded straight to S3 does not appear on the share. | Skim |

### Useful moments inside the lab

| Time | What you learn |
|------|----------------|
| [26:53:44](https://youtu.be/5JTBS09e7ew?t=96824) | Activating the gateway by IP address, with CloudWatch logging options |
| [26:54:46](https://youtu.be/5JTBS09e7ew?t=96886) | A 150 GB cache disk is attached automatically; the security group already opens NFS port 2049 |
| [26:57:49](https://youtu.be/5JTBS09e7ew?t=97069) | `sudo mount -t nfs -o nolock,hard <gateway-ip>:/<bucket> <dir>` (fails without `sudo`) |
| [26:58:50](https://youtu.be/5JTBS09e7ew?t=97130) | Upload direct to S3: not visible on the share |
| [26:59:51](https://youtu.be/5JTBS09e7ew?t=97191) | File created on the share: visible in S3 after a wait |

## Lecture index

| Time | Topic |
|------|-------|
| [26:48:31](https://youtu.be/5JTBS09e7ew?t=96511) | Overview of the gateway types |
| [26:50:04](https://youtu.be/5JTBS09e7ew?t=96604) | S3 File Gateway |
| [27:02:35](https://youtu.be/5JTBS09e7ew?t=97355) | FSx File Gateway |
| [27:03:45](https://youtu.be/5JTBS09e7ew?t=97425) | Volume Gateway: stored and cached volumes |
| [27:06:40](https://youtu.be/5JTBS09e7ew?t=97600) | Tape Gateway |

## Notes

### Gateway types

| Type | Looks like | Protocol | Data ends up in |
|------|------------|----------|-----------------|
| S3 File Gateway | Shared folder | NFS, SMB | S3 objects, one per file |
| FSx File Gateway | Windows share | SMB | FSx for Windows File Server |
| Volume Gateway | Disk | iSCSI | S3, restorable as EBS snapshots |
| Tape Gateway | Tape library | iSCSI VTL | S3, then Glacier when a tape is archived |

### Cached or stored volumes

| | Cached volumes | Stored volumes |
|---|----------------|----------------|
| Primary copy of the data | S3 | On premises |
| Kept locally | Only recently used data | Everything |
| Sent to AWS | All data | Asynchronous point-in-time snapshots |
| Volume size | Up to 32 TiB | 1 GiB to 16 TiB |
| Choose it when | You want to shrink on-premises storage | You need low-latency access to the whole dataset |

### Other facts

- Objects written through S3 File Gateway are normal S3 objects: versioning, lifecycle rules, replication and bucket policies all apply.
- Objects added to the bucket directly only appear on the share after a cache refresh (`RefreshCache`).
- Volume snapshots are incremental and stored as EBS snapshots.
- Tape Gateway works with existing backup software such as Veeam, NetBackup and Backup Exec.

## Memory hooks

- File Gateway is a shared folder. Volume Gateway is a remote drive. Tape Gateway is a robot tape library with no robot.
- **Cached** = cloud is the home, the cache is local. **Stored** = stored at your site, backed up to the cloud.

## Exam traps

- "On-premises apps need NFS or SMB access to data that should live in S3": S3 File Gateway.
- "Replace physical tapes without changing the backup software": Tape Gateway.
- "Keep all data on site for low latency, with off-site backups in AWS": Volume Gateway with stored volumes.
- "Reduce local storage but keep frequently used data fast": Volume Gateway with cached volumes.
- "Files uploaded to the bucket do not show on the file share": refresh the gateway cache.

## Outdated or wrong in the video

- **"Cached volumes can be between 1 gigabyte and 32 gigabytes."** A slip. The limit is 32 TiB per cached volume. Very sure.
- **"The direction is from the gateway to S3."** His conclusion after the direct upload did not appear. The share can show objects added in S3, but only after a cache refresh. Very sure.
- **FSx File Gateway.** Closed to new customers since late 2024, so it is unlikely to be the right answer in a new exam. Fairly sure.

Related: [[FCC 09.2 - S3 buckets and objects]] · [[FCC 10 - Amazon S3 Glacier]] · [[FCC 12 - FSx]] · [[FCC 05 - EBS]] · [[FCC 00 - Watch guide]]
