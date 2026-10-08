# FCC 12 - FSx

> Video chapter [26:37:17](https://youtu.be/5JTBS09e7ew?t=95837) to 26:48:28 · 11 min total · 4 min lecture, 7 min lab
> Fast path: read this note and skip the chapter. The only thing to memorise is which of the four FSx file systems fits which workload.

## Summary

- FSx is a family of managed file systems built on existing commercial or open-source products. EFS is AWS's own NFS service; FSx runs "someone else's file system for you".
- Four types: Windows File Server, Lustre, NetApp ONTAP, OpenZFS.
- **Windows File Server** is the one exams ask about most: SMB shares joined to Active Directory.
- **Lustre** is for high performance computing and machine learning, and can link to an S3 bucket.
- Amazon File Cache is a separate, temporary high-speed cache in front of data held on premises or in S3. It lives in the FSx console.

## Labs and demos

| Time | Len | What it shows | Takeaway | Verdict |
|------|-----|---------------|----------|---------|
| [26:41:03](https://youtu.be/5JTBS09e7ew?t=96063) | 7m | Creates an FSx for OpenZFS file system in the console (single AZ, 64 GiB minimum), launches an EC2 instance in the same AZ, mounts it over NFS and writes a file | Mounting is a plain `mount -t nfs` against the file system's DNS name. His first try fails because he pastes the EC2 instance's address instead. | Skip |

## Lecture index

| Time | Topic |
|------|-------|
| [26:37:17](https://youtu.be/5JTBS09e7ew?t=95837) | The four FSx file systems |
| [26:39:57](https://youtu.be/5JTBS09e7ew?t=95997) | Amazon File Cache |

## Notes

### The four file systems

| FSx for | Protocols | Clients | Pick it when |
|---------|-----------|---------|--------------|
| Windows File Server | SMB | Windows, also Linux and macOS | Windows shares, Active Directory, NTFS permissions, DFS |
| Lustre | Lustre | Linux | HPC, machine learning, media rendering; needs hundreds of GB/s and sub-millisecond latency |
| NetApp ONTAP | NFS, SMB, iSCSI | Windows, Linux, macOS | Moving existing NetApp storage to AWS; one file system for several protocols |
| OpenZFS | NFS | Linux, Windows, macOS | Moving ZFS or other Linux file servers to AWS; snapshots and clones |

### EFS or FSx

| Need | Answer |
|------|--------|
| Shared Linux storage, nothing to size, pay per GB used | EFS |
| Windows shares with Active Directory | FSx for Windows File Server |
| Fastest parallel file system for compute clusters | FSx for Lustre |
| Keep NetApp features such as SnapMirror | FSx for NetApp ONTAP |

### Other facts

- FSx file systems are provisioned: you choose storage capacity and throughput. EFS is elastic.
- Windows File Server offers Single-AZ and Multi-AZ deployments, on SSD or HDD storage.
- Lustre can present an S3 bucket as files and write results back to it.
- Mount command from the lab:

```bash
sudo mkdir /fsx
sudo mount -t nfs -o nfsvers=4.2 fs-0123456789abcdef0.fsx.us-east-1.amazonaws.com:/fsx /fsx
```

## Memory hooks

- **W**indows File Server = **W**indows and Active Directory. **L**ustre = **L**inux clusters, **l**ightning fast.
- EFS is elastic and Linux only. FSx is fixed size and "bring your favourite file system".

## Exam traps

- "Windows application needs a shared drive with Active Directory authentication": FSx for Windows File Server, never EFS.
- "HPC job needs to process data that lives in S3 with a POSIX file system": FSx for Lustre linked to the bucket.
- "File share must survive the loss of an AZ": choose a Multi-AZ FSx deployment.
- SMB means Windows; NFS means Linux. That one clue answers most file storage questions.

## Outdated or wrong in the video

- Nothing incorrect noticed. The lecture reads from the product pages and gives no detail on deployment types, backups or monitoring. Fairly sure.

Related: [[FCC 11 - EFS]] · [[FCC 13 - Storage Gateway]] · [[FCC 39 - AWS Directory Service]] · [[FCC 00 - Watch guide]]
