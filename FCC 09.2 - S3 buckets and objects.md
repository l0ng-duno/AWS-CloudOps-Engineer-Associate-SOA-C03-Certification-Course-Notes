# FCC 09.2 - S3 buckets and objects

> Video section [11:56:20](https://youtu.be/5JTBS09e7ew?t=42980) to 20:00:45 (part 2 of 5 of the S3 chapter) · 8h04m total · about 1h20m lecture, the rest labs
> Fast path: read this note, then watch two labs: the Glacier restore (12 min) and the Object Lock behaviour from [19:21:40](https://youtu.be/5JTBS09e7ew?t=69700) (about 25 min). Skip the 2h23m event notification lab.

## Summary

- A bucket is infrastructure and an object is data, so objects get their own versions of tags, permissions and metadata.
- Bucket names follow DNS rules: 3 to 63 characters, lowercase letters, numbers, dots and hyphens.
- Archived objects (Glacier Flexible Retrieval, Deep Archive) must be restored before you can read them, and the restore is a temporary copy.
- Object Lock makes objects write-once-read-many. Compliance mode cannot be bypassed by anyone; governance mode can, with a special permission.
- Event notifications send bucket events to SNS, SQS, Lambda or EventBridge.

## Labs and demos

| Time | Len | What it shows | Takeaway | Verdict |
|------|-----|---------------|----------|---------|
| [11:57:25](https://youtu.be/5JTBS09e7ew?t=43045) | 12m | Uploads a file to the Glacier storage class, fails to download it, restores it with `s3api restore-object`, then downloads it | The first restore used the Standard tier (3 to 5 hours). He upgrades it to Expedited in the console and it is ready in minutes. | **Watch** |
| [12:31:45](https://youtu.be/5JTBS09e7ew?t=45105) | 23m | S3 Batch Operations: replaces tags on 20 objects from a CSV manifest | Ten minutes are lost because the manifest saved from Excel was tab-separated. The job needs an IAM role trusted by `batchoperations.s3.amazonaws.com` and waits for you to confirm before it runs. | Skip |
| [12:58:03](https://youtu.be/5JTBS09e7ew?t=46683) | 41m | S3 Select from a Ruby script on a CSV file | Mostly Ruby and SQL trial and error. The table is always called `S3Object`, and numbers must be `CAST` before comparing. | Skip |
| [13:42:35](https://youtu.be/5JTBS09e7ew?t=49355) | 5m | Adds a tag to an object with `s3api put-object-tagging` | Tags are passed as a `TagSet` list of key and value pairs. | Skip |
| [13:53:53](https://youtu.be/5JTBS09e7ew?t=50033) | 2h23m | Event notifications built in Terraform: one bucket sending to an SQS queue, an SNS topic and a Lambda function. A Ruby script reads the queue, a Ruby web server receives from the topic, and a Python function logs the event | All three work in the end. EventBridge is switched on but never used. Most of the time is Terraform, Bash and Ruby debugging. See "Useful moments" below. | Skip |
| [16:21:39](https://youtu.be/5JTBS09e7ew?t=58899) | 30m | Multipart upload by hand with `s3api`: create the upload, split a 50 MB file into five parts, upload each part, list parts, complete | Works at [16:50:14](https://youtu.be/5JTBS09e7ew?t=60614). The object is not visible in the bucket until the upload is completed. About half the lab is typos and hand-editing the parts JSON. The 2-minute lecture at [16:51:28](https://youtu.be/5JTBS09e7ew?t=60688) explains the same flow. | Skim: the moments below |
| [16:55:56](https://youtu.be/5JTBS09e7ew?t=60956) | 6m | Byte-range fetch: `s3api get-object --range bytes=0-4` returns just "hello" | Works first time. | Skip |
| [17:04:20](https://youtu.be/5JTBS09e7ew?t=61460) | 35m | Server access logging between two buckets, then querying the logs with Athena | Two commands: a bucket policy that lets `logging.s3.amazonaws.com` write, and `put-bucket-logging` on the source. He puts the policy on the source bucket and loses ten minutes to "Policy has an invalid resource"; the fix is at [17:18:51](https://youtu.be/5JTBS09e7ew?t=62331). Logs arrive after 30 to 60 minutes ([17:23:09](https://youtu.be/5JTBS09e7ew?t=62589)). Athena refuses to run until a results location in S3 is set ([17:29:20](https://youtu.be/5JTBS09e7ew?t=62960)). The log holds every request, including the console's own calls. | Skim: Athena from [17:27:16](https://youtu.be/5JTBS09e7ew?t=62836) (10 min) |
| [17:42:36](https://youtu.be/5JTBS09e7ew?t=63756) | 8m | DataSync task copying one S3 bucket to another in the console | The console generates the IAM roles for you. The first run copied nothing because the upload to the source bucket had failed on a misnamed file. | Skip |
| [17:56:20](https://youtu.be/5JTBS09e7ew?t=64580) | 18m | ETags in Terraform: `etag = filemd5(...)` on an object so Terraform notices when the file changes | A Terraform technique, not exam content. The point is at [18:11:12](https://youtu.be/5JTBS09e7ew?t=65472): `terraform plan` sees no change in an edited file until the `etag` line is added. Six minutes go on a missing Terraform CLI and bad credentials. | Skip |
| [18:16:10](https://youtu.be/5JTBS09e7ew?t=65770) | 23m | Checksums: compares `md5sum` with the ETag, then tries to upload with his own CRC32 and SHA-1 values | The MD5 matches the ETag. His own values are rejected by the CLI and the console. With the console field left blank, S3 calculates the checksum itself ([18:36:53](https://youtu.be/5JTBS09e7ew?t=67013)). He gives up without finding the cause. | Skip |
| [18:40:59](https://youtu.be/5JTBS09e7ew?t=67259) | 12m | Prefixes: creates a "folder" with `s3api put-object --key hello/` and no body, then builds a folder key exactly 1,024 characters long | That key is accepted. Adding `hello.txt` to it fails with "Your key is too long". `list-objects` returns the nested folders as one key. The limit is 1,024 bytes for the whole key; the number of slashes is not limited. | Skip |
| [18:57:08](https://youtu.be/5JTBS09e7ew?t=68228) | 8m | User-defined metadata with `put-object --metadata`, read back with `head-object` | The CLI adds the `x-amz-meta-` prefix and lowercases the key. | Skip |
| [19:10:01](https://youtu.be/5JTBS09e7ew?t=69001) | 38m | Object Lock: default governance retention, a compliance-mode object, and a legal hold | See "Useful moments" below. Setup and malformed-XML errors fill the first 11 minutes. The legal hold part has about seven minutes of wrong turns. The bucket cannot be deleted until the compliance object expires. | Skim from [19:21:40](https://youtu.be/5JTBS09e7ew?t=69700) |
| [19:54:30](https://youtu.be/5JTBS09e7ew?t=71670) | 6m | Uploads through the standard and the dual-stack endpoint with `--endpoint-url` | Both work. | Skip |

### Useful moments inside the event notification lab

| Time | What you learn |
|------|----------------|
| [14:03:10](https://youtu.be/5JTBS09e7ew?t=50590) | Turning on EventBridge delivery for a bucket is a single setting |
| [14:36:01](https://youtu.be/5JTBS09e7ew?t=52561) | S3 needs permission on each destination: an `aws_lambda_permission` for the function here, a queue policy at [14:06:14](https://youtu.be/5JTBS09e7ew?t=50774) |
| [14:57:33](https://youtu.be/5JTBS09e7ew?t=53853) | Several messages per upload (first seen at [14:52:26](https://youtu.be/5JTBS09e7ew?t=53546)). He blames the `s3:ObjectCreated:*` wildcard and narrows it to `s3:ObjectCreated:Put`. Doubtful; see "Outdated or wrong" |
| [15:02:41](https://youtu.be/5JTBS09e7ew?t=54161) | A notification body: a `Records` list with the event name, `s3.bucket.name` and `s3.object.key` |
| [15:25:18](https://youtu.be/5JTBS09e7ew?t=55518) | Reading an SQS message does not remove it. The script must call `delete_message` with the receipt handle |
| [15:48:33](https://youtu.be/5JTBS09e7ew?t=56913) | An HTTPS subscription must be confirmed: the endpoint has to call the `SubscribeURL` that SNS posts to it (done at [15:51:02](https://youtu.be/5JTBS09e7ew?t=57062)) |
| [15:56:07](https://youtu.be/5JTBS09e7ew?t=57367) | Through SNS, the S3 event arrives as a JSON string inside the `Message` field |
| [15:58:10](https://youtu.be/5JTBS09e7ew?t=57490) | No log group for the function: its role cannot write to CloudWatch Logs. Fixed with `logs:CreateLogGroup`, `logs:CreateLogStream` and `logs:PutLogEvents`; logs appear at [16:08:23](https://youtu.be/5JTBS09e7ew?t=58103) |
| [16:15:33](https://youtu.be/5JTBS09e7ew?t=58533) | `terraform destroy` fails on a bucket that still contains objects; he empties it in the console |

### Useful moments inside the multipart upload lab

| Time | What you learn |
|------|----------------|
| [16:26:52](https://youtu.be/5JTBS09e7ew?t=59212) | `create-multipart-upload` returns an `UploadId`; every later call needs it |
| [16:27:52](https://youtu.be/5JTBS09e7ew?t=59272) | Nothing shows in the console yet. `list-multipart-uploads` lists unfinished uploads |
| [16:39:03](https://youtu.be/5JTBS09e7ew?t=59943) | `upload-part --body` takes a plain file path, with no `file://` prefix |
| [16:40:03](https://youtu.be/5JTBS09e7ew?t=60003) | `list-parts` returns every part number and ETag |
| [16:47:10](https://youtu.be/5JTBS09e7ew?t=60430) | The file for `complete-multipart-upload`: `{"Parts": [{"PartNumber": 1, "ETag": "..."}]}`, passed as `file://parts.json` |

### Useful moments inside the Object Lock lab

| Time | What you learn |
|------|----------------|
| [19:13:48](https://youtu.be/5JTBS09e7ew?t=69228) | `put-object-lock-configuration` fails until versioning is on |
| [19:20:39](https://youtu.be/5JTBS09e7ew?t=69639) | A new upload inherits the bucket default: governance mode, one day |
| [19:21:40](https://youtu.be/5JTBS09e7ew?t=69700) | `aws s3 rm` on the locked object succeeds, because it only adds a delete marker |
| [19:27:47](https://youtu.be/5JTBS09e7ew?t=70067) | Deleting the version is denied, even for an admin, until `--bypass-governance-retention` is added |
| [19:30:53](https://youtu.be/5JTBS09e7ew?t=70253) | Compliance: `put-object` with `--object-lock-mode COMPLIANCE` and a retain-until date. The version cannot be deleted ([19:33:55](https://youtu.be/5JTBS09e7ew?t=70435)) |
| [19:34:57](https://youtu.be/5JTBS09e7ew?t=70497) | Legal hold: `put-object-legal-hold --legal-hold Status=ON` |
| [19:43:12](https://youtu.be/5JTBS09e7ew?t=70992) | Turning the hold off fails while the current version is a delete marker. Adding `--version-id` fixes it |
| [19:45:15](https://youtu.be/5JTBS09e7ew?t=71115) | Hold off, delete still denied: the default governance retention is on the same version, so the bypass flag is needed again |
| [19:46:18](https://youtu.be/5JTBS09e7ew?t=71178) | Console: Object Lock cannot be disabled once enabled. Only the default retention can be removed |

## Lecture index

| Time | Topic |
|------|-------|
| [11:56:20](https://youtu.be/5JTBS09e7ew?t=42980) | Buckets overview |
| [12:09:25](https://youtu.be/5JTBS09e7ew?t=43765) | Bucket naming rules (examples at [12:28:52](https://youtu.be/5JTBS09e7ew?t=44932)) |
| [12:12:25](https://youtu.be/5JTBS09e7ew?t=43945) | Access points, Multi-Region Access Points, Object Lambda, Mountpoint |
| [12:18:50](https://youtu.be/5JTBS09e7ew?t=44330) | Archived objects and restoring them |
| [12:23:12](https://youtu.be/5JTBS09e7ew?t=44592) | Requester Pays |
| [12:26:34](https://youtu.be/5JTBS09e7ew?t=44794) | Marketplace products for S3 |
| [12:30:39](https://youtu.be/5JTBS09e7ew?t=45039) | Batch Operations, Inventory |
| [12:56:05](https://youtu.be/5JTBS09e7ew?t=46565) | S3 Select; Select vs Athena at [13:39:27](https://youtu.be/5JTBS09e7ew?t=49167) |
| [13:41:23](https://youtu.be/5JTBS09e7ew?t=49283) | Object tags |
| [13:48:01](https://youtu.be/5JTBS09e7ew?t=49681) | Server access logs, event notifications |
| [13:50:56](https://youtu.be/5JTBS09e7ew?t=49856) | Bucket limits |
| [16:16:45](https://youtu.be/5JTBS09e7ew?t=58605) | Storage class analysis, Storage Lens |
| [16:51:28](https://youtu.be/5JTBS09e7ew?t=60688) | Multipart upload, byte-range fetching |
| [17:02:13](https://youtu.be/5JTBS09e7ew?t=61333) | Services that store data in S3 (EBS snapshots, RDS exports, CloudTrail, CloudWatch Logs exports, Firehose, Athena results) |
| [17:39:03](https://youtu.be/5JTBS09e7ew?t=63543) | Bucket types; DataSync at [17:41:34](https://youtu.be/5JTBS09e7ew?t=63694) |
| [17:50:43](https://youtu.be/5JTBS09e7ew?t=64243) | Folders, objects overview, ETags |
| [18:14:52](https://youtu.be/5JTBS09e7ew?t=65692) | Checksums |
| [18:39:48](https://youtu.be/5JTBS09e7ew?t=67188) | Prefixes |
| [18:53:17](https://youtu.be/5JTBS09e7ew?t=67997) | Metadata |
| [19:04:44](https://youtu.be/5JTBS09e7ew?t=68684) | WORM, Object Lock, legal holds |
| [19:47:42](https://youtu.be/5JTBS09e7ew?t=71262) | S3 URI, the four CLI command sets, request styles, dual-stack endpoints |

## Notes

### Bucket naming rules

| Rule | Detail |
|------|--------|
| Length | 3 to 63 characters |
| Characters | Lowercase letters, numbers, dots, hyphens |
| Start and end | A letter or a number |
| Not allowed | Uppercase, underscores, spaces, two dots in a row, the form of an IP address |
| Reserved | Prefixes `xn--` and `sthree-`; suffixes `-s3alias` and `--ol-s3` |
| Uniqueness | Global; a name is free again only after its bucket is deleted |

### Bucket types

| | General purpose | Directory |
|---|-----------------|-----------|
| Structure | Flat; folders are key prefixes | Real folder hierarchy |
| Storage classes | All except S3 Express One Zone | S3 Express One Zone (S3 One Zone-IA in Local Zones) |
| Use | Almost everything | Single-digit millisecond access |
| Default quota per account | 10,000 | 100 |

### DataSync

- Moves data to, from and between storage systems: NFS, SMB, HDFS and object storage; S3, EFS and FSx; other clouds such as Google Cloud Storage and Azure Blob Storage.
- A task is a source location, a destination location and options. It can run once or on a schedule.

### Access points family

| Feature | What it does |
|---------|--------------|
| Access point | A named endpoint on a bucket with its own policy and network setting (internet or one VPC). Splits one huge bucket policy into smaller ones. |
| Multi-Region Access Point | One global endpoint that routes to the nearest of several buckets. Usually paired with replication. |
| Object Lambda access point | Runs a Lambda function on GET, HEAD or LIST to transform what is returned, such as redacting personal data. The stored object is unchanged. |
| Mountpoint for S3 | Mounts a bucket as a Linux file system. Reads and creates files; cannot modify existing files or use file locking. |

### Archived objects and restore

| | Glacier Flexible Retrieval and Deep Archive | Intelligent-Tiering archive tiers |
|---|---------------------------------------------|-----------------------------------|
| How objects get there | You choose the storage class | Moved automatically |
| Restore | `restore-object` with a number of days; makes a temporary copy | `restore-object` with no options; moves the object back |
| Speed | You pick the tier: Expedited, Standard (default) or Bulk | Not selectable |
| Cost | You pay for the archive and the temporary copy | Monitoring fee |

### Requester Pays

- The bucket owner still pays for storage. The requester pays for requests and data transfer.
- Every request must be authenticated and must include the `x-amz-request-payer` header. Without it the request fails with 403.
- Anonymous access is not possible on a Requester Pays bucket.

### Bulk and query tools

| Tool | Purpose | Remember |
|------|---------|----------|
| Batch Operations | Run one action on millions of objects: copy, tag, restore, invoke Lambda, set Object Lock | Needs a manifest: a CSV of bucket and key, or an S3 Inventory report |
| S3 Inventory | A scheduled list (daily or weekly) of objects and their metadata | Output as CSV, ORC or Parquet; feeds Batch Operations |
| S3 Select | SQL against one object (CSV, JSON, Parquet) | One object only |
| Athena | SQL across many objects and buckets, with joins | The tool AWS suggests for querying access logs |

### Monitoring and analysis

| Feature | What you get | Remember |
|---------|--------------|----------|
| Server access logs | A text record of every request, delivered to another bucket | Free apart from storage; delivery is delayed; the target bucket's policy must allow `logging.s3.amazonaws.com` to write; the target bucket cannot use Object Lock or Requester Pays |
| Event notifications | Events such as object created, removed, restored, replicated or expired | Destinations: SNS, SQS, Lambda, EventBridge. Delivered at least once, so duplicates are possible. The queue, topic or function needs a resource policy that lets S3 send to it. `s3:ObjectCreated:Put` misses multipart uploads; `s3:ObjectCreated:*` covers every upload |
| Storage class analysis | Recommends when to move objects from Standard to Standard-IA | Needs 30 days or more of observation; it only recommends |
| Storage Lens | Organization-wide storage metrics dashboard | A default dashboard exists; advanced metrics cost extra |

### Uploading and downloading large objects

| Feature | How it works | Numbers |
|---------|--------------|---------|
| Multipart upload | Start an upload, send parts in parallel, then complete it with the list of part numbers and ETags | Recommended from 100 MB; up to 10,000 parts |
| Byte-range fetch | Add a `Range` header to GET to download part of an object | Typical ranges are 8 MB or 16 MB |

### Object properties

| Property | What it is | Remember |
|----------|------------|----------|
| Key and prefix | The full name; the part before the last `/` is the prefix | Maximum 1,024 bytes |
| ETag | A hash of the content, usually MD5 | Changes only when content changes, not metadata |
| Checksum | Integrity check on upload and download | CRC32, CRC32C, SHA-1 or SHA-256 |
| System metadata | `Content-Type`, `Cache-Control`, `Content-Encoding` and similar | Mostly set by S3; a few can be edited |
| User metadata | Your own key and value pairs | Keys start with `x-amz-meta-` |
| Object tags | Up to 10 per object | Usable in IAM policies, lifecycle filters and metrics |

### Object Lock (WORM)

| | Governance mode | Compliance mode | Legal hold |
|---|-----------------|-----------------|------------|
| Who can remove it | Users with `s3:BypassGovernanceRetention` | Nobody, including root | Anyone with `s3:PutObjectLegalHold` |
| Duration | A retention period you can shorten | A retention period that cannot be shortened | No end date; stays until removed |
| Use | Testing, internal protection | Regulatory retention | Litigation |

- Object Lock requires versioning and works on object versions.
- A plain delete still succeeds on a locked object: it only adds a delete marker. The lock protects the version.
- Legal holds and retention periods are independent of each other.
- Once enabled, Object Lock cannot be disabled and versioning cannot be suspended.

### Requests and endpoints

| Item | Meaning |
|------|---------|
| S3 URI | `s3://bucket/key`, the short address used by the CLI |
| Virtual-hosted style | `https://bucket.s3.region.amazonaws.com/key`; the current standard |
| Path style | `https://s3.region.amazonaws.com/bucket/key`; legacy |
| Dual-stack endpoint | `s3.dualstack.region.amazonaws.com`; serves IPv4 and IPv6 |
| CLI command sets | `s3` (high level), `s3api` (low level), `s3control` (access points, Batch Operations, Storage Lens), `s3outposts` |

## Memory hooks

- Bucket name test: could it be a subdomain? Then it is probably valid.
- **G**overnance = **g**uarded but a manager can open it. **C**ompliance = **c**emented shut.
- Restore tiers by speed: **E**xpedited, **S**tandard, **B**ulk, as in "**e**very **s**low **b**oat".
- Select is one file; Athena is all the files.

## Exam traps

- "Even the root user must not be able to delete it" means Object Lock in compliance mode.
- A 403 on a Requester Pays bucket usually means the request-payer header is missing.
- "Object is in Glacier and the download fails": restore it first.
- A consumer of event notifications must tolerate duplicates.
- Server access logs are best effort and delayed. For guaranteed, near-real-time API records, use CloudTrail data events.
- "Present different views of the same object to different applications without copying it" points to S3 Object Lambda.

## Outdated or wrong in the video

- **"Object Lock can only be turned on when the bucket is created."** It can be enabled on an existing bucket since late 2023. Very sure.
- **"Object Lock can only be set through the API, not the console."** The console supports it. Fairly sure.
- **"Batch Operations is console only."** Jobs can be created with `aws s3control create-job`. Very sure.
- **S3 Select** is presented as a current feature. AWS closed it to new customers in 2024 and points people to Athena. Fairly sure.
- **"Limit of 100 buckets."** The default is now 10,000. Fairly sure.
- **"Limit of 10 directory buckets", and directory buckets only for S3 Express One Zone.** The default is now 100 per account, and directory buckets in Local Zones store S3 One Zone-IA. Fairly sure.
- **Extra queue messages blamed on the `s3:ObjectCreated:*` wildcard.** One upload produces one event, whatever the filter. The extras were most likely the `s3:TestEvent` that S3 sends when a notification configuration is saved, plus his repeated test runs. Fairly sure.
- **"SQS de-dupes them."** A standard queue does not. Only FIFO queues remove duplicates. Very sure.
- **"A range of 0 to 999 gives the first 100 bytes."** That is the first 1,000 bytes; a slip of the tongue. Very sure.
- **Multipart uploads "never expire".** True, and that is the catch: abandoned parts are billed until removed, so add a lifecycle rule to abort incomplete multipart uploads. Very sure.
- **Why his own checksums were rejected.** S3 expects the checksum as the base64 encoding of the raw digest bytes. He passes the hex digest, and at [18:37:54](https://youtu.be/5JTBS09e7ew?t=67074) base64-encodes the hex text, which gives a different value. Fairly sure.

Related: [[FCC 09.1 - S3 basics, CLI and IaC tooling]] · [[FCC 09.3 - S3 storage classes]] · [[FCC 09.4 - S3 security and encryption]] · [[FCC 09.5 - S3 replication, versioning and lifecycle]] · [[FCC 10 - Amazon S3 Glacier]] · [[FCC 00 - Watch guide]]
