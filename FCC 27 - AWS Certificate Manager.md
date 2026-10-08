# FCC 27 - AWS Certificate Manager

> Video chapter [52:21:05](https://youtu.be/5JTBS09e7ew?t=188465) to 52:39:39 · 19 min total · 4 min lecture, 15 min lab
> Fast path: read this note. If you want to see the console once, watch 5 minutes of the lab from [52:32:05](https://youtu.be/5JTBS09e7ew?t=189125). The first 7 minutes of the lab are set-up, wrong credentials and a wrong template.

## Summary

- ACM issues, stores and deploys TLS certificates for AWS services. Public certificates are free.
- You attach a certificate to a load balancer, a CloudFront distribution or an API Gateway. You do not install it on a server.
- Put the bare domain and a wildcard on one certificate: `example.com` and `*.example.com`.
- Prove you own the domain by DNS or by email. DNS is the easier one.
- A certificate belongs to one Region. A resource in another Region needs its own certificate there.

## Labs and demos

| Time | Len | What it shows | Takeaway | Verdict |
|------|-----|---------------|----------|---------|
| [52:25:03](https://youtu.be/5JTBS09e7ew?t=188703) | 15m | HTTPS on an Application Load Balancer: deploys the ALB with CloudFormation, requests a public certificate, adds a 443 listener, points a Route 53 record at the ALB | The first 7 minutes are set-up and two mistakes: the stack goes to the wrong account, then he finds he deployed an Auto Scaling group template and not the ALB one. The ACM part, [52:32:05](https://youtu.be/5JTBS09e7ew?t=189125) to 52:37:10, works first time and the certificate is issued almost at once. Clean-up: a certificate in use cannot be deleted, and the stack delete fails once on the target group before a retry works. | Skim |

## Lecture index

| Time | Topic |
|------|-------|
| [52:21:05](https://youtu.be/5JTBS09e7ew?t=188465) | What ACM is, public and private certificates, wildcards, services it attaches to |
| [52:22:42](https://youtu.be/5JTBS09e7ew?t=188562) | TLS termination at the load balancer |
| [52:24:00](https://youtu.be/5JTBS09e7ew?t=188640) | End-to-end encryption |

## Notes

### Certificate types

| Type | Where it comes from | Cost | Renewal |
|------|---------------------|------|---------|
| Public | Issued by ACM | Free | Automatic |
| Private | Issued by a private certificate authority you run in AWS Private CA | The CA costs about $400 a month | Automatic |
| Imported | Bought elsewhere and uploaded to ACM | Free | Manual: you import a new one before it expires |

In the request screen, choose "public". Picking the private option by mistake is the costly error he warns about.

### Validation

| Method | How | Note |
|--------|-----|------|
| DNS | Add the CNAME record ACM gives you. With a Route 53 zone it is one button. | Renewal stays automatic while the record exists |
| Email | Approve a link sent to the domain's contact addresses | Someone has to approve again at renewal |

### Where TLS ends

| | Terminate at the load balancer | End to end |
|---|-------------------------------|------------|
| Encrypted | Client to ALB | Client to ALB, and ALB to instance |
| Certificates | One ACM certificate on the listener | ACM on the listener, plus a certificate on every instance |
| Effort | Low. Add instances without touching certificates. | High. Install and rotate on each server. |
| Use when | Almost always | A compliance rule demands encryption inside the network |

### The lab in five steps

1. ACM, Request, public certificate. Names: `example.org` and `*.example.org`.
2. DNS validation, then "Create records in Route 53". Status goes from Pending validation to Issued.
3. EC2, Load Balancers, add a listener: HTTPS on 443, forward to the target group, certificate "From ACM".
4. Route 53: an A record with Alias on, target "Application Load Balancer", the Region, the ALB.
5. Open `https://example.org`. The browser shows a certificate issued by Amazon.

### Not in the video

| Topic | Fact |
|-------|------|
| CloudFront | The certificate must be requested in us-east-1, wherever the origin is |
| ALB, NLB, regional API Gateway | The certificate must be in the same Region as the resource |
| Imported certificates | ACM does not renew them. Watch the `DaysToExpiry` metric, the Config rule `acm-certificate-expiration-check`, or the EventBridge expiry event. |
| HTTP to HTTPS | Add a redirect action on the port 80 listener |
| Several domains on one ALB | A listener can hold many certificates and picks one by host name (SNI) |
| TLS versions and ciphers | Set by the listener's security policy |
| Wildcard scope | `*.example.com` covers one level of subdomain and does not cover `example.com` itself |

## Memory hooks

- ACM guards the front door: load balancer, CloudFront, API Gateway.
- CloudFront is global, so its certificates live in us-east-1.
- Always request the pair: bare domain plus star.

## Exam traps

- "The certificate does not appear in the CloudFront drop-down": it was requested outside us-east-1.
- "Certificate stuck in Pending validation": the CNAME validation record is missing from DNS.
- "The site went down when a certificate expired": it was an imported certificate, which ACM does not renew.
- "Encrypt traffic between the ALB and the instances": HTTPS target group and a certificate on each instance.
- "Point the bare domain at an ALB": an alias A record, not a CNAME.
- "Cannot delete the certificate": it is still attached to a resource.

## Outdated or wrong in the video

- **"Private certificates are ones you import, $400 a month."** Three things are mixed up. Private certificates are issued by AWS Private CA, and the $400 a month is for the CA. Importing your own certificate is free. Very sure.
- **"The domain has to be in Route 53."** DNS validation works with any DNS provider; you add the CNAME yourself. Very sure.
- **"It doesn't matter if it's A or CNAME."** For the bare domain it does: a CNAME is not allowed there, so use an alias A record. Very sure.
- **"I don't think you can do end to end with ACM."** Since 2025 ACM can issue exportable public certificates, for a fee, that you can install on EC2. The free ones still cannot be exported. Fairly sure.
- **"The validation step is a pain in CloudFormation."** `AWS::CertificateManager::Certificate` creates the Route 53 validation record itself when `DomainValidationOptions` includes the `HostedZoneId`. Very sure.

Related: [[FCC 19 - ELB]] · [[FCC 16 - Amazon Route 53]] · [[FCC 17 - CloudFront]] · [[FCC 29 - AWS CloudFormation]] · [[FCC 00 - Watch guide]]
