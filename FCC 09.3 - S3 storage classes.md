# FCC 09.3 - S3 storage classes

> Video section [20:00:45](https://youtu.be/5JTBS09e7ew?t=72045) to 20:35:39 (part 3 of 5 of the S3 chapter) · 35 min total · 30 min lecture, 5 min lab
> Fast path: read this note and skip the section. If you want one summary slide, watch the comparison matrix at [20:31:50](https://youtu.be/5JTBS09e7ew?t=73910) (4 min).

## Summary

- Storage classes trade storage price against availability, retrieval speed and retrieval fees. The class is set per object.
- Every current class is designed for eleven nines of durability. Availability is what differs.
- "IA" always means a retrieval fee and a 30-day minimum charge. "One Zone" means a single Availability Zone.
- Three Glacier classes: Instant Retrieval (milliseconds), Flexible Retrieval (minutes to hours), Deep Archive (hours).
- Intelligent-Tiering moves objects between tiers for you, for a monitoring fee.

## Labs and demos

| Time | Len | What it shows | Takeaway | Verdict |
|------|-----|---------------|----------|---------|
| [20:10:45](https://youtu.be/5JTBS09e7ew?t=72645) | 5m | Changes an object to Standard-IA with `aws s3 cp --storage-class STANDARD_IA` and checks it in the console | Works first time. A bucket has no default storage class; you set it on upload or with a lifecycle rule. | Skip |

## Lecture index

| Time | Topic |
|------|-------|
| [20:00:45](https://youtu.be/5JTBS09e7ew?t=72045) | Overview of all classes |
| [20:05:10](https://youtu.be/5JTBS09e7ew?t=72310) | Standard |
| [20:07:14](https://youtu.be/5JTBS09e7ew?t=72434) | Reduced Redundancy (legacy) |
| [20:08:44](https://youtu.be/5JTBS09e7ew?t=72524) | Standard-IA |
| [20:15:41](https://youtu.be/5JTBS09e7ew?t=72941) | Express One Zone |
| [20:17:27](https://youtu.be/5JTBS09e7ew?t=73047) | One Zone-IA |
| [20:19:41](https://youtu.be/5JTBS09e7ew?t=73181) | Glacier "vault" service vs Glacier storage classes |
| [20:22:49](https://youtu.be/5JTBS09e7ew?t=73369) | Glacier Instant Retrieval |
| [20:24:52](https://youtu.be/5JTBS09e7ew?t=73492) | Glacier Flexible Retrieval |
| [20:28:31](https://youtu.be/5JTBS09e7ew?t=73711) | Glacier Deep Archive |
| [20:29:51](https://youtu.be/5JTBS09e7ew?t=73791) | Intelligent-Tiering |
| [20:31:50](https://youtu.be/5JTBS09e7ew?t=73910) | Comparison matrix |

## Notes

### Comparison

| Class | Availability | AZs | Retrieval | Min storage charge | Retrieval fee |
|-------|--------------|-----|-----------|--------------------|---------------|
| Standard | 99.99% | 3 or more | Milliseconds | None | No |
| Intelligent-Tiering | 99.9% | 3 or more | Milliseconds (archive tiers slower) | None | No, but a monitoring fee |
| Express One Zone | 99.95% | 1 | Single-digit milliseconds | None | No |
| Standard-IA | 99.9% | 3 or more | Milliseconds | 30 days | Yes |
| One Zone-IA | 99.5% | 1 | Milliseconds | 30 days | Yes |
| Glacier Instant Retrieval | 99.9% | 3 or more | Milliseconds | 90 days | Yes |
| Glacier Flexible Retrieval | n/a until restored | 3 or more | Minutes to hours | 90 days | Yes |
| Glacier Deep Archive | n/a until restored | 3 or more | Hours | 180 days | Yes |

### When to pick each

| Class | Pick it when |
|-------|--------------|
| Standard | Data is accessed often, or you do not know the pattern and want no surprises |
| Intelligent-Tiering | Access patterns are unknown or changing and you want it handled automatically |
| Express One Zone | You need the lowest latency and can accept one AZ; uses directory buckets |
| Standard-IA | Accessed less than about once a month, but must be available instantly |
| One Zone-IA | Same, and the data can be recreated if the AZ is lost; about 20% cheaper than Standard-IA |
| Glacier Instant Retrieval | Accessed about once a quarter, still needs millisecond access |
| Glacier Flexible Retrieval | Archives where waiting minutes to hours is fine |
| Glacier Deep Archive | Long-term retention read once or twice a year; the cheapest storage |

### Retrieval tiers

| Tier | Flexible Retrieval | Deep Archive |
|------|--------------------|--------------|
| Expedited | 1 to 5 minutes | Not available |
| Standard | 3 to 5 hours | Within 12 hours |
| Bulk | 5 to 12 hours | Within 48 hours |

### Intelligent-Tiering tiers

| Tier | Object moves here after | Equivalent to |
|------|-------------------------|---------------|
| Frequent Access | Default | Standard |
| Infrequent Access | 30 days without access | Standard-IA |
| Archive Instant Access | 90 days without access | Glacier Instant Retrieval |
| Archive Access (opt-in) | 90 days or more | Glacier Flexible Retrieval |
| Deep Archive Access (opt-in) | 180 days or more | Glacier Deep Archive |

### Small-object overheads

- Standard-IA, One Zone-IA and Glacier Instant Retrieval bill every object as at least 128 KB.
- Glacier Flexible Retrieval and Deep Archive add about 40 KB of index and name data per object. Archive a few large files, such as one zip, not thousands of tiny ones.

### Glacier naming

- "S3 Glacier" was first a separate service that stores archives in vaults. It still exists.
- The S3 Glacier storage classes are part of S3. You use buckets and the normal S3 API, and no vault is involved.

## Memory hooks

- Minimum days: 30, 30, 90, 90, 180 (the two IA classes, the two faster Glaciers, Deep Archive).
- IA = "I'll access it rarely, and pay when I do".
- Deep Archive has no express lane: there is no Expedited tier.

## Exam traps

- "Lowest cost, retrieval within 12 hours is acceptable" points to Glacier Deep Archive.
- "Rarely accessed but must be retrievable immediately" points to Standard-IA or Glacier Instant Retrieval, never Flexible Retrieval.
- "Unknown or unpredictable access pattern" points to Intelligent-Tiering.
- "Reproducible data, infrequent access, lowest cost with instant access" points to One Zone-IA.
- Deleting or moving an object before its minimum duration still bills the full minimum.

## Outdated or wrong in the video

- **"Express One Zone costs 50% less than Standard."** That figure applies to request costs. Its storage price per GB is higher than Standard. Very sure.
- **"Deep Archive bulk retrieval takes the same time as standard."** Standard is within 12 hours and Bulk within 48 hours. Very sure.
- **Intelligent-Tiering shown with a 30-day minimum storage duration.** It has no minimum duration charge. He says he is unsure of this himself. Fairly sure.
- **"Standard-IA is 50% cheaper than Standard."** It is closer to 45%, so read it as "about half". Fairly sure.

Related: [[FCC 09.2 - S3 buckets and objects]] · [[FCC 09.5 - S3 replication, versioning and lifecycle]] · [[FCC 10 - Amazon S3 Glacier]] · [[FCC 00 - Watch guide]]
