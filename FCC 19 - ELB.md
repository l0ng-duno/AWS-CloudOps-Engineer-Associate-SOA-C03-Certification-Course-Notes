# FCC 19 - ELB

> Video chapter [39:27:09](https://youtu.be/5JTBS09e7ew?t=142029) to 40:31:14 · 1h04m total · about 25 min lecture, 39 min lab
> Fast path: read this note. If you want to see a load balancer in the console, watch 4 minutes of the lab from [40:06:14](https://youtu.be/5JTBS09e7ew?t=144374) to 40:10:30 (HTTPS listener rejected, healthy target, security group chain). The lectures are slides read aloud and are fully covered below.

## Summary

- Elastic Load Balancing has four types: **ALB** (layer 7, HTTP), **NLB** (layer 4, TCP and UDP, static IP), **GWLB** (layer 3, virtual appliances) and **CLB** (legacy).
- Traffic flow: **listener** (port and protocol) → **rule** (ALB only) → **target group** → **target**.
- Health checks are set on the target group. The load balancer only stops sending traffic to an unhealthy target. It never terminates it; an Auto Scaling group does that.
- An ALB rule is conditions plus an action plus a priority. Actions: forward, redirect, fixed response, authenticate.
- Know the defaults: cross-zone behaviour per type, deregistration delay 300 seconds, client IP in `X-Forwarded-For`.

## Labs and demos

| Time | Len | What it shows | Takeaway | Verdict |
|------|-----|---------------|----------|---------|
| [39:32:38](https://youtu.be/5JTBS09e7ew?t=142358) | 39m | Writes a CloudFormation template for an internet-facing ALB, a listener, a target group with a health check, one Apache EC2 instance and two security groups, then deploys it | About 32 minutes are spent reading the CloudFormation reference and pasting properties. The first deploy fails because an HTTPS listener must have a certificate. He replaces it with an HTTP listener on port 80 and it works. The target shows healthy. The instance's own IP does not load, because its security group only admits the ALB's security group; the ALB's DNS name does. Routing rules are never shown. | Skim |

### Useful moments inside the ALB lab

| Time | What you learn |
|------|----------------|
| [39:36:31](https://youtu.be/5JTBS09e7ew?t=142591) | An ALB needs subnets in at least two Availability Zones, one subnet per zone |
| [39:42:37](https://youtu.be/5JTBS09e7ew?t=142957) | A port 80 listener whose default action redirects to HTTPS |
| [39:43:39](https://youtu.be/5JTBS09e7ew?t=143019) | A forward action that points at a target group |
| [39:48:41](https://youtu.be/5JTBS09e7ew?t=143321) | Health check numbers: 10 s interval, 5 s timeout, 2 healthy, 2 unhealthy. Targets listen on port 80 because TLS ends at the load balancer. |
| [39:50:42](https://youtu.be/5JTBS09e7ew?t=143442) | `TargetType: instance` and registering the instance under `Targets` |
| [39:57:03](https://youtu.be/5JTBS09e7ew?t=143823) | Web server security group that allows port 80 only from the ALB's security group |
| [40:06:14](https://youtu.be/5JTBS09e7ew?t=144374) | Stack fails with "A certificate must be specified for HTTPS listeners"; he switches to HTTP |
| [40:09:17](https://youtu.be/5JTBS09e7ew?t=144557) | Target is healthy; the direct IP is blocked; the ALB DNS name works |

## Lecture index

| Time | Topic |
|------|-------|
| [39:27:09](https://youtu.be/5JTBS09e7ew?t=142029) | What a load balancer is, the four types |
| [39:29:12](https://youtu.be/5JTBS09e7ew?t=142152) | Listeners, rules, target groups |
| [39:30:42](https://youtu.be/5JTBS09e7ew?t=142242) | Application Load Balancer |
| [40:11:37](https://youtu.be/5JTBS09e7ew?t=144697) | Network Load Balancer |
| [40:12:36](https://youtu.be/5JTBS09e7ew?t=144756) | Classic Load Balancer |
| [40:13:21](https://youtu.be/5JTBS09e7ew?t=144801) | Sticky sessions |
| [40:15:34](https://youtu.be/5JTBS09e7ew?t=144934) | Cross-zone load balancing |
| [40:16:48](https://youtu.be/5JTBS09e7ew?t=145008) | `X-Forwarded-For` header |
| [40:17:26](https://youtu.be/5JTBS09e7ew?t=145046) | Health checks |
| [40:18:24](https://youtu.be/5JTBS09e7ew?t=145104) | Request routing |
| [40:19:15](https://youtu.be/5JTBS09e7ew?t=145155) | ALB rules: parts and limits |
| [40:20:05](https://youtu.be/5JTBS09e7ew?t=145205) | Rule action types |
| [40:21:12](https://youtu.be/5JTBS09e7ew?t=145272) | Fixed response action |
| [40:22:12](https://youtu.be/5JTBS09e7ew?t=145332) | Redirect and forward actions |
| [40:23:28](https://youtu.be/5JTBS09e7ew?t=145408) | Rule condition types, then one slide each until 40:28:07 |
| [40:28:07](https://youtu.be/5JTBS09e7ew?t=145687) | Target types |
| [40:30:02](https://youtu.be/5JTBS09e7ew?t=145802) | Deregistration delay (connection draining) |

## Notes

### The four load balancers

| | ALB | NLB | GWLB | CLB |
|---|-----|-----|------|-----|
| Layer | 7 | 4 | 3 | 4 and 7 |
| Traffic | HTTP, HTTPS, WebSocket, HTTP/2, gRPC | TCP, UDP | All IP traffic, sent through virtual appliances | HTTP, HTTPS, TCP |
| Target groups | Yes | Yes | Yes | No, instances register directly |
| Address | DNS name only | Static IP per zone | n/a | DNS name only |
| Known for | Routing rules, WAF, built-in authentication, Lambda targets | Millions of requests a second, very low latency, keeps the client's source IP | Firewalls and traffic inspection | Legacy, avoid |

- A load balancer lives in one VPC.
- An ALB can have AWS WAF, an ACM certificate, CloudFront or Global Accelerator in front of it.
- Static IP for an HTTP application: put an NLB in front of the ALB, with the ALB as its target.

### Traffic flow

```text
client → listener (port 80 or 443) → rule (ALB only) → target group → target
```

### Health checks

| Fact | Detail |
|------|--------|
| Where configured | On the target group (ALB, NLB). On the load balancer itself for CLB. |
| What happens | The load balancer calls a protocol, port and path on each target and expects a success code |
| Unhealthy target | Gets no traffic. It is **not** terminated. |
| Who replaces it | An Auto Scaling group whose health check type is ELB |

| Setting | His lab | ALB default |
|---------|---------|-------------|
| Interval | 10 s | 30 s |
| Timeout | 5 s | 5 s |
| Healthy threshold | 2 | 5 |
| Unhealthy threshold | 2 | 2 |
| Path and success code | `/`, 200 | `/`, 200 |

Time until a new target takes traffic is about interval × healthy threshold: 20 s with his values, 150 s with the defaults.

### ALB rules

- A rule = **conditions** + **action** + **priority**. Lowest priority number is evaluated first. The listener's default rule is last.
- Limits he gives: 100 rules per ALB, 5 condition values per rule, 5 weighted target groups per rule.
- Wildcards in conditions: `*` matches zero or more characters, `?` matches exactly one.

| Condition type | Routes on | Example |
|----------------|-----------|---------|
| `host-header` | Host name | `*.example.com` |
| `path-pattern` | URL path | `/img/*` |
| `http-header` | Any request header | `User-Agent` contains `*Chrome*` |
| `http-request-method` | Method | `GET`, `POST` |
| `query-string` | Key and value | `version=v1` |
| `source-ip` | Client address in CIDR form, IPv4 or IPv6 | `192.0.2.0/24`; no wildcards |

| Action type | What it does | Remember |
|-------------|--------------|----------|
| `forward` | Sends to one or more target groups | Several target groups need weights, 0 to 999. Weighted forwarding can break stickiness. |
| `redirect` | Returns 301 (permanent) or 302 (temporary) | Reuse parts of the request with `#{protocol}`, `#{host}`, `#{port}`, `#{path}`, `#{query}`. Classic use: HTTP to HTTPS. |
| `fixed-response` | Returns a status code (2XX, 4XX or 5XX), content type and body without contacting a target | Counted in the `HTTP_Fixed_Response_Count` metric |
| `authenticate-cognito` | Signs the user in through a Cognito user pool | Built in |
| `authenticate-oidc` | Signs the user in through any OIDC identity provider | Built in |

For gRPC and HTTP/2 target groups, only `forward` is allowed.

### Target types

| Type | You register | Remember |
|------|--------------|----------|
| `instance` | EC2 instance ID | Keeps working when the instance's IP changes |
| `ip` | Private IP address | Required for Fargate tasks. Ranges: 10.0.0.0/8, 100.64.0.0/10, 172.16.0.0/12, 192.168.0.0/16. |
| `lambda` | One Lambda function | ALB only. The function does not have to be in the VPC. |
| `alb` | An ALB | Target of an NLB only |

### Stickiness, cross-zone, client IP, draining

| Feature | Facts |
|---------|-------|
| Sticky sessions (session affinity) | A cookie binds a client to one target. Set on the target group. Use when session data is stored on the instance. |
| Duration-based cookie | Made by the load balancer, always named `AWSALB` |
| Application-based cookie | You name your application's cookie per target group. Reserved names are not allowed: `AWSALB`, `AWSALBAPP`, `AWSALBTG`. |
| Stickiness limits | The client must accept cookies. Not supported when cross-zone is off. WebSocket connections are sticky anyway. |
| `X-Forwarded-For` | Holds the real client IP. Without it the application only sees the load balancer's private IP. |
| Deregistration delay | Also called connection draining. A deregistering target is `draining`: no new requests, in-flight requests may finish. Default 300 s, range 0 to 3600 s. Set it to about your longest request. |
| IP targets | A deregistered IP cannot be registered again until the delay has ended |

| Cross-zone load balancing | Default | Can you change it? |
|---------------------------|---------|--------------------|
| ALB | On | Not on the load balancer; it can be turned off per target group |
| NLB | Off | Yes. Data between zones is charged when on. |
| CLB | Off through the API or CLI, on when made in the console | Yes |

- **On**: every target gets an equal share, whatever its zone.
- **Off**: each zone gets an equal share and splits it among its own targets. A zone with few targets overloads them.

```bash
aws elbv2 modify-load-balancer-attributes --load-balancer-arn <arn> \
  --attributes Key=load_balancing.cross_zone.enabled,Value=true
```

### Not in the video

| Symptom or topic | Fact |
|------------------|------|
| Target is unhealthy | Check, in order: the target's security group allows the health check port from the load balancer's security group; the path returns the expected code; the application listens on that port; it answers within the timeout |
| HTTP 502 | The target sent a malformed response or closed the connection |
| HTTP 503 | The target group has no registered targets |
| HTTP 504 | The target did not answer before the idle timeout (ALB default 60 s), or a security group or NACL blocks the load balancer |
| Whose error is it | `HTTPCode_ELB_5XX_Count` comes from the load balancer, `HTTPCode_Target_5XX_Count` from your application |
| Other metrics | `HealthyHostCount`, `UnHealthyHostCount`, `TargetResponseTime` |
| Access logs | Off by default. Delivered to an S3 bucket whose policy allows the delivery. |

## Memory hooks

- **A**LB reads the **a**pplication request (layer 7). **N**LB only sees the **n**etwork connection (layer 4). **G**WLB is a **g**ate to appliances (layer 3).
- The load balancer benches a sick player. The Auto Scaling group replaces the player.
- Rule = IF condition THEN action, lowest number first.
- 502 bad answer, 503 nobody home, 504 too slow.

## Exam traps

- "Targets are unhealthy but the application works when tested on the instance": the instance's security group does not allow the load balancer, or the health check path or code is wrong.
- "Clients must allow-list fixed IP addresses for an application on an ALB": NLB in front of the ALB, or Global Accelerator.
- "Application logs show only the load balancer's IP": read `X-Forwarded-For`. An NLB keeps the client IP.
- "Requests are cut off during deployments or scale-in": raise the deregistration delay. "Deregistration takes too long": lower it.
- "Send `/api/*` and `/images/*` to different fleets with one load balancer": ALB path-based rules. Several domains: host-based rules.
- "Unhealthy instances behind the load balancer are never replaced": the Auto Scaling group still uses the EC2 health check type; change it to ELB.

## Outdated or wrong in the video

- **"Cognito authentication, probably through a Lambda."** The ALB has built-in `authenticate-cognito` and `authenticate-oidc` actions. No Lambda is involved. Very sure.
- **"NLB cross-zone is on by default" and "ALB cross-zone cannot be turned off."** NLB is off by default. ALB can be turned off per target group. Very sure.
- **"NLB has no configurable deregistration delay."** It has the same setting: 0 to 3600 seconds, default 300. Very sure.
- **"Send traffic to an ALB from an NLB or an ALB."** Only an NLB can have an ALB as a target. Very sure.
- **"NLB works on layer 3 or 4."** NLB is layer 4. Layer 3 is the Gateway Load Balancer. Very sure.
- **"The load balancer's default DNS name has HTTPS."** It does not. An HTTPS listener needs a certificate, which is why his first deploy fails. Very sure.
- **Health states "in service" and "out of service".** That is Classic Load Balancer wording. Target groups report `healthy`, `unhealthy`, `initial`, `draining` and `unused`. Very sure.

Related: [[FCC 20 - ASG]] · [[FCC 02 - Amazon EC2]] · [[FCC 15.1 - VPC core, NACLs, security groups and route tables]] · [[FCC 27 - AWS Certificate Manager]] · [[FCC 18 - AWS Global Accelerator]] · [[FCC 00 - Watch guide]]
