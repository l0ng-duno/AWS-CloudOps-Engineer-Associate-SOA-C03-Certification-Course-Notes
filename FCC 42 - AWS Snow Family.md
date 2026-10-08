# FCC 42 - AWS Snow Family

> Video chapter [66:39:30](https://youtu.be/5JTBS09e7ew?t=239970) to 66:50:24 · 11 min total · 9 min lecture, 2 min console demo
> Fast path: read this note and skip the video. Most of what it describes has been withdrawn (see the last section). Keep the idea, "ship the data on a device when the network is too slow", and the three size classes.

## Summary

- Snow devices move data into or out of AWS physically, for when the network is too slow, too costly or not there.
- Three sizes in the video: **Snowcone** (terabytes, fits in a backpack), **Snowball Edge** (tens to hundreds of terabytes, also runs EC2 instances) and **Snowmobile** (a truck, up to 100 PB).
- Data is encrypted with 256-bit keys and lands in S3. Jobs are import or export.
- Rule of thumb from his example: 100 TB over the internet can take more than 100 days; a Snowball takes under a week.
- This is the last chapter of the video.

## Labs and demos

| Time | Len | What it shows | Takeaway | Verdict |
|------|-----|---------------|----------|---------|
| [66:48:31](https://youtu.be/5JTBS09e7ew?t=240511) | 2m | Clicks through the "order a Snow Family device" form for a Snowcone and stops before the address | Steps: job name, import or export, device, NFS transfer, optional EC2 compute and IoT Greengrass, bucket, OpsHub management, encryption key, shipping. A Snowcone ships without power supply or network cable. | Skip |

## Lecture index

| Time | Topic |
|------|-------|
| [66:39:30](https://youtu.be/5JTBS09e7ew?t=239970) | Original Snowball (older slide): why ship data, security features, sizes |
| [66:41:08](https://youtu.be/5JTBS09e7ew?t=240068) | Snowball Edge: clustering, protocols, device options |
| [66:42:56](https://youtu.be/5JTBS09e7ew?t=240176) | Snow Family overview |
| [66:44:22](https://youtu.be/5JTBS09e7ew?t=240262) | Snowcone |
| [66:46:45](https://youtu.be/5JTBS09e7ew?t=240405) | Comparison table |

## Notes

### The family as the video describes it

| | Snowcone | Snowball Edge | Snowmobile |
|---|----------|---------------|------------|
| Capacity | 8 TB HDD or 14 TB SSD | 80 TB HDD, or 210 TB NVMe (storage optimized); 28 TB NVMe (compute optimized) | 100 PB |
| vCPUs | 2 | 40, or 104 on the newer models | None |
| Weight | 2.1 kg (4.5 lb) | About 50 lb | A shipping container |
| Runs EC2 instances | Yes, small ones | Yes; a GPU option exists | No |
| Clustering | No | 3 to 16 nodes | No |
| Extra | Battery power, Wi-Fi, DataSync built in, IoT Greengrass | NFS and S3 interfaces | Driven to your data centre |

### Features common to the devices

- Tamper-resistant and weatherproof, with a Trusted Platform Module chip.
- An E Ink shipping label, so no box or paper label is needed.
- 256-bit encryption end to end.
- A job must be finished within 90 days of the device being prepared.
- Snowcone can send data back in two ways: ship the device, or run DataSync over the network.

### Not in the video

| Topic | Fact |
|-------|------|
| Keys | Encryption keys are managed in KMS and never stored on the device |
| After return | AWS erases the device once the import is complete |
| Glacier | Data is imported to S3 first. Use a lifecycle rule to move it to a Glacier storage class. |
| Management | OpsHub is the graphical tool for unlocking and managing devices |
| Online alternative | DataSync over the internet or Direct Connect, when the link is good enough |

## Memory hooks

- Cone, ball, mobile: backpack, suitcase, truck.
- If the upload would take more than about a week, ship it.

## Exam traps

- "Move tens of terabytes to S3 from a site with a slow link": Snowball Edge, in questions written before 2026.
- "Ongoing, repeated transfers over a good network link": DataSync, not Snow.
- "Process data at a remote site with no connectivity, then ship it": Snowball Edge compute optimized.
- "Data must end up in Glacier": import to S3, then a lifecycle rule.

## Outdated or wrong in the video

- **Snowmobile.** Retired in 2024. Very sure.
- **Snowcone.** Discontinued in November 2024, together with the older Snowball Edge models (80 TB storage optimized, 52-vCPU compute optimized, GPU). Very sure.
- **Snowball Edge.** Closed to new customers since November 2025. AWS points new customers to DataSync and AWS Data Transfer Terminal for moving data, and to Outposts for computing at the edge. Fairly sure.
- **Original Snowball, 50 TB and 80 TB.** Retired long ago, as he says himself. The first two minutes are an old slide. Very sure.

Related: [[FCC 09.2 - S3 buckets and objects]] · [[FCC 10 - Amazon S3 Glacier]] · [[FCC 13 - Storage Gateway]] · [[FCC 38 - AWS Transfer Family]] · [[FCC 00 - Watch guide]]
