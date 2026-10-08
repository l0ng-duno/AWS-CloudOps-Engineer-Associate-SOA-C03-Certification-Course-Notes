# FCC 09.5 - S3 replication, versioning and lifecycle

> Video section [23:39:40](https://youtu.be/5JTBS09e7ew?t=85180) to 25:48:05 (part 5 of 5 of the S3 chapter) · 2h08m total · about 30 min lecture, 1h38m labs
> Fast path: read this note, then watch the last 6 minutes of the replication lab (from [24:01:09](https://youtu.be/5JTBS09e7ew?t=86469)) and the lifecycle lab (7 min). The versioning lab is optional; its row lists the 20 minutes worth seeing. These three features are the most exam-relevant part of S3 for an operations role.

## Summary

- Versioning keeps every version of an object. A normal delete only adds a delete marker; the data is still there.
- Versioning can be suspended but never switched off again.
- Replication copies new objects to another bucket, in the same or another Region. It needs versioning on both buckets and an IAM role.
- Lifecycle rules move objects to cheaper classes and expire them on a schedule. Transitions only go "down" the list of classes.
- S3 is strongly consistent: a read straight after a write returns the new data.

## Labs and demos

| Time | Len | What it shows | Takeaway | Verdict |
|------|-----|---------------|----------|---------|
| [23:50:56](https://youtu.be/5JTBS09e7ew?t=85856) | 17m | Cross-Region replication with the CLI: buckets in `ca-central-1` and `us-east-1`, versioning on both, an IAM policy and role, `put-bucket-replication` | Nothing fails. The first 10 minutes are setup: buckets, versioning, JSON copied from the docs, the role (commands under "Replication"). A new upload reaches the destination within a minute. Deleting it in the source leaves the copy, because the rule sets `DeleteMarkerReplication` to `Disabled`; he does not make that link. Deleting the buckets fails until the console **Empty** button has removed all versions. | Skim: [24:01:09](https://youtu.be/5JTBS09e7ew?t=86469) to 24:07:19 |
| [24:14:41](https://youtu.be/5JTBS09e7ew?t=87281) | 72m | Versioning end to end in CloudShell (enable on an existing bucket, read versions, delete, restore), then MFA Delete, then suspended versioning | The versioning half works with small stumbles. Enabling MFA Delete takes 21 minutes of failed attempts (24:50:12 to 25:11:33). It only worked with root access keys, an MFA device on the root user, and a CLI outside CloudShell. | Skim: [24:40:01](https://youtu.be/5JTBS09e7ew?t=88801) to 24:50:12 and [25:15:37](https://youtu.be/5JTBS09e7ew?t=90937) to 25:25:55 |
| [25:37:55](https://youtu.be/5JTBS09e7ew?t=92275) | 7m | Lifecycle rule with `put-bucket-lifecycle-configuration` that moves objects to Standard-IA | Rejected three times: `Days` must be a number, the rule needs a filter, and a transition to Standard-IA must be 30 days or more. | **Watch** |

### Useful moments inside the versioning lab

| Time | What you learn |
|------|----------------|
| [24:23:32](https://youtu.be/5JTBS09e7ew?t=87812) | `aws s3 ls` and `list-objects` show no version data; `list-object-versions` does |
| [24:26:35](https://youtu.be/5JTBS09e7ew?t=87995) | Turning versioning on for an existing bucket. The object already there keeps version ID `null` ([24:28:40](https://youtu.be/5JTBS09e7ew?t=88120)); only the next upload gets a real ID |
| [24:40:01](https://youtu.be/5JTBS09e7ew?t=88801) | Reading one version with `get-object --version-id`; `null` works as an ID. `aws s3 cp` cannot pick a version |
| [24:41:02](https://youtu.be/5JTBS09e7ew?t=88862) | A delete without a version ID returns a delete marker with its own version ID; all three versions are still listed |
| [24:43:04](https://youtu.be/5JTBS09e7ew?t=88984) | After the delete, `get-object` returns "NoSuchKey" although every version still exists |
| [24:47:08](https://youtu.be/5JTBS09e7ew?t=89228) | Restoring by downloading an old version and uploading it again. The delete marker stays in the history |
| [24:50:12](https://youtu.be/5JTBS09e7ew?t=89412) | MFA Delete starts. He registers an MFA device on his IAM user, which turns out to be useless |
| [24:58:19](https://youtu.be/5JTBS09e7ew?t=89899) | Error "DevPay and Mfa are mutually exclusive authorization methods". Misleading: it means the caller is not the root user |
| [25:00:21](https://youtu.be/5JTBS09e7ew?t=90021) | He works out that only root can enable it. As root in CloudShell it still fails, even with the MFA device moved to the root user ([25:04:27](https://youtu.be/5JTBS09e7ew?t=90267)) |
| [25:08:32](https://youtu.be/5JTBS09e7ew?t=90512) | The fix: root access keys in another terminal. The command works at [25:11:33](https://youtu.be/5JTBS09e7ew?t=90693) |
| [25:15:37](https://youtu.be/5JTBS09e7ew?t=90937) | With MFA Delete on, an IAM user's plain delete still works, because it only adds a marker |
| [25:17:42](https://youtu.be/5JTBS09e7ew?t=91062) | Deleting a specific version is refused: "Mfa Authentication must be used for this request". With `--mfa` added it works, for root only ([25:19:44](https://youtu.be/5JTBS09e7ew?t=91184)) |
| [25:21:46](https://youtu.be/5JTBS09e7ew?t=91306) | Turning it off: CLI only. `Status=Suspended,MFADelete=Disabled` plus `--mfa` does both at once. An expired code is rejected |
| [25:23:51](https://youtu.be/5JTBS09e7ew?t=91431) | With versioning suspended, new uploads get version `null` and overwrite each other |

## Lecture index

| Time | Topic |
|------|-------|
| [23:39:40](https://youtu.be/5JTBS09e7ew?t=85180) | Data consistency |
| [23:41:22](https://youtu.be/5JTBS09e7ew?t=85282) | Replication overview, CRR, SRR, bidirectional, batch |
| [23:49:55](https://youtu.be/5JTBS09e7ew?t=85795) | Replication options: RTC, metrics, delete markers |
| [24:07:59](https://youtu.be/5JTBS09e7ew?t=86879) | Versioning, adding and deleting versions |
| [24:13:29](https://youtu.be/5JTBS09e7ew?t=87209) | MFA Delete |
| [25:26:52](https://youtu.be/5JTBS09e7ew?t=91612) | Lifecycle: transitions, expiry, restrictions |
| [25:36:15](https://youtu.be/5JTBS09e7ew?t=92175) | Anatomy of a lifecycle configuration |
| [25:45:01](https://youtu.be/5JTBS09e7ew?t=92701) | Transfer Acceleration |
| [25:46:28](https://youtu.be/5JTBS09e7ew?t=92788) | Presigned URLs |

## Notes

### Versioning

| State | Behaviour |
|-------|-----------|
| Unversioned | The default; overwrites and deletes are final |
| Enabled | Every write creates a new version ID. Objects uploaded before it was enabled keep version ID `null` |
| Suspended | New writes get version `null` and replace the previous `null` version; older versions are kept |

| Action | Result in a versioned bucket |
|--------|------------------------------|
| GET without a version ID | The current version |
| DELETE without a version ID | Adds a delete marker; GET now returns 404 |
| DELETE with a version ID | Permanently removes that version |
| Undelete | Delete the delete marker, or copy an old version back on top |

### MFA Delete

- An optional extra on a versioned bucket.
- Required to permanently delete a version or to change the versioning state. A plain delete (marker only) does not need it.
- Only the root user can turn it on, and only through the API or CLI, not the console.
- In the lab it needed an MFA device on the root user, root access keys, and a CLI outside CloudShell. Delete the root keys afterwards.
- `--mfa` takes the device ARN, a space, then the current six-digit code.

```bash
aws s3api put-bucket-versioning --bucket BUCKET \
  --versioning-configuration Status=Enabled,MFADelete=Enabled \
  --mfa "arn:aws:iam::ACCOUNT_ID:mfa/DEVICE_NAME 123456"
```

### Replication

| Type | Meaning | Typical reason |
|------|---------|----------------|
| Cross-Region (CRR) | To a bucket in another Region | Disaster recovery, compliance, lower latency |
| Same-Region (SRR) | To a bucket in the same Region | Log aggregation, a copy for test, data that must stay in one Region |
| Bidirectional | A rule in each direction | Failover and back, usually with a Multi-Region Access Point |
| Batch Replication | On demand, for objects that already exist or failed | Back-filling after turning replication on |

| Rule | Detail |
|------|--------|
| Versioning | Must be enabled on source and destination |
| Permissions | An IAM role that S3 assumes to read the source and write the destination |
| What is copied | Only objects created after the rule exists |
| Cross-account | Supported |
| CRR or SRR | Decided only by where the destination bucket is; the rule names no Region |
| Delete markers | Not replicated unless you enable it; markers made by lifecycle rules are never replicated |
| Replication Time Control | 99.99% of objects within 15 minutes, for a fee, with metrics |
| Replication metrics | Objects pending (count and size), maximum replication time, failures; CloudWatch charges apply |

The lab's steps, after creating two buckets and enabling versioning on both:

```bash
aws iam create-policy --policy-name NAME --policy-document file://policy.json
aws iam create-role --role-name NAME --assume-role-policy-document file://trust.json
aws iam attach-role-policy --role-name NAME --policy-arn POLICY_ARN
aws s3api put-bucket-replication --bucket SOURCE --replication-configuration file://replication.json
```

- All three JSON files are copied from the docs. `replication.json` holds the role ARN and one rule: status, priority, delete marker replication, filter, destination bucket ARN.

### Lifecycle rules

| Action type | Examples |
|-------------|----------|
| Transition | Move current or noncurrent versions to another storage class after N days |
| Expiration | Delete current versions, permanently delete noncurrent versions, remove expired delete markers, abort incomplete multipart uploads |

Transition order, top to bottom only:

```text
Standard → Standard-IA → Intelligent-Tiering → One Zone-IA
        → Glacier Instant Retrieval → Glacier Flexible Retrieval → Glacier Deep Archive
```

| Constraint | Detail |
|------------|--------|
| Direction | Never back up the list, and never to Standard |
| Before a move to Standard-IA or One Zone-IA | The object must be at least 30 days old |
| Small objects | Objects under 128 KB are not transitioned by default |
| Filters | Prefix, object tags, object size |
| Scope | Applies to existing objects as well as new ones; up to 1,000 rules per bucket |
| Expiry in a versioned bucket | Adds a delete marker; a separate rule removes noncurrent versions |

```json
{ "Rules": [ { "ID": "move-to-ia", "Status": "Enabled", "Filter": { "Prefix": "" },
    "Transitions": [ { "Days": 30, "StorageClass": "STANDARD_IA" } ],
    "Expiration": { "Days": 365 } } ] }
```

### Transfer Acceleration and presigned URLs

| Feature | What it does | Remember |
|---------|--------------|----------|
| Transfer Acceleration | Uploads enter AWS at the nearest CloudFront edge location and travel on the AWS network | Endpoint `bucket.s3-accelerate.amazonaws.com`; bucket name cannot contain dots; up to 20 minutes to activate |
| Presigned URL | A time-limited link that lets someone download or upload one private object | `aws s3 presign s3://bucket/key --expires-in 300`; it carries the signer's permissions |

## Memory hooks

- Delete in a versioned bucket puts a "closed" sign on the door. The shop is still full.
- Replication needs two things on both sides and one in the middle: versioning, versioning, and a role.
- Lifecycle is a staircase you can only walk down.

## Exam traps

- "Protect against accidental deletion": versioning. Add MFA Delete for stronger protection; only the root user can enable it, from the CLI or API.
- "Storage cost keeps growing on a versioned bucket": add a lifecycle rule to expire noncurrent versions.
- "Existing objects were not replicated": expected; use Batch Replication.
- "Cannot delete a bucket that looks empty": it still holds old versions and delete markers.
- "Give a user temporary access to one private object without an AWS account": presigned URL.
- "Speed up uploads from users far from the bucket's Region": Transfer Acceleration.

## Outdated or wrong in the video

- **"Batch Replication has no CLI or API."** It is an S3 Batch Operations job and can be created with `aws s3control create-job`. Fairly sure.
- **Download and re-upload is "the only way" to restore a version, and no command removes a delete marker ([24:44:06](https://youtu.be/5JTBS09e7ew?t=89046)).** `delete-object --version-id` with the marker's ID removes it, which is the quickest undelete. `copy-object` also takes `--copy-source "bucket/key?versionId=ID"`. Very sure.
- **His notes say a GET on a deleted object returns a 403 ([24:45:06](https://youtu.be/5JTBS09e7ew?t=89106)).** It returns 404 NoSuchKey, as his own terminal had just shown. Very sure.
- **He is unsure whether the console can create presigned URLs.** It can, for downloads, valid for up to 12 hours. Fairly sure.
- **Consistency explanation.** The example mixes up waiting time and staleness. The fact to keep is simple: since December 2020, S3 gives strong read-after-write consistency for all operations. Very sure.

Related: [[FCC 09.2 - S3 buckets and objects]] · [[FCC 09.3 - S3 storage classes]] · [[FCC 09.4 - S3 security and encryption]] · [[FCC 17 - CloudFront]] · [[FCC 00 - Watch guide]]
