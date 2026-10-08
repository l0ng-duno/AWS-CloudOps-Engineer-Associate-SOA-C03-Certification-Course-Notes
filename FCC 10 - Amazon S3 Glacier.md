# FCC 10 - Amazon S3 Glacier

> Video chapter [25:48:05](https://youtu.be/5JTBS09e7ew?t=92885) to 26:12:24 · 24 min total · all lecture, no labs
> Fast path: read this note and skip the chapter. It is an older recording about the original Glacier "vault" service. The exam mostly asks about the S3 Glacier storage classes, which are in [[FCC 09.3 - S3 storage classes]].

## Summary

- The original Glacier stores **archives** in **vaults**. You cannot browse a vault; you ask for an inventory and wait.
- Everything you read from a vault is an asynchronous **job**: start it, wait, then download the output. SNS can tell you when it is done.
- Three retrieval speeds: Expedited (1 to 5 min), Standard (3 to 5 h), Bulk (5 to 12 h). Faster costs more.
- **Vault Lock** makes a vault write once, read many (WORM). After the 24-hour test window the policy can never be changed.
- Each archive carries 32 KB of overhead, so store a few large files, not many small ones.

## Labs and demos

None. He says he will show API, CLI and SDK examples, but they are slides only.

## Lecture index

| Time | Topic |
|------|-------|
| [25:48:07](https://youtu.be/5JTBS09e7ew?t=92887) | What Glacier is |
| [25:49:03](https://youtu.be/5JTBS09e7ew?t=92943) | Use case: keep financial records for seven years |
| [25:50:11](https://youtu.be/5JTBS09e7ew?t=93011) | Security, data model (vault, archive, job), job operations |
| [25:55:51](https://youtu.be/5JTBS09e7ew?t=93351) | Vault inventory, archive retrieval, Glacier Select |
| [26:00:24](https://youtu.be/5JTBS09e7ew?t=93624) | Retrieval tiers, getting data in, deleting a vault |
| [26:05:24](https://youtu.be/5JTBS09e7ew?t=93924) | Vault access policy |
| [26:06:07](https://youtu.be/5JTBS09e7ew?t=93967) | Vault Lock and the policy format |
| [26:09:05](https://youtu.be/5JTBS09e7ew?t=94145) | Data retrieval policies and provisioned capacity |
| [26:11:01](https://youtu.be/5JTBS09e7ew?t=94261) | Archive overhead |

## Notes

### Data model

| Term | Meaning |
|------|---------|
| Vault | Container for archives, in one Region |
| Archive | One stored item (a file or a zip), up to 40 TB, immutable |
| Job | An asynchronous request: `inventory-retrieval`, `archive-retrieval` or `select` |
| Vault inventory | List of archives in a vault; refreshed about once a day |
| Notification configuration | One per vault; publishes to SNS when a job finishes |

### Reading from a vault

```bash
aws glacier initiate-job --account-id - --vault-name my-vault \
  --job-parameters '{"Type": "inventory-retrieval"}'
aws glacier describe-job --account-id - --vault-name my-vault --job-id <id>
aws glacier get-job-output --account-id - --vault-name my-vault --job-id <id> out.json
```

- The first inventory is ready about a day after the first upload.
- An archive retrieval can ask for a byte range instead of the whole archive.

### Retrieval tiers

| Tier | Time | Notes |
|------|------|-------|
| Expedited | 1 to 5 minutes | Archives up to 250 MB; most expensive |
| Standard | 3 to 5 hours | The default |
| Bulk | 5 to 12 hours | Cheapest; for large amounts |

### Getting data in

| Way | Notes |
|-----|-------|
| S3 lifecycle rule | The usual way: objects move from a bucket to a Glacier storage class |
| Multipart upload through the API, CLI or SDK | There is no console upload for vaults |
| Snow devices | For very large offline transfers (see the outdated list) |

### Policies

| Policy | What it does |
|--------|--------------|
| Vault access policy | Resource policy, one per vault: who may do what |
| Vault Lock policy | Compliance rules such as "deny delete for 365 days"; immutable once locked |
| Data retrieval policy | Per Region spending guard: No Retrieval Limit (default), Free Tier Only, or Max Retrieval Rate |

Locking a vault:

| Step | Detail |
|------|--------|
| 1. Initiate | Attaches the policy and returns a lock ID; state is "in progress" |
| 2. Test | You have 24 hours to check that it behaves as intended |
| 3. Complete or abort | Complete with the lock ID to make it permanent; otherwise abort, or let it expire |

### Other facts

- **Encryption**: AES-256 at rest by default, TLS in transit.
- **Provisioned capacity**: guarantees Expedited retrievals are available. One unit is 3 expedited retrievals every 5 minutes and up to 150 MB/s.
- **Deleting a vault**: it must hold no archives as of the last inventory, with no writes since then.
- **Overhead**: 32 KB of index and metadata is billed per archive.

## Memory hooks

- Glacier is a storage locker with a clerk: you fill in a request (job), wait, and collect.
- Vault Lock is a time-lock safe with a 24-hour trial period.
- Expedited, Standard, Bulk: minutes, hours, half a day.

## Exam traps

- "Records must not be altered or deleted for seven years": Vault Lock, or S3 Object Lock in compliance mode for objects in a bucket.
- "Cheapest archive, retrieval within 12 hours is fine": Glacier Deep Archive, which is a storage class, not this vault service.
- "Need archived data within minutes": Expedited tier, with provisioned capacity if it must be guaranteed.
- "Thousands of small log files are costly in Glacier": combine them into one archive first.

## Outdated or wrong in the video

- **The whole chapter describes the vault-based service.** AWS now steers everyone to the S3 Glacier storage classes, and the standalone vault service stopped accepting new customers in late 2025. Fairly sure.
- **"Data must sit 30 days in S3 before moving to Glacier."** There is no such wait for Glacier transitions. The 30-day rule applies to transitions into Standard-IA and One Zone-IA. Fairly sure.
- **Snowball and Snowmobile as upload routes.** Snowmobile is retired and the Snow family is closed to new customers. Fairly sure.
- **Glacier Select.** Closed to new customers since 2024, like S3 Select. Fairly sure.
- **"API, CLI and the CDK."** He means the SDK. Very sure.

Related: [[FCC 09.3 - S3 storage classes]] · [[FCC 09.5 - S3 replication, versioning and lifecycle]] · [[FCC 42 - AWS Snow Family]] · [[FCC 00 - Watch guide]]
