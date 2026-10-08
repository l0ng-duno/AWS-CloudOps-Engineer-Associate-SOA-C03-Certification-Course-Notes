# FCC 05 - EBS

> Video chapter [04:30:23](https://youtu.be/5JTBS09e7ew?t=16223) to 05:16:02 · 46 min total · about 21 min lecture, 24 min labs
> Fast path: read this note, then watch the 4-minute resize lab. Skip the rest.

## Summary

- EBS is persistent block storage for EC2. A volume lives in one Availability Zone and is replicated inside it.
- Pick the volume type from the numbers: gp for general use, io for more than 16,000 IOPS, st1 and sc1 for cheap throughput.
- After growing a volume you must also grow the partition and the file system inside the instance.
- Encryption is chosen at creation. To encrypt an existing volume, snapshot it and copy the snapshot with encryption on.
- Snapshots are how volumes move between AZs and Regions.

## Labs and demos

| Time | Len | What it shows | Takeaway | Verdict |
|------|-----|---------------|----------|---------|
| [04:50:55](https://youtu.be/5JTBS09e7ew?t=17455) | 4m | Grows a root volume from 8 GiB to 12 GiB with `aws ec2 modify-volume`, then extends it inside the instance | `lsblk` shows the disk at 12G but the partition still at 8G until he runs `growpart` and `xfs_growfs`. | **Watch** |
| [04:55:10](https://youtu.be/5JTBS09e7ew?t=17710) | 4m | Creates an encrypted volume with `Encrypted: true` in CloudFormation, then starts the snapshot-copy route for an existing volume | He stops at the "Encrypt this snapshot" checkbox and does not finish. | Skip |
| [04:59:21](https://youtu.be/5JTBS09e7ew?t=17961) | 16m | Multi-Attach: one io2 volume attached to two instances in the same AZ | Eleven minutes are CloudFormation errors: wrong folder, a typo, missing `Iops`, an AMI ID from another Region. The actual demonstration starts at [05:10:59](https://youtu.be/5JTBS09e7ew?t=18659). | Skip |

## Lecture index

| Time | Topic |
|------|-------|
| [04:30:23](https://youtu.be/5JTBS09e7ew?t=16223) | IOPS, throughput, bandwidth; list of volume types |
| [04:32:22](https://youtu.be/5JTBS09e7ew?t=16342) | Volume type comparison |
| [04:36:42](https://youtu.be/5JTBS09e7ew?t=16602) | Hard disk drives and RPM |
| [04:38:21](https://youtu.be/5JTBS09e7ew?t=16701) | RAID levels |
| [04:40:19](https://youtu.be/5JTBS09e7ew?t=16819) | SSDs, SATA, NVMe, form factors (hardware background, not exam content) |
| [04:45:20](https://youtu.be/5JTBS09e7ew?t=17120) | Moving volumes between AZs and Regions |
| [04:46:15](https://youtu.be/5JTBS09e7ew?t=17175) | EBS vs instance store, MBR vs GPT |
| [04:49:21](https://youtu.be/5JTBS09e7ew?t=17361) | Logical block addressing |
| [05:15:14](https://youtu.be/5JTBS09e7ew?t=18914) | Data Lifecycle Manager |

## Notes

### Terms

| Term | Meaning |
|------|---------|
| IOPS | Small read and write operations per second |
| Throughput | Data moved per second, in MiB/s |
| Bandwidth | The maximum possible rate: the pipe, where throughput is the water |

### SSD volumes

| | gp2 | gp3 | io1 | io2 Block Express |
|---|-----|-----|-----|-------------------|
| Use | General, boot | General, boot (cheaper than gp2) | I/O-heavy databases | Sub-millisecond latency, largest databases |
| Size | 1 GiB to 16 TiB | 1 GiB to 16 TiB | 4 GiB to 16 TiB | 4 GiB to 64 TiB |
| Max IOPS | 16,000 | 16,000 | 64,000 | 256,000 |
| Max throughput | 250 MiB/s | 1,000 MiB/s | 1,000 MiB/s | 4,000 MiB/s |
| Durability | 99.8 to 99.9% | 99.8 to 99.9% | 99.8 to 99.9% | 99.999% |
| Multi-Attach | No | No | Yes | Yes |

### HDD volumes

| | st1 (throughput optimized) | sc1 (cold) | standard (magnetic) |
|---|----------------------------|------------|---------------------|
| Use | Big data, data warehouses, logs | Infrequently accessed, lowest cost | Previous generation |
| Size | 125 GiB to 16 TiB | 125 GiB to 16 TiB | 1 GiB to 1 TiB |
| Max IOPS | 500 | 250 | 40 to 200 |
| Max throughput | 500 MiB/s | 250 MiB/s | 40 to 90 MiB/s |
| Boot volume | No | No | Yes |

### RAID

| Level | Name | Min disks | Survives |
|-------|------|-----------|----------|
| 0 | Striping | 2 | No failure; speed and capacity only |
| 1 | Mirroring | 2 | One disk |
| 5 | Striping with parity | 3 | One disk |
| 6 | Striping with double parity | 4 | Two disks |
| 10 | Mirror plus stripe | 4 | One disk per mirror |

### EBS vs instance store

| | EBS volume | Instance store volume |
|---|------------|-----------------------|
| Location | Network-attached, independent of the host | Disks physically attached to the host |
| Stop the instance | Allowed, data persists | Not possible for instance store-backed instances |
| Data lost when | You delete the volume | The instance is terminated or the host fails |
| Use for | Anything that must persist | Caches, buffers, temporary data |

### MBR vs GPT

| | MBR | GPT |
|---|-----|-----|
| Firmware | BIOS | UEFI |
| Partitions | 4 primary | 128 |
| Max disk size | 2 TiB | Far larger |

### Common tasks

```bash
# grow a volume, then grow the partition and the XFS file system
aws ec2 modify-volume --volume-id vol-0123456789abcdef0 --size 12
lsblk
sudo growpart /dev/nvme0n1 1
sudo xfs_growfs -d /
```

| Task | Steps |
|------|-------|
| Move to another AZ | Snapshot → create a volume from it in the target AZ |
| Move to another Region | Snapshot → copy the snapshot to the Region → create a volume |
| Encrypt an existing volume | Snapshot → copy the snapshot with encryption → create a volume |

### Other points

- **Multi-Attach**: io1 and io2 only, all instances in the same AZ.
- **Data Lifecycle Manager**: automates creating, keeping and deleting EBS snapshots and EBS-backed AMIs. Policy types: snapshot, AMI, and cross-account copy event.
- **Block size**: the default 4 KiB block gives a 16 TiB maximum volume, which is why 16 TiB keeps appearing.
- **io2**: every io2 volume created after 21 November 2023 is io2 Block Express.

## Memory hooks

- 16 and 64: gp tops out at 16,000 IOPS, io1 at 64,000. Above that it is io2 Block Express.
- **s**t1 = **s**treaming throughput, s**c**1 = **c**old and **c**heap.
- RAID 0 has zero redundancy. RAID 1 is one copy mirrored.
- Resize is three steps: volume, partition, file system.

## Exam traps

- "Volume was resized but the OS still shows the old size": extend the partition and file system.
- st1 and sc1 cannot be boot volumes.
- An EBS volume cannot attach to an instance in a different AZ.
- "More than 16,000 IOPS" rules out gp2 and, for exam purposes, gp3.
- Instance store data survives a reboot and is lost on stop, terminate or host failure.

## Outdated or wrong in the video

- **gp3 limits.** AWS raised gp3 in late 2025 to 64 TiB, 80,000 IOPS and 2,000 MiB/s. The table above shows the video's figures, which older exam questions still assume. Fairly sure.
- **Moving a volume "requires creating an AMI".** It does not. A snapshot is enough: create the volume from it in the new AZ, or copy the snapshot to the new Region. The AMI step is only needed when moving a whole bootable instance. Very sure.
- **Multi-Attach lab formats the shared volume with XFS and mounts it on both instances.** That only appears to work. Ordinary file systems are not cluster-aware, and writing from two instances can corrupt data; Multi-Attach needs a clustered file system or an application that coordinates writes. Very sure.

Related: [[01 - EC2 & EBS]] · [[02 - Amazon Data Life Cycle Manager]] · [[FCC 02 - Amazon EC2]] · [[FCC 03 - Amazon Machine Images AMIs]] · [[FCC 11 - EFS]] · [[FCC 00 - Watch guide]]
