# FCC 09.4 - S3 security and encryption

> Video section [20:35:39](https://youtu.be/5JTBS09e7ew?t=74139) to 23:39:40 (part 4 of 5 of the S3 chapter) · 3h04m total · about 45 min lecture, 2h20m labs
> Fast path: read this note. Optional: six minutes of the cross-account bucket policy lab from [21:18:22](https://youtu.be/5JTBS09e7ew?t=76702) and the SSE-C test at [23:20:21](https://youtu.be/5JTBS09e7ew?t=84021). Skip the 47-minute CORS lab.

## Summary

- Block Public Access is on by default and overrides any ACL or policy that would make a bucket public.
- Bucket policies are the main way to grant access to a bucket, including to other accounts. ACLs are legacy.
- Every new object is encrypted at rest with SSE-S3 unless you choose SSE-KMS, DSSE-KMS or SSE-C.
- SSE-KMS adds key policies, audit and cost. Uploading needs `kms:GenerateDataKey`; downloading needs `kms:Decrypt`.
- Gateway endpoints (free) and interface endpoints (PrivateLink, paid) keep S3 traffic off the internet.

## Labs and demos

| Time | Len | What it shows | Takeaway | Verdict |
|------|-----|---------------|----------|---------|
| [20:43:36](https://youtu.be/5JTBS09e7ew?t=74616) | 29m | Grants another AWS account access with a bucket ACL, using `s3api put-bucket-acl` | Three hurdles: Object Ownership set to "bucket owner enforced" (which disables ACLs), the grantee must be a canonical user ID, not an account ID, and `--access-control-policy` takes JSON, not the XML in the docs (5 minutes lost from [21:02:05](https://youtu.be/5JTBS09e7ew?t=75725)). Turning off Block Public Access for ACLs was probably not needed, as he says later. The other account can then list and upload. | Skip |
| [21:14:38](https://youtu.be/5JTBS09e7ew?t=76478) | 12m | Cross-account access with a bucket policy: `s3api put-bucket-policy` naming an IAM user of the other account as principal, then listing and uploading from that account | Works with Block Public Access left on, because the grant is not public. Only error: a stray comma ("policy contains invalid JSON"). The first four minutes are setup. The IAM side in the other account is never shown. | Skim: [21:18:22](https://youtu.be/5JTBS09e7ew?t=76702) to 21:24:29 |
| [21:31:21](https://youtu.be/5JTBS09e7ew?t=77481) | 10m | IAM Access Analyzer for S3: creates an analyzer, then makes a bucket public to see it flagged | The new finding only appears after he recreates the analyzer; scans can take up to 30 minutes. One button re-blocks public access. | Skip |
| [21:47:01](https://youtu.be/5JTBS09e7ew?t=78421) | 47m | CORS: two static website buckets, then an API Gateway mock endpoint called from the page with a JavaScript POST | No CORS error for 37 minutes: a Google Font and a `<script>` from a second bucket both load. The error only appears on the POST to API Gateway. `put-bucket-cors` on the website bucket does not fix it; "Enable CORS" in API Gateway plus a redeploy does. The rest is typos and JavaScript fixes. | Skip |
| [22:46:54](https://youtu.be/5JTBS09e7ew?t=82014) | 37m | Encryption: default SSE-S3, then SSE-KMS with `s3api put-object`, then SSE-C. DSSE-KMS is not attempted. | About 24 minutes are trouble: 11 on AWS managed versus customer managed keys, 10 on SSE-C through `s3api` (key and MD5 rejected, never solved), 3 on `openssl` syntax. What works: the SSE-KMS upload, and SSE-C through `aws s3 cp`. | Skim |
| [23:28:29](https://youtu.be/5JTBS09e7ew?t=84509) | 11m | Client-side encryption with the Ruby SDK | Reading with the key returns plaintext; reading without it returns ciphertext. | Skip |

### Useful moments inside the CORS lab

| Time | What you learn |
|------|----------------|
| [21:58:26](https://youtu.be/5JTBS09e7ew?t=79106) | Website endpoint: `http://<bucket>.s3-website.<region>.amazonaws.com`. Some Regions use `s3-website-<region>`; that form fails for ca-central-1. |
| [22:24:05](https://youtu.be/5JTBS09e7ew?t=80645) | The first CORS error in the browser console, on the POST; he explains the preflight request |
| [22:27:08](https://youtu.be/5JTBS09e7ew?t=80828) | API Gateway "Enable CORS" adds the headers and an OPTIONS method; after "Deploy API" the call works ([22:29:09](https://youtu.be/5JTBS09e7ew?t=80949)) |

### Useful moments inside the encryption lab

| Time | What you learn |
|------|----------------|
| [22:48:29](https://youtu.be/5JTBS09e7ew?t=82109) | A plain upload already shows SSE-S3. Changing an existing object's encryption means writing it again as a copy. |
| [23:00:44](https://youtu.be/5JTBS09e7ew?t=82844) | The AWS managed key `aws/s3` only appears in a Region after the first SSE-KMS upload there ([23:01:47](https://youtu.be/5JTBS09e7ew?t=82907)) |
| [23:03:49](https://youtu.be/5JTBS09e7ew?t=83029) | `s3api` wants `--ssekms-key-id`; `aws s3 cp` wants `--sse-kms-key-id`. The object becomes SSE-KMS while the bucket default stays SSE-S3. It downloads with no extra flags ([23:05:51](https://youtu.be/5JTBS09e7ew?t=83151)), because the caller has `kms:Decrypt`. |
| [23:20:21](https://youtu.be/5JTBS09e7ew?t=84021) | SSE-C that works, through `aws s3 cp`. Download without the key: 400 Bad Request. With the key: fine ([23:21:22](https://youtu.be/5JTBS09e7ew?t=84082)). |

## Lecture index

| Time | Topic |
|------|-------|
| [20:35:39](https://youtu.be/5JTBS09e7ew?t=74139) | Security overview |
| [20:40:12](https://youtu.be/5JTBS09e7ew?t=74412) | Block Public Access |
| [20:42:06](https://youtu.be/5JTBS09e7ew?t=74526) | ACLs |
| [21:12:45](https://youtu.be/5JTBS09e7ew?t=76365) | Bucket policies; bucket policy vs IAM policy at [21:26:49](https://youtu.be/5JTBS09e7ew?t=77209) |
| [21:28:50](https://youtu.be/5JTBS09e7ew?t=77330) | Access Grants, IAM Access Analyzer for S3 |
| [21:40:54](https://youtu.be/5JTBS09e7ew?t=78054) | Keeping traffic private: interface and gateway endpoints |
| [21:43:11](https://youtu.be/5JTBS09e7ew?t=78191) | CORS |
| [22:34:18](https://youtu.be/5JTBS09e7ew?t=81258) | Encryption overview, encryption in transit |
| [22:37:10](https://youtu.be/5JTBS09e7ew?t=81430) | Server-side encryption: SSE-S3, SSE-KMS, SSE-C, DSSE-KMS |
| [23:24:05](https://youtu.be/5JTBS09e7ew?t=84245) | S3 Bucket Keys |
| [23:26:46](https://youtu.be/5JTBS09e7ew?t=84406) | Client-side encryption |

## Notes

### Ways to control access

| Mechanism | What it is | Use it for |
|-----------|------------|------------|
| Block Public Access | Four switches that block public ACLs and public policies; set per account, bucket or access point | Leave on unless you are deliberately hosting public content |
| Bucket policy | JSON resource policy on one bucket; names principals; up to 20 KB | Cross-account access, conditions such as IP range, tags or encryption |
| IAM policy | Attached to a user, group or role | Access to many buckets and other services for your own identities |
| ACL | Legacy read and write grants to other accounts; no conditions, no deny | Avoid; disabled by default through Object Ownership |
| Access point | A named endpoint with its own policy | Splitting a large bucket policy per application |
| Access Grants | Maps directory identities (IAM Identity Center, Active Directory, Okta) to prefixes | Giving corporate users data access with temporary credentials |
| IAM Access Analyzer for S3 | Lists buckets shared publicly or with other accounts | Finding exposure you did not intend |

### Cross-account bucket policy from the lab

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": {"AWS": "arn:aws:iam::111122223333:user/other-user"},
    "Action": ["s3:GetObject", "s3:PutObject", "s3:ListBucket"],
    "Resource": ["arn:aws:s3:::my-bucket", "arn:aws:s3:::my-bucket/*"]
  }]
}
```

- `s3:ListBucket` applies to the bucket ARN; object actions apply to `my-bucket/*`. Hence two resources.
- The principal can be a user, a role or a whole account (`...:root`).
- Both sides must allow it: this policy, plus S3 permissions on the identity in its own account.

### Server-side encryption

| | SSE-S3 | SSE-KMS | DSSE-KMS | SSE-C |
|---|--------|---------|----------|-------|
| Who holds the key | S3 | KMS key (AWS managed or yours) | KMS key | You; sent with every request |
| Default | Yes, always on | No | No | No |
| Cost | Free | KMS key and request charges | KMS charges | Free |
| Audit and key policy | No | Yes, through KMS and CloudTrail | Yes | You manage everything |
| Remember | AES-256 | Key must be in the bucket's Region | Two layers of encryption | Lose the key, lose the data |

### Other encryption facts

- **In transit**: TLS, version 1.2 at least, 1.3 preferred.
- **Client-side encryption**: you encrypt before uploading, so AWS never sees the key or the plaintext.
- **S3 Bucket Key**: a short-lived bucket-level key that cuts KMS requests, and therefore cost, by up to 99% for SSE-KMS.
- **Metadata**: server-side encryption covers the object data, not its metadata.

```bash
# SSE-KMS
aws s3api put-object --bucket my-bucket --key hello.txt --body hello.txt \
  --server-side-encryption aws:kms --ssekms-key-id <key-id>

# SSE-C: 32-byte key file (byte count goes last); same two flags to download
openssl rand -out ssec.key 32
aws s3 cp hello.txt s3://my-bucket/ --sse-c AES256 --sse-c-key fileb://ssec.key
```

### KMS keys for SSE-KMS

| | AWS managed key (`aws/s3`) | Customer managed key |
|---|----------------------------|----------------------|
| Created by | S3, on first use in a Region; never by you | You (`kms create-key` or console) |
| Monthly fee | None | About $1 per key |
| Key policy | Fixed by AWS; no cross-account use | Yours to edit |
| Delete | Not possible | Scheduled, after a 7 to 30 day wait |

### Keeping traffic private

| | Gateway endpoint | Interface endpoint (PrivateLink) |
|---|------------------|----------------------------------|
| Works with | S3 and DynamoDB only | Most AWS services |
| How | An entry in the route table | A network interface with a private IP in your subnet |
| Cost | Free | Hourly and per-GB charges |
| Reachable from on premises | No | Yes, over VPN or Direct Connect |

### CORS

- A browser rule: a page from one origin may call another origin only if that other server returns headers such as `Access-Control-Allow-Origin`.
- For S3, the CORS configuration is JSON on the bucket: allowed origins, methods and headers. Set it with `aws s3api put-bucket-cors`.
- It goes on the bucket (or API) being called, not on the bucket that serves the page. In the lab the page calls API Gateway, so the bucket setting changes nothing.
- CORS applies to JavaScript calls (`fetch`, XHR). A plain `<script src>` or `<img>` tag is not checked.
- Preflight: before a call such as a POST with JSON, the browser sends an OPTIONS request and expects the CORS headers back.
- Avoid `*` as the allowed origin in production.

## Memory hooks

- Block Public Access is the master switch: a public policy does nothing while it is on.
- SSE-**S3**: S3's keys. SSE-**KMS**: KMS's keys with a logbook. SSE-**C**: the **c**ustomer carries the key.
- **G**ateway endpoint = **g**ratis, and only for S3 and DynamoDB.

## Exam traps

- "Access denied on an object encrypted with SSE-KMS, although the bucket policy allows it": the caller lacks `kms:Decrypt` on the key.
- "Enforce encryption in transit": a bucket policy that denies requests where `aws:SecureTransport` is false.
- "Private EC2 instances must reach S3 without a NAT gateway and at no cost": gateway endpoint.
- "Need an audit trail of who used the encryption key": SSE-KMS.
- "A site in bucket A fetches files from bucket B, or calls an API, and gets a CORS error": configure CORS on bucket B or the API (the server being called), not on A.
- "Another account must read SSE-KMS objects": customer managed key with that account in the key policy; `aws/s3` cannot be shared.

## Outdated or wrong in the video

- **"Gateway endpoints do not have fine-grained permissions."** They support endpoint policies, the same as interface endpoints. Very sure.
- **DSSE-KMS described as client-side plus server-side encryption.** Both layers are applied on the server by S3. The key exchange he describes belongs to client-side encryption with KMS. Very sure.
- **TLS and SSL history told backwards.** SSL 2 and 3 came first, then TLS 1.0 to 1.3. Very sure.
- **Cross-account access seems to need only the bucket policy.** He is surprised that no role or IAM setup was needed ([21:24:29](https://youtu.be/5JTBS09e7ew?t=77069)). The other account's user must already have had S3 permissions there. Both sides have to allow the access. Very sure.
- **CORS errors come mostly from POST and PUT, not GET ([22:24:05](https://youtu.be/5JTBS09e7ew?t=80645)).** Any cross-origin `fetch` or XHR call needs the header, GET included; the POST only adds a preflight. His script loaded because `<script src>` is not checked, and the font because Google Fonts sends the header itself. Very sure.
- **SSE-C works on a brand-new bucket.** Since April 2026 new buckets block SSE-C by default; unblock it in the bucket's encryption configuration first. Fairly sure.
- **Bucket Keys for SSE-S3.** He wonders whether they help there. Bucket Keys apply to SSE-KMS. Fairly sure.

Related: [[FCC 09.2 - S3 buckets and objects]] · [[FCC 09.5 - S3 replication, versioning and lifecycle]] · [[FCC 21 - Identity and Access Management IAM]] · [[FCC 15.2 - VPC gateways, IPv6, endpoints and flow logs]] · [[FCC 00 - Watch guide]]
