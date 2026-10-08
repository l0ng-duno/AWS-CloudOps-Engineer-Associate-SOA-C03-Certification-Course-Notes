# FCC 03 - Amazon Machine Images AMIs

> Video chapter [02:34:01](https://youtu.be/5JTBS09e7ew?t=9241) to 03:03:53 · 30 min total · about 19 min lecture, 11 min lab
> Fast path: read this note. If you want to see the two exam commands run, watch 5 minutes from [02:58:06](https://youtu.be/5JTBS09e7ew?t=10686).

## Summary

- An AMI is the template an instance boots from: a root volume template, launch permissions and a block device mapping.
- AMIs are Region-specific. The same image has a different AMI ID in every Region and for every architecture.
- Copying is the only way to move an AMI to another Region, and also the way to encrypt an unencrypted AMI.
- Deregistering an AMI does not delete its snapshots. You delete those yourself, or keep paying for them.

## Labs and demos

| Time | Len | What it shows | Takeaway | Verdict |
|------|-----|---------------|----------|---------|
| [02:52:49](https://youtu.be/5JTBS09e7ew?t=10369) | 11m | Launches an unencrypted instance, makes an AMI with `aws ec2 create-image`, then copies it from us-east-1 to ca-central-1 with `aws ec2 copy-image --encrypted` | The copy arrives encrypted in the new Region. He then deregisters both AMIs and deletes the leftover snapshots by hand. The first 5 minutes are a tour of the AMI picker and Marketplace. | Skim |

## Lecture index

| Time | Topic |
|------|-------|
| [02:34:01](https://youtu.be/5JTBS09e7ew?t=9241) | Older recording: what an AMI is, Marketplace, creating and copying (6 min, repeated by the next item) |
| [02:40:10](https://youtu.be/5JTBS09e7ew?t=9610) | Current lecture: everything in this note (12 min) |

## Notes

### What an AMI holds

| Part | Meaning |
|------|---------|
| Root volume template | An EBS snapshot or an instance store template: OS, packages, your application |
| Launch permissions | Which AWS accounts may launch it |
| Block device mapping | Which volumes are attached at launch |

### Operations

| Action | What it does | Remember |
|--------|--------------|----------|
| Create | Makes an AMI from a running or stopped instance | Console: Actions → Image and templates → Create image |
| Copy | Duplicates an AMI, optionally to another Region | The way to go cross-Region and the way to encrypt |
| Store / restore | Saves the AMI to an S3 bucket and brings it back | Used to move between AWS partitions |
| Deregister | The AMI "delete"; no new launches | Snapshots stay behind |
| Deprecate | Sets a date after which people should stop using it | For shared or Marketplace AMIs |
| Disable | Blocks launches, can be re-enabled | A soft delete |

### Sharing

| Setting | Who can launch |
|---------|----------------|
| Implicit | Only the owner (default) |
| Explicit | Named accounts, organizations or OUs |
| Public | Everyone; it becomes a community AMI |

### Properties to recognise

| Property | Options | Note |
|----------|---------|------|
| Root device type | EBS-backed, instance store-backed | EBS-backed storage survives a stop; instance store data does not persist |
| Boot mode | Legacy BIOS, UEFI, UEFI preferred | UEFI supports Secure Boot and volumes over 2 TB |
| Virtualization | HVM, PV | HVM is the modern default and faster; PV is legacy |
| ENA | Enabled or not | Enhanced networking up to 100 Gbps; all Nitro instances use it |

### Where AMIs fit

- Launch templates reference an AMI, so updating the template rolls a new AMI out to an Auto Scaling group.
- Systems Manager Automation or EC2 Image Builder can patch and re-bake AMIs on a schedule.
- Marketplace AMIs come from vendors, often with an hourly fee on top of EC2, such as CIS hardened images.

```bash
aws ec2 create-image --instance-id i-0123456789abcdef0 --name my-ami-000
aws ec2 copy-image --source-region us-east-1 --source-image-id ami-0123456789abcdef0 \
  --name my-copied-ami --region ca-central-1 --encrypted
```

## Memory hooks

- AMI = "save the whole recipe"; EBS snapshot = "save one disk".
- Copy does two jobs that both start with "cross": cross-Region and cross into encryption.
- Deregister removes the label and leaves the boxes (snapshots) in storage.

## Exam traps

- "Use the same AMI ID in another Region" never works. Copy the AMI first.
- "Encrypt an existing unencrypted AMI": copy it with encryption enabled.
- Storage charges continuing after an AMI is deregistered point to leftover snapshots.
- "Share with one specific account" means editing launch permissions, not making the AMI public.

## Outdated or wrong in the video

- **Launch configurations** are presented as how Auto Scaling groups use AMIs. They are legacy and take no new instance types; use launch templates. He notices this himself at the end of the lab. Very sure.
- **"Stop an instance store-backed instance and you lose the data."** Instance store-backed instances cannot be stopped at all, only rebooted or terminated. Very sure.
- **He is unsure whether EBS volumes are encrypted by default.** They are not, unless you turn on the account-level "EBS encryption by default" setting for that Region. Very sure.

Related: [[01 - EC2 & EBS]] · [[FCC 02 - Amazon EC2]] · [[FCC 04 - EC2 Image Builder]] · [[FCC 05 - EBS]] · [[FCC 20 - ASG]] · [[FCC 00 - Watch guide]]
