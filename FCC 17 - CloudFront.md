# FCC 17 - CloudFront

> Video chapter [38:36:48](https://youtu.be/5JTBS09e7ew?t=139008) to 39:11:18 · 35 min total · all lecture, no labs
> Fast path: read this note and skip the video. It is 23 short slides read aloud, with no console work. The tables below hold every fact he states, plus a few exam points he leaves out.

## Summary

- CloudFront is a CDN: a global cache in front of an **origin** (S3 bucket, EC2, load balancer, any HTTP server). A **distribution** is the configuration that ties origins to cache behaviours.
- Three policies control a behaviour. The **cache policy** sets the cache key and TTLs. The **origin request policy** sets what is forwarded to the origin. The **response headers policy** adds headers to the reply.
- Lock an S3 bucket to CloudFront with **Origin Access Control (OAC)**. Origin Access Identity (OAI) is the legacy way.
- Code at the edge: **CloudFront Functions** (tiny, fast, viewer events only) or **Lambda@Edge** (slower, all four events, network access).
- Stale content is fixed with an **invalidation** or with versioned file names. Private content uses **signed URLs** (one file) or **signed cookies** (many files).

## Labs and demos

There is no lab or console demo in this chapter. He only shows slides with JSON and code samples.

## Lecture index

| Time | Topic |
|------|-------|
| [38:36:51](https://youtu.be/5JTBS09e7ew?t=139011) | What a CDN is, core components |
| [38:38:45](https://youtu.be/5JTBS09e7ew?t=139125) | Lambda@Edge: four triggers, use cases, code samples |
| [38:41:21](https://youtu.be/5JTBS09e7ew?t=139281) | CloudFront Functions |
| [38:42:25](https://youtu.be/5JTBS09e7ew?t=139345) | CloudFront Functions compared with Lambda@Edge |
| [38:44:55](https://youtu.be/5JTBS09e7ew?t=139495) | Origins |
| [38:46:11](https://youtu.be/5JTBS09e7ew?t=139571) | Origin groups (failover) |
| [38:47:30](https://youtu.be/5JTBS09e7ew?t=139650) | Cache key and cache policy |
| [38:49:22](https://youtu.be/5JTBS09e7ew?t=139762) | Origin request policy |
| [38:51:49](https://youtu.be/5JTBS09e7ew?t=139909) | Response headers policy |
| [38:53:16](https://youtu.be/5JTBS09e7ew?t=139996) | Origin Access Identity (OAI) |
| [38:55:03](https://youtu.be/5JTBS09e7ew?t=140103) | Origin Access Control (OAC) |
| [38:56:39](https://youtu.be/5JTBS09e7ew?t=140199) | Origin Shield |
| [38:57:23](https://youtu.be/5JTBS09e7ew?t=140243) | Geographic restrictions |
| [38:58:02](https://youtu.be/5JTBS09e7ew?t=140282) | Caching on query strings |
| [38:58:59](https://youtu.be/5JTBS09e7ew?t=140339) | Caching on cookies |
| [39:00:29](https://youtu.be/5JTBS09e7ew?t=140429) | Content type (MIME) |
| [39:01:36](https://youtu.be/5JTBS09e7ew?t=140496) | Invalidating the cache |
| [39:03:11](https://youtu.be/5JTBS09e7ew?t=140591) | Cache busting with a digest in the file name |
| [39:05:32](https://youtu.be/5JTBS09e7ew?t=140732) | Default root object |
| [39:06:31](https://youtu.be/5JTBS09e7ew?t=140791) | Requiring HTTPS |
| [39:07:42](https://youtu.be/5JTBS09e7ew?t=140862) | Signed URLs |
| [39:09:10](https://youtu.be/5JTBS09e7ew?t=140950) | Signed cookies |
| [39:10:18](https://youtu.be/5JTBS09e7ew?t=141018) | Custom error pages |

## Notes

### Core components

| Part | What it is |
|------|------------|
| Origin | Where the original files live: S3 bucket, EC2 instance, load balancer, any HTTP server |
| Edge location | Cache close to the users |
| Regional edge cache | Larger cache between the origin and the edge locations |
| Distribution | The configuration: origins, cache behaviours, policies |
| Origin group | A primary and a secondary origin, for failover |
| Origin Shield | One extra cache layer in front of the origin; fewer origin requests, higher hit ratio |

- Serves static and dynamic content, video streaming and WebSockets. AWS WAF can sit in front.
- Origin failover is triggered by HTTP status codes you choose (for example 500, 502, 503, 504) or by a connection failure. It happens only for GET, HEAD and OPTIONS requests.

### Edge code: CloudFront Functions and Lambda@Edge

| | CloudFront Functions | Lambda@Edge |
|---|----------------------|-------------|
| Language | JavaScript | Node.js, Python |
| Events | Viewer request, viewer response | Viewer request, origin request, origin response, viewer response |
| Runs at | Edge locations | Regional edge caches |
| Scale | 10 million requests a second or more | 10,000 requests a second per Region |
| Run time | Under 1 ms | 5 s (viewer events), 30 s (origin events) |
| Memory | 2 MB | 128 MB or more |
| Code size | 10 KB | 1 MB (viewer), 50 MB (origin) |
| Network, file system, request body | No | Yes |
| Build and test inside CloudFront | Yes | No |
| Price | Per request, has a free tier | Per request and duration, no free tier |

| Event | Typical use |
|-------|-------------|
| Viewer request | Redirect HTTP to HTTPS, check a cookie or token, normalise the cache key |
| Origin request | Rewrite the URL, add an auth header for the origin, pick mobile or desktop content from the User-Agent |
| Origin response | Change caching headers, turn an origin error into a custom page |
| Viewer response | Add security headers, set cookies |

Rule of thumb: use CloudFront Functions when they can do the job. Use Lambda@Edge when you need origin events, network calls, the request body or more time.

### The three policies

| Policy | Controls | Managed policies worth knowing |
|--------|----------|--------------------------------|
| Cache policy | The cache key (which headers, cookies, query strings count) and the TTLs: minimum, maximum, default | CachingOptimized, CachingDisabled, CachingOptimizedForUncompressedObjects |
| Origin request policy | Extra headers, cookies and query strings sent to the origin on a cache miss, without adding them to the cache key | AllViewer, AllViewerExceptHostHeader, CORS-S3Origin, CORS-CustomOrigin |
| Response headers policy | Headers added to the response for the viewer (CORS, security headers) | SimpleCORS, CORS-With-Preflight, SecurityHeadersPolicy |

- **Cache key**: the identity of a cached object. Same key as an earlier request means a cache hit. More values in the key means fewer hits.
- **Cache hit ratio**: the share of requests served from the edge. Higher is faster and cheaper.
- By default the origin receives only the URL path, the body and a few headers. Query strings and cookies are neither forwarded nor cached on unless a policy says so.
- Everything in the cache key is also sent to the origin.
- All TTLs set to 0 turns caching off.
- A managed policy is referenced by its ID, not its name.
- Example of a query string in the cache key: `?lang=fr` to cache one copy per language.

### Restricting S3 to CloudFront

| | OAI (legacy) | OAC (use this) |
|---|--------------|----------------|
| Bucket policy principal | A special CloudFront user ARN containing the OAI ID | Service `cloudfront.amazonaws.com`, with a condition on the distribution ARN |
| Signing | No SigV4 | SigV4 with short-term credentials |
| SSE-KMS objects | No | Yes |
| HTTP methods | Limited | GET, HEAD, OPTIONS, PUT, POST, PATCH, DELETE |
| Regions | Only Regions launched before December 2022 | All |

### Keeping content fresh

| Method | How | Note |
|--------|-----|------|
| Wait for the TTL | Nothing to do | Object is fetched again after expiry |
| Invalidation | `create-invalidation` with paths | The path must start with `/`; `/*` clears everything |
| Versioned file names | Put a hash of the content in the name, e.g. `style.3f9a1c2b.css` | No invalidation needed; old and new versions can live side by side. Build tools such as webpack do it for you. |

```bash
aws cloudfront create-invalidation --distribution-id E123EXAMPLE --paths "/index.html" "/css/*"
aws cloudfront create-invalidation --distribution-id E123EXAMPLE --invalidation-batch file://batch.json
```

```json
{ "Paths": { "Quantity": 2, "Items": ["/index.html", "/css/*"] },
  "CallerReference": "deploy-2026-10-08-01" }
```

`CallerReference` must be a new unique value each time.

### Access and delivery settings

| Setting | Fact |
|---------|------|
| Geographic restriction | Allow list or block list of country codes; blocked viewers get 403 |
| Default root object | The file returned for the bare domain, usually `index.html`. Not applied to subdirectories. |
| Viewer protocol policy | Set per cache behaviour to require HTTPS from viewers. A custom domain needs a certificate from ACM. |
| Origin protocol policy | For custom origins; the origin needs its own certificate |
| S3 website endpoint as origin | HTTP only, so HTTPS to the origin cannot be required. Prefer the bucket (REST) endpoint. |
| Content type | CloudFront does not work out the MIME type. Set `Content-Type` when you upload to the origin. |
| Custom error pages | Map an origin error code to a page, and optionally change the status code. For a single-page app, return `/index.html` with 200. |

### Private content

| | Signed URL | Signed cookies |
|---|------------|----------------|
| Use for | One file, e.g. a paid PDF | Many files, e.g. a video library, without changing URLs |
| Policy | Canned (URL and expiry) or custom (more restrictions) | Same policy syntax |
| How | Your application signs the policy with a private key | Your application sets `CloudFront-Policy`, `CloudFront-Signature`, `CloudFront-Key-Pair-Id` |

### Not in the video

| Topic | Fact |
|-------|------|
| Certificate Region | The ACM certificate for a distribution must be in us-east-1 |
| Lambda@Edge Region | The function must be created in us-east-1 |
| AWS WAF | A web ACL for CloudFront has global scope and is created in us-east-1 |
| OAC and endpoints | OAC works only with the S3 REST endpoint, not the static website endpoint |
| Troubleshooting a miss | Response header `X-Cache: Hit from cloudfront` or `Miss from cloudfront` |
| Single-page app with OAC | A missing object returns 403 from S3, not 404, so map 403 as well |

## Memory hooks

- Cache policy = what makes two requests "the same". Origin request policy = what the origin gets told. Response headers policy = what the viewer gets told.
- OA**C** is the **c**urrent one. OA**I** is **i**n the past.
- Functions are a sticky note on the door (edge, tiny). Lambda@Edge is an office down the hall (regional, capable).
- URL for one, cookie for many.

## Exam traps

- "Users still see the old file after a deployment": create an invalidation, or use versioned file names.
- "The bucket must be reachable only through CloudFront": OAC plus a bucket policy that allows the CloudFront service with the distribution ARN.
- "Cache hit ratio is low": too many headers, cookies or query strings in the cache key. Move what the origin needs but does not vary on into the origin request policy.
- "Serve from a second origin when the first returns 5xx": origin group. It does not fail over POST or PUT.
- "Block viewers from certain countries": geographic restriction, viewers get 403.
- "The certificate does not appear in the distribution settings": it was requested outside us-east-1.

## Outdated or wrong in the video

- **"ECMAScript 5.1 is the only language for CloudFront Functions."** JavaScript runtime 2.0 (`cloudfront-js-2.0`) adds newer syntax such as `async`/`await`. Very sure.
- **"The default root object defaults to index.html."** There is no default. If you set none, a request for `/` is passed to the origin as it is. Very sure.
- **"Signed URLs are signed with a CloudFront key pair made in the console."** That is the legacy method and needs the root user. Now you create your own key pair, upload the public key and add it to a trusted key group. Very sure.
- **"Invalidating everything is expensive because every file is copied to all edges again."** CloudFront does not push files; each edge fetches on the next request. The cost is extra origin load. For billing, `/*` counts as one path and the first 1,000 paths a month are free on pay-as-you-go pricing. Very sure.
- **"A distribution is a collection of edge locations."** It is a configuration that is deployed to all edge locations. Very sure.

Related: [[FCC 09.4 - S3 security and encryption]] · [[FCC 16 - Amazon Route 53]] · [[FCC 18 - AWS Global Accelerator]] · [[FCC 27 - AWS Certificate Manager]] · [[FCC 00 - Watch guide]]
