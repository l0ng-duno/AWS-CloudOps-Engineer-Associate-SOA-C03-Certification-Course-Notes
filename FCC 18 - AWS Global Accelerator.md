# FCC 18 - AWS Global Accelerator

> Video chapter [39:11:18](https://youtu.be/5JTBS09e7ew?t=141078) to 39:27:09 · 16 min total · 2 min lecture, 14 min lab
> Fast path: read this note and skip the video. The lecture is one slide and the lab only proves that a page loads through the accelerator. The exam points (static IPs, failover, how it differs from CloudFront) are in "Not in the video" below.

## Summary

- Global Accelerator sends users to the nearest AWS edge location, then carries their traffic over the AWS network to your application instead of the public internet.
- Hierarchy: **accelerator** → **listener** (ports and protocol) → **endpoint group** (one per Region) → **endpoints**.
- Endpoints can be an Application Load Balancer, a Network Load Balancer, an EC2 instance or an Elastic IP.
- Two types: **standard** (routes to the nearest healthy endpoint) and **custom routing** (you map users to one specific EC2 instance).
- It does not cache anything. That is the main difference from CloudFront.

## Labs and demos

| Time | Len | What it shows | Takeaway | Verdict |
|------|-----|---------------|----------|---------|
| [39:12:45](https://youtu.be/5JTBS09e7ew?t=141165) | 14m | Builds a standard accelerator with CloudFormation: an EC2 Apache server, an Elastic IP, the accelerator, a listener on TCP 80 and an endpoint group in ca-central-1 with the Elastic IP as its endpoint. Opens the accelerator's DNS name, then runs the public speed comparison tool. | Works after one small fix (a stray `!Ref` in the endpoint ID). The web page loads through the accelerator's DNS name. He cannot tell whether it is faster, and the comparison tool shows mixed results. Health checks, failover and the traffic dial are not demonstrated. | Skip |

## Lecture index

| Time | Topic |
|------|-------|
| [39:11:18](https://youtu.be/5JTBS09e7ew?t=141078) | What it is, standard and custom routing, listeners, endpoint groups, endpoints, traffic dial |

## Notes

### Components

| Part | What it does |
|------|--------------|
| Accelerator | The top-level resource. Gives you a DNS name and static IP addresses as a fixed entry point. |
| Listener | Accepts traffic on the ports and protocol you set (TCP or UDP) and passes it to an endpoint group |
| Endpoint group | The endpoints in one Region. Has a **traffic dial**, a percentage from 0 to 100, to shift traffic between Regions. |
| Endpoint | The resource that receives traffic: ALB, NLB, EC2 instance or Elastic IP |

### CloudFormation resources he uses

| Resource | Key properties |
|----------|----------------|
| `AWS::GlobalAccelerator::Accelerator` | `Name` is required. It cannot start with a hyphen. |
| `AWS::GlobalAccelerator::Listener` | `AcceleratorArn`, `Protocol`, `PortRanges` |
| `AWS::GlobalAccelerator::EndpointGroup` | `ListenerArn`, `EndpointGroupRegion`, `TrafficDialPercentage`, `EndpointConfigurations` |

The `EndpointId` value depends on the endpoint type:

| Endpoint | `EndpointId` |
|----------|--------------|
| Load balancer | Its ARN |
| EC2 instance | The instance ID |
| Elastic IP | The allocation ID, e.g. `!GetAtt StaticIp.AllocationId` |

### Not in the video

| Topic | Fact |
|-------|------|
| Static IPs | Each accelerator gets two static anycast IPv4 addresses. They do not change, so clients and firewalls can allow-list them. |
| Failover | Endpoints are health-checked. Traffic moves to a healthy endpoint or another Region in under a minute, with no wait for DNS caches to expire. |
| Weights | Endpoints inside a group have weights. The traffic dial works one level up, per Region. |
| Protocols | TCP and UDP, so it suits non-HTTP workloads such as gaming, VoIP and IoT |
| Client IP | The client IP address can be preserved for ALB and EC2 endpoints |

| | CloudFront | Global Accelerator |
|---|------------|--------------------|
| Caches content | Yes | No |
| Traffic | HTTP and HTTPS | Any TCP or UDP |
| Entry point | A DNS name with changing IPs | Two static anycast IPs |
| Pick it for | Web content that can be cached | Static IPs, fast Regional failover, non-HTTP traffic |

## Memory hooks

- CloudFront brings the **content** closer to the user. Global Accelerator brings the **network entrance** closer to the user.
- Accelerator → Listener → Endpoint group → Endpoint: "**A** **L**ong **E**xpress **E**ntrance".
- Traffic dial = volume knob per Region. Weight = volume knob per endpoint.

## Exam traps

- "The application needs fixed IP addresses that clients can allow-list, in front of load balancers in several Regions": Global Accelerator.
- "Fail over between Regions without waiting for DNS TTLs": Global Accelerator, not Route 53 failover routing.
- "Speed up a UDP game server for players worldwide": Global Accelerator. CloudFront handles HTTP only.
- "Move 10% of traffic to a new Region": lower or raise the traffic dial on the endpoint groups.
- "Cache static files at the edge": CloudFront, not Global Accelerator.

## Outdated or wrong in the video

- **"For specific EC2 instances we'd have to use custom routing."** A standard accelerator takes EC2 instances and Elastic IPs as endpoints, as his own lab then shows. Custom routing is for sending each user to one chosen instance and port, for example a game session. Very sure.
- **"Like a reverse CDN."** A loose analogy. Nothing is cached; it is a network path optimisation. Very sure.

Related: [[FCC 17 - CloudFront]] · [[FCC 16 - Amazon Route 53]] · [[FCC 19 - ELB]] · [[FCC 29 - AWS CloudFormation]] · [[FCC 00 - Watch guide]]
