# FCC 15.2 - VPC gateways, IPv6, endpoints and flow logs

> Video section [31:48:03](https://youtu.be/5JTBS09e7ew?t=114483) to 34:10:38 (part 2 of 3 of the VPC chapter) · 2h22m total · about 40 min lecture, 1h42m labs
> Fast path: read this note. The labs are console clicking plus one 39-minute debugging session. If you watch anything, watch the short stretches named in the labs tables (about 15 minutes in all): the IPv6 routes, the endpoint with no subnet, the gateway endpoint being deleted, and flow log records arriving.

## Summary

- An internet gateway (IGW) gives two-way internet access for IPv4 and IPv6. An egress-only internet gateway gives outbound-only access for IPv6. A NAT gateway does the outbound-only job for IPv4.
- A gateway endpoint is a free route table entry for S3 and DynamoDB only. An interface endpoint is a paid network interface (PrivateLink) in your subnet and covers most other services.
- Session Manager in a subnet with no internet path needs interface endpoints for `ssm`, `ssmmessages` and `ec2messages`.
- VPC Flow Logs record traffic metadata (addresses, ports, ACCEPT or REJECT) for a VPC, subnet or network interface, and deliver it to CloudWatch Logs, S3 or Firehose.
- An Elastic IP is a static public IPv4 address that you can move between instances. It is Regional and IPv4 only.

## Labs and demos

| Time | Len | What it shows | Takeaway | Verdict |
|------|-----|---------------|----------|---------|
| [31:54:04](https://youtu.be/5JTBS09e7ew?t=114844) | 3m | Creates an internet gateway in the console, attaches it to a new VPC and adds the default routes | A VPC takes one IGW. A new VPC has no internet route until you add `0.0.0.0/0` and `::/0` pointing at the IGW. Deleting the VPC deletes the IGW too. | Skip |
| [31:58:31](https://youtu.be/5JTBS09e7ew?t=115111) | 19m | Builds a dual-stack VPC with the CLI: IPv6 CIDR, egress-only internet gateway, IGW, NAT gateway with DNS64, then an instance opened through Session Manager | `curl ifconfig.me` returns an IPv6 address, so IPv6 leaves through the egress-only gateway. He pastes his co-founder's commands and phones him about the `64:ff9b::/96` route. The first instance never connects (he does not pick the SSM role on that launch); a second launch works 4 minutes later. Watch [32:06:16](https://youtu.be/5JTBS09e7ew?t=115576) to 32:10:00 (the routes) and [32:16:26](https://youtu.be/5JTBS09e7ew?t=116186) (the result, 1 min). | Skim |
| [32:22:32](https://youtu.be/5JTBS09e7ew?t=116552) | 6m | Launches a web server, allocates an Elastic IP, associates it, then disassociates it | The site answers on the Elastic IP and hangs once it is disassociated. The dialog lets you pick an instance or a network interface. Release the address afterwards or it keeps costing money. The lab starts about 50 seconds after the link. | Skip |
| [32:31:17](https://youtu.be/5JTBS09e7ew?t=117077) | 4m | Shows AWS-managed prefix lists, uses the CloudFront one in a security group rule, then creates his own list | You choose a prefix list (`pl-...`) where you would type a CIDR. He adds it as an outbound rule by mistake. Your own list needs an address family and a maximum number of entries. | Skip |
| [32:38:02](https://youtu.be/5JTBS09e7ew?t=117482) | 4m | Converts an IPv4-only VPC to dual-stack in the console | Three edits: add an Amazon-provided IPv6 CIDR to the VPC, add a `::/0` route to the IGW, add an IPv6 CIDR to the existing subnet. Auto-assign IPv6 and DNS64 are optional subnet settings. | Skip |
| [32:57:01](https://youtu.be/5JTBS09e7ew?t=118621) | 39m | Tries to open Session Manager on an instance in a private subnet with no NAT, using interface endpoints | It connects only at [33:32:33](https://youtu.be/5JTBS09e7ew?t=120753). About 30 minutes are trial and error. Watch [33:07:48](https://youtu.be/5JTBS09e7ew?t=119268) to 33:09:50 (missing subnet) and [33:20:10](https://youtu.be/5JTBS09e7ew?t=120010) to 33:22:12 (the two extra endpoints). | Skim |
| [33:37:42](https://youtu.be/5JTBS09e7ew?t=121062) | 15m | A Lambda function in a private subnet lists an S3 bucket through a gateway endpoint | With the endpoint the listing works. After he deletes the endpoint the function times out, which proves the path. Most of the time goes on Lambda code and an IAM policy. Watch [33:40:49](https://youtu.be/5JTBS09e7ew?t=121249) (creating the endpoint, 1 min) and [33:50:02](https://youtu.be/5JTBS09e7ew?t=121802) to 33:51:30 (works, delete, timeout). | Skim |
| [33:57:30](https://youtu.be/5JTBS09e7ew?t=122250) | 13m | Turns on a flow log for a VPC, sends it to CloudWatch Logs and watches records in Live Tail | ACCEPT records for the web server's network interface appear soon after he loads the page. Five minutes go on building the IAM role by hand. Watch [34:00:14](https://youtu.be/5JTBS09e7ew?t=122414) (the create dialog, 1 min) and [34:07:25](https://youtu.be/5JTBS09e7ew?t=122845) to 34:09:00 (records arriving). | Skim |

### Useful moments inside the egress-only gateway lab

| Time | What you learn |
|------|----------------|
| [32:00:10](https://youtu.be/5JTBS09e7ew?t=115210) | `create-vpc` with `--amazon-provided-ipv6-cidr-block`. The VPC gets a /56, which shows "associating" for a moment. |
| [32:01:11](https://youtu.be/5JTBS09e7ew?t=115271) | The subnet gets an IPv4 CIDR and a /64 IPv6 CIDR, so it is dual-stack. |
| [32:02:11](https://youtu.be/5JTBS09e7ew?t=115331) | `modify-subnet-attribute` turns on DNS64. `create-egress-only-internet-gateway` takes the VPC ID, so there is no separate attach step. |
| [32:06:16](https://youtu.be/5JTBS09e7ew?t=115576) | Route `::/0` to the egress-only gateway. He does not recognise `64:ff9b::/96`. |
| [32:08:18](https://youtu.be/5JTBS09e7ew?t=115698) | Explanation: `64:ff9b::/96` is the NAT64 prefix. Route it to the NAT gateway so IPv6 clients can reach IPv4-only services. |
| [32:09:18](https://youtu.be/5JTBS09e7ew?t=115758) | Adds `0.0.0.0/0` to the IGW. There is no `::/0` route to the IGW, so all IPv6 leaves through the egress-only gateway. |
| [32:16:26](https://youtu.be/5JTBS09e7ew?t=116186) | Result: `curl ifconfig.me` prints an IPv6 address. |

### Useful moments inside the interface endpoint lab

| Time | What you learn |
|------|----------------|
| [33:02:40](https://youtu.be/5JTBS09e7ew?t=118960) | Starting point: Session Manager cannot connect, as expected. |
| [33:04:47](https://youtu.be/5JTBS09e7ew?t=119087) | Creates an interface endpoint for `com.amazonaws.<region>.ssm` with a full-access policy. Still no connection. |
| [33:07:48](https://youtu.be/5JTBS09e7ew?t=119268) | First cause: the endpoint has no subnet selected. He ticks the subnet and a new network interface appears. You need one per Availability Zone. |
| [33:09:50](https://youtu.be/5JTBS09e7ew?t=119390) | Security groups: the endpoint must accept traffic from the instance. He attaches the default group to the endpoint. |
| [33:15:04](https://youtu.be/5JTBS09e7ew?t=119704) | Five minutes in Systems Manager Quick Setup (host management). It changes nothing. Skip. |
| [33:20:10](https://youtu.be/5JTBS09e7ew?t=120010) | Second cause: Session Manager also needs `ssmmessages` and `ec2messages`. He creates both. Still no connection. |
| [33:27:17](https://youtu.be/5JTBS09e7ew?t=120437) | Redeploys the instance in the VPC's default security group, with no public IP and no user data. |
| [33:32:33](https://youtu.be/5JTBS09e7ew?t=120753) | It connects. He never isolates which change fixed it. His lesson: define the endpoints in CloudFormation. |

## Lecture index

| Time | Topic |
|------|-------|
| [31:48:03](https://youtu.be/5JTBS09e7ew?t=114483) | What a gateway is; every AWS service with "gateway" in its name |
| [31:53:04](https://youtu.be/5JTBS09e7ew?t=114784) | Internet gateway |
| [31:57:15](https://youtu.be/5JTBS09e7ew?t=115035) | Egress-only internet gateway |
| [32:17:53](https://youtu.be/5JTBS09e7ew?t=116273) | Elastic IPs; CLI commands from [32:20:30](https://youtu.be/5JTBS09e7ew?t=116430) |
| [32:28:46](https://youtu.be/5JTBS09e7ew?t=116926) | Managed prefix lists |
| [32:35:30](https://youtu.be/5JTBS09e7ew?t=117330) | IPv6 on AWS |
| [32:36:41](https://youtu.be/5JTBS09e7ew?t=117401) | Migrating a VPC from IPv4 to dual-stack |
| [32:41:43](https://youtu.be/5JTBS09e7ew?t=117703) | Network Address Usage |
| [32:44:38](https://youtu.be/5JTBS09e7ew?t=117878) | Direct Connect |
| [32:50:43](https://youtu.be/5JTBS09e7ew?t=118243) | VPC endpoints overview |
| [32:52:20](https://youtu.be/5JTBS09e7ew?t=118340) | PrivateLink |
| [32:55:19](https://youtu.be/5JTBS09e7ew?t=118519) | Interface endpoints |
| [33:35:38](https://youtu.be/5JTBS09e7ew?t=120938) | Gateway Load Balancer endpoints |
| [33:37:15](https://youtu.be/5JTBS09e7ew?t=121035) | Gateway endpoints |
| [33:52:49](https://youtu.be/5JTBS09e7ew?t=121969) | Comparison of the three endpoint types |
| [33:55:38](https://youtu.be/5JTBS09e7ew?t=122138) | VPC Flow Logs |

## Notes

### Gateways at a glance

A gateway sits between two networks and controls the traffic between them.

| Gateway | Job |
|---------|-----|
| Internet gateway (IGW) | Inbound and outbound internet, IPv4 and IPv6 |
| Egress-only internet gateway | Outbound-only internet, IPv6 |
| NAT gateway | Outbound-only internet for private IPv4 (see part 3) |
| Virtual private gateway | AWS end of a VPN connection |
| Customer gateway | On-premises end of a VPN connection |
| Transit gateway | Hub that connects many VPCs (see part 3) |
| Direct Connect gateway | Entry point for a Direct Connect link |
| Gateway Load Balancer | Layer 3 load balancer in front of third-party appliances such as firewalls and IDS/IPS |
| Carrier gateway | Telecom networks, for AWS Wavelength |
| Not VPC parts | API Gateway, Storage Gateway, Backup gateway, IoT device gateway |

### Internet gateway and egress-only internet gateway

| | Internet gateway | Egress-only internet gateway |
|---|------------------|------------------------------|
| Direction | In and out | Out only; replies are allowed back |
| IP version | IPv4 and IPv6 | IPv6 only |
| Route | `0.0.0.0/0` and `::/0` to `igw-...` | `::/0` to `eigw-...` |
| NAT | One-to-one NAT for instances with a public IPv4. None for IPv6. | None |
| Setup | Create, attach to the VPC, add routes | Create for a VPC, add the route |
| IPv4 counterpart | n/a | NAT gateway |

- The default VPC already has an IGW. In your own VPC you create it, attach it and add the routes.
- AWS-assigned IPv6 addresses are public. An IGW would let the internet reach them, so a "private" IPv6 subnet uses the egress-only gateway.

### IPv6, DNS64 and NAT64

| Term | Meaning |
|------|---------|
| Dual-stack | IPv4 and IPv6 side by side. A VPC always keeps IPv4. |
| Amazon-provided IPv6 CIDR | /56 for the VPC, /64 for each subnet |
| DNS64 | Subnet setting. The resolver returns a made-up IPv6 address for a name that only has IPv4. |
| NAT64 | The NAT gateway translates that traffic. Route `64:ff9b::/96` to the NAT gateway. |

- All AWS services support IPv4. IPv6 support varies by service, as IPv6-only or dual-stack, on public or private endpoints.

### Migrating a VPC to dual-stack

1. Associate an IPv6 CIDR block with the VPC.
2. Associate an IPv6 CIDR block with each subnet.
3. Add `::/0` routes: to the IGW for public subnets, to an egress-only gateway for private ones.
4. Add IPv6 rules to security groups.
5. Assign IPv6 addresses to instances. Change the instance type if an old one does not support IPv6.

### Elastic IPs

| Item | Detail |
|------|--------|
| What | A static public IPv4 address from Amazon's pool, or from your own range (BYOIP) |
| Why | An auto-assigned public IP changes when an instance is stopped and started or replaced. An Elastic IP stays, and can be remapped to a standby instance. |
| Scope | One Region. It cannot move to another Region. |
| Attach to | An instance or a network interface |
| IPv6 | Not supported and not needed |
| Cost | Every public IPv4 address is charged by the hour, attached or not |

```bash
aws ec2 allocate-address                     # returns eipalloc-...
aws ec2 associate-address --allocation-id eipalloc-... --instance-id i-...
aws ec2 disassociate-address --association-id eipassoc-...
aws ec2 release-address --allocation-id eipalloc-...
```

- `--network-interface-id` associates with a network interface instead of an instance.
- `--allow-reassociation` lets an address that is already in use move to the new target.
- On `allocate-address`: `--address` tries to get back an address you released, `--public-ipv4-pool` uses your own range, `--network-border-group` targets a Local Zone or Wavelength Zone.

### Managed prefix lists

- A named set of CIDR blocks that you reference in security group rules and route tables as `pl-...`.
- One list holds IPv4 or IPv6, never both.
- AWS-managed lists (S3, DynamoDB, CloudFront, Route 53 health checks, VPC Lattice) are kept up to date by AWS.
- Customer-managed lists hold your own CIDRs and have a fixed maximum number of entries.
- The CloudFront list covers every customer's distributions, not only yours. It is tighter than `0.0.0.0/0`, nothing more.

### Network Address Usage (NAU)

- A count of how much of a VPC's address capacity is used. It matters only for very large VPCs.
- 1 unit: each IP address on a network interface, each extra interface, each prefix on an interface.
- 6 units: Network Load Balancer, VPC endpoint, transit gateway attachment, Lambda function, NAT gateway, EFS mount target.
- Limit: 64,000 per VPC (up to 256,000 on request). Peered VPCs share 128,000 (up to 512,000).
- Turn on metrics with `aws ec2 modify-vpc-attribute --enable-network-address-usage-metrics`, then alarm on the CloudWatch metric `NetworkAddressUsage`.

### Direct Connect

| Item | Detail |
|------|--------|
| What | A dedicated private network link from your premises to AWS |
| Why | More bandwidth, steadier latency than the internet, lower data transfer cost |
| Speeds | 50 to 500 Mbps on the low end; 1, 10 and 100 Gbps ports |
| How you get one | Your equipment sits in a Direct Connect location (a colocation data centre), or you go through a partner |
| Dedicated connection | A physical port for you, billed per hour by AWS |
| Hosted connection | A logical connection through a partner, who sets the billing |
| Pricing factors | Port size (capacity), port hours, data transfer out. Data transfer in is free. |

He says the pricing factors could be examined and the cabling details (fibre, BGP, VLAN tagging) will not be.

### VPC endpoints

An endpoint lets resources in a VPC reach a service without an IGW, NAT device, VPN or Direct Connect. Instances need no public IP and the traffic stays on the AWS network.

| | Interface endpoint | Gateway endpoint | Gateway Load Balancer endpoint |
|---|--------------------|------------------|--------------------------------|
| What it is | A network interface with a private IP in your subnet | A target in your route tables, no network interface | A network interface that hands traffic to a Gateway Load Balancer |
| Reaches | Most AWS services, Marketplace partner services, services in other accounts | S3 and DynamoDB only | Third-party appliances: firewalls, IDS/IPS, deep packet inspection |
| PrivateLink | Yes | No | Yes |
| How traffic is steered | DNS resolves the service name to the private IP | A route to the service, added to the route tables you tick | A route that points at the endpoint |
| Cost | Per hour per AZ (about 7 USD a month each) plus per GB | Free | Per hour plus per GB |
| Placement | One interface per AZ you select; has security groups | Per VPC; no security group | Per AZ |

- PrivateLink has two sides. The consumer creates an interface endpoint. The provider publishes an endpoint service behind a Network Load Balancer.
- Interface and gateway endpoints take an endpoint policy. The default is full access.

### Session Manager in a private subnet

| Requirement | Detail |
|-------------|--------|
| Instance role | `AmazonSSMManagedInstanceCore` |
| Interface endpoints | `com.amazonaws.<region>.ssm`, `.ssmmessages`, `.ec2messages` |
| Subnets | Tick a subnet in every AZ the instances use |
| Security group | The endpoint's group must allow inbound 443 from the instances |
| Private DNS | Leave it enabled so the normal service name resolves to the endpoint. The VPC needs DNS support and DNS hostnames on. |
| After fixing | Reboot or relaunch the instance so the SSM Agent tries again |
| Cost | Three endpoints, multiplied by the number of AZs |

### VPC Flow Logs

| Setting | Choices |
|---------|---------|
| Scope | VPC, subnet, network interface, transit gateway, transit gateway attachment |
| Filter | Accepted, rejected or all traffic |
| Aggregation interval | 10 minutes (default) or 1 minute |
| Destination | CloudWatch Logs, S3, Amazon Data Firehose (he uses the old name, Kinesis Data Firehose) |
| Format | Default (below) or custom |
| For CloudWatch Logs | A log group, and an IAM role trusted by `vpc-flow-logs.amazonaws.com` with `logs:CreateLogGroup`, `CreateLogStream`, `PutLogEvents`, `DescribeLogGroups`, `DescribeLogStreams` |

Default record, 14 fields in this order:

```text
version account-id interface-id srcaddr dstaddr srcport dstport protocol packets bytes start end action log-status
2 123456789010 eni-1235b8ca123456789 172.31.16.139 172.31.16.21 20641 22 6 20 4249 1418530010 1418530070 ACCEPT OK
```

- `protocol` is a number: 6 is TCP, 17 is UDP, 1 is ICMP.
- `start` and `end` are Unix seconds.
- `action` is ACCEPT or REJECT.
- `log-status` is OK, NODATA or SKIPDATA.

### Not in the video

These points are tested and not covered in this part.

| Topic | Fact |
|-------|------|
| Flow log content | Metadata only, never packet contents. Use Traffic Mirroring for payloads. |
| Not logged | Traffic to the Amazon DNS server, instance metadata (`169.254.169.254`), Time Sync, DHCP |
| Reading rejects | Inbound REJECT: a security group or NACL blocked the request. Inbound ACCEPT with outbound REJECT: a NACL blocked the reply, because NACLs are stateless and security groups are not. |
| Changing a flow log | Not possible. Delete it and create a new one. |
| Querying | CloudWatch Logs Insights for a log group, Athena for S3 |
| Gateway endpoint reach | Only from inside that VPC, to same-Region S3 or DynamoDB. From on premises or a peered VPC, use an S3 interface endpoint. To lock a bucket to an endpoint, use the bucket policy condition `aws:sourceVpce`. |
| Direct Connect | Not encrypted by default (add a VPN over it or MACsec). It takes weeks to install, so a Site-to-Site VPN is the usual quick option and backup. |
| Elastic IP quota | 5 per Region by default |

## Memory hooks

- Egress-only gateway is the NAT gateway of IPv6: a one-way door out.
- Gateway endpoint goes in a route table and is free. Interface endpoint goes in a subnet and has a bill.
- Gateway endpoints serve the two "data" services: S3 and DynamoDB.
- Flow log record order: who (account, interface), where (addresses, ports, protocol), how much (packets, bytes), when (start, end), verdict (action, status).

## Exam traps

- "Instances with IPv6 addresses must download updates but must not be reachable from the internet": egress-only internet gateway with a `::/0` route. A NAT gateway is the IPv4 answer.
- "Private instances need S3 or DynamoDB at the lowest cost, without internet": gateway endpoint. If the path starts on premises or in another VPC, it must be an interface endpoint.
- "Session Manager shows the instance as unreachable in a subnet with no NAT": add the `ssm`, `ssmmessages` and `ec2messages` interface endpoints, and check the endpoint security group allows 443.
- "Find out why a connection is refused or timing out": VPC Flow Logs, filter on REJECT. ACCEPT in and REJECT out points at a NACL.
- "A partner allowlists our server's IP and it changed after a stop and start": Elastic IP.
- "Allow only CloudFront to reach the load balancer": security group rule that references the CloudFront origin-facing managed prefix list.

## Outdated or wrong in the video

- **"Elastic IPs cost a dollar when allocated and unassociated."** Since 1 February 2024 every public IPv4 address costs 0.005 USD an hour (about 3.60 USD a month), in use or idle. Very sure.
- **"The IP address changes every time you restart."** It changes on stop and start, not on reboot. Very sure.
- **Reassociation as automatic failover.** `--allow-reassociation` only permits moving an address that is already associated. Nothing re-attaches it for you after a failure. Very sure.
- **"IPv4 subnets can't be migrated."** You add an IPv6 CIDR to an existing subnet and it becomes dual-stack, as his own lab shows. IPv6-only subnets also exist, but only as new subnets. Very sure.
- **"Session Manager does not support IPv6."** AWS now lists IPv6 support for Systems Manager's public and private endpoints, so the NAT64 detour may no longer be needed. Fairly sure.
- **"This instance has no IPv4 outbound" (IPv6 lab).** It had a public IPv4 and a `0.0.0.0/0` route to the IGW. `curl` prefers IPv6 when both exist, so the result still proves the IPv6 path. Fairly sure.
- **"The CloudFront prefix list only exists in us-east-1."** It exists in almost every Region, with a different ID in each, so his test of pasting the us-east-1 ID elsewhere proves nothing. It also counts as 55 rules against the default 60 per security group. Fairly sure.
- **"Three types of VPC endpoint."** There are now five: resource and service-network endpoints (VPC Lattice) were added. The three here are the ones to know. Very sure.
- **"Interface endpoints are bidirectional."** Connections can only be opened from your VPC towards the service. Fairly sure.
- **`ec2messages` endpoint.** SSM Agent 3.3.40.0 and later use `ssmmessages` when it is available, so new instances can work with two endpoints. Exam answers still list three. Fairly sure.
- **Direct Connect numbers.** Dedicated ports now go up to 400 Gbps. Planned maintenance notices come 14, 7 and 1 days ahead (he says 10, 5 and 1), and emergency work has no guaranteed 60-minute warning. Fairly sure.

Related: [[FCC 15.1 - VPC core, NACLs, security groups and route tables]] · [[FCC 15.3 - VPC VPN, NAT, bastions, Transit Gateway and peering]] · [[FCC 14 - Cloud Networking]] · [[FCC 22 - AWS Systems Manager]] · [[FCC 23.1 - CloudWatch Logs and Logs Insights]] · [[FCC 00 - Watch guide]]
