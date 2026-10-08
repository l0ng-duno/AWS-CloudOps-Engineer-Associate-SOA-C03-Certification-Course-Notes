# FCC 15.3 - VPC VPN, NAT, bastions, Transit Gateway and peering

> Video section [34:10:38](https://youtu.be/5JTBS09e7ew?t=123038) to 37:07:30 (part 3 of 3 of the VPC chapter) · 2h57m total · about 35 min lecture, 2h22m labs
> Fast path: read this note; no lab needs watching in full. If time allows, watch the Reachability Analyzer part of the Transit Gateway lab (7 min from [36:34:36](https://youtu.be/5JTBS09e7ew?t=131676)) and the peering commands and proof (5 min, times in the table). Skip the bastion and Lattice labs.

## Summary

- Site-to-Site VPN is an IPsec link over the internet between a customer gateway (your side) and a virtual private gateway or transit gateway (AWS side). Every connection has two tunnels.
- A NAT gateway gives private subnets outbound-only IPv4 internet access. It needs three things: a public subnet, an Elastic IP, and a `0.0.0.0/0` route to it in the private route table.
- A NAT instance is the do-it-yourself version: an EC2 instance you patch and size, with the source/destination check off.
- VPC peering is a one-to-one private link with no transitive routing. Transit Gateway is the hub for many VPCs, VPN and Direct Connect.
- Session Manager usually removes the need for a bastion host.

## Labs and demos

| Time | Len | What it shows | Takeaway | Verdict |
|------|-----|---------------|----------|---------|
| [34:27:50](https://youtu.be/5JTBS09e7ew?t=124070) | 20m | NAT gateway: a private instance can reach Systems Manager only through a NAT gateway | Mostly troubleshooting, and the two failures are the lesson. Creation fails in a VPC with no internet gateway. Then Session Manager cannot connect until the private route table gets `0.0.0.0/0` → NAT gateway and the instance is stopped and started. Watch [34:43:39](https://youtu.be/5JTBS09e7ew?t=125019) to 34:46:25 (3 min) for the route fix. | Skim |
| [34:49:28](https://youtu.be/5JTBS09e7ew?t=125368) | 17m | NAT instance built by hand on Amazon Linux 2023: IP forwarding plus an iptables masquerade rule | His user data did nothing (`sysctl` lacked `sudo`, iptables was not installed), so he reruns the commands by hand. `curl ifconfig.me` from the private instance then returns the NAT's public IP. On AL2023 the interface is `ens5`, not `eth0`. | Skip |
| [35:09:22](https://youtu.be/5JTBS09e7ew?t=126562) | 37m | Bastion host using the Apache Guacamole image from AWS Marketplace | SSH to a Linux instance works in the browser once he adds the connection by hand ([35:25:04](https://youtu.be/5JTBS09e7ew?t=127504) to 35:28:07, 3 min). RDP to Windows through Guacamole never works and he gives up. About 25 minutes are dead ends. | Skim |
| [35:49:32](https://youtu.be/5JTBS09e7ew?t=128972) | 19m | VPC Lattice: a service network and two services, each with a listener and a target group pointing at an EC2 web server | From instance 1, `wget` to the Lattice domain name of service 2 returns the page ([36:03:48](https://youtu.be/5JTBS09e7ew?t=129828)). The names only work inside a VPC associated with the service network. Tear-down is manual and in reverse order: associations, services, targets, target groups, network. | Skip |
| [36:11:52](https://youtu.be/5JTBS09e7ew?t=130312) | 32m | Transit Gateway joining two VPCs, debugged with Reachability Analyzer | Private-IP traffic fails for over 20 minutes because the VPC route tables are wrong. Reachability Analyzer names the fault: no route to the transit gateway. Watch [36:34:36](https://youtu.be/5JTBS09e7ew?t=131676) to 36:41:47 (7 min). | Skim |
| [36:50:58](https://youtu.be/5JTBS09e7ew?t=132658) | 17m | VPC peering with the CLI, then proof that traffic uses the peering link | Create, accept, add a route each way. After he removes VPC B's internet route, the private IP still answers and the public IP does not. Watch [36:52:01](https://youtu.be/5JTBS09e7ew?t=132721) to 36:55:08 (commands) and [37:04:21](https://youtu.be/5JTBS09e7ew?t=133461) to 37:06:26 (proof). | Skim |

### Useful moments inside the bastion lab

| Time | What you learn |
|------|----------------|
| [35:14:49](https://youtu.be/5JTBS09e7ew?t=126889) | Launched in the public subnet but with no public IP, so he attaches an Elastic IP |
| [35:16:52](https://youtu.be/5JTBS09e7ew?t=127012) | Login is `guacadmin` with the instance ID as the password |
| [35:17:54](https://youtu.be/5JTBS09e7ew?t=127074) | Six minutes waiting for Guacamole to discover instances by itself; it never does |
| [35:25:04](https://youtu.be/5JTBS09e7ew?t=127504) | Adds an SSH connection: private IP, port 22, `ec2-user`, pasted private key. `whoami` works |
| [35:35:15](https://youtu.be/5JTBS09e7ew?t=128115) | Windows "Get password" decrypts the Administrator password with the key pair's private key |
| [35:40:21](https://youtu.be/5JTBS09e7ew?t=128421) | The native RDP client fails until the security group allows TCP 3389 |
| [35:43:24](https://youtu.be/5JTBS09e7ew?t=128604) | Guacamole RDP still fails; he abandons it |

### Useful moments inside the Transit Gateway lab

| Time | What you learn |
|------|----------------|
| [36:13:57](https://youtu.be/5JTBS09e7ew?t=130437) | Creates the transit gateway with default settings, then one attachment per VPC ([36:15:57](https://youtu.be/5JTBS09e7ew?t=130557)) |
| [36:17:00](https://youtu.be/5JTBS09e7ew?t=130620) | A `0.0.0.0/0` route to the transit gateway is rejected: that destination already points at the internet gateway |
| [36:24:23](https://youtu.be/5JTBS09e7ew?t=131063) | Test: the other instance's public IP answers, its private IP does not |
| [36:29:27](https://youtu.be/5JTBS09e7ew?t=131367) | Opens Reachability Analyzer and creates an instance-to-instance path |
| [36:34:36](https://youtu.be/5JTBS09e7ew?t=131676) | Reads the result hop by hop: ENI, security group, NACL, transit gateway. Only the forward path is analysed |
| [36:37:42](https://youtu.be/5JTBS09e7ew?t=131862) | Finding: "no route to destination", the route table has no route to the transit gateway |
| [36:39:44](https://youtu.be/5JTBS09e7ew?t=131984) | He had the targets swapped. Correct: `0.0.0.0/0` → internet gateway, other VPC's CIDR → transit gateway |
| [36:40:45](https://youtu.be/5JTBS09e7ew?t=132045) | Status "Reachable"; `wget` to the private IP now works |

## Lecture index

| Time | Topic |
|------|-------|
| [34:10:38](https://youtu.be/5JTBS09e7ew?t=123038) | AWS VPN overview, IPsec |
| [34:11:49](https://youtu.be/5JTBS09e7ew?t=123109) | Site-to-Site VPN: components, features, pricing, limits |
| [34:15:53](https://youtu.be/5JTBS09e7ew?t=123353) | Virtual private gateway and ASN |
| [34:17:20](https://youtu.be/5JTBS09e7ew?t=123440) | Customer gateway, two tunnels, device configuration |
| [34:19:44](https://youtu.be/5JTBS09e7ew?t=123584) | Transit gateway as the VPN endpoint |
| [34:20:40](https://youtu.be/5JTBS09e7ew?t=123640) | Client VPN |
| [34:23:01](https://youtu.be/5JTBS09e7ew?t=123781) | What NAT is |
| [34:24:34](https://youtu.be/5JTBS09e7ew?t=123874) | NAT gateway: cost, public and private types, DNS64 and NAT64 |
| [34:47:51](https://youtu.be/5JTBS09e7ew?t=125271) | NAT instance |
| [35:06:49](https://youtu.be/5JTBS09e7ew?t=126409) | Jump boxes and bastions |
| [35:46:19](https://youtu.be/5JTBS09e7ew?t=128779) | VPC Lattice |
| [36:08:49](https://youtu.be/5JTBS09e7ew?t=130129) | Transit Gateway |
| [36:44:14](https://youtu.be/5JTBS09e7ew?t=132254) | Traffic Mirroring |
| [36:45:20](https://youtu.be/5JTBS09e7ew?t=132320) | Route 53 Resolver DNS Firewall |
| [36:47:17](https://youtu.be/5JTBS09e7ew?t=132437) | Network Firewall |
| [36:48:38](https://youtu.be/5JTBS09e7ew?t=132518) | VPC peering |

Security groups, NACLs and route tables are in part 1. Internet gateways, endpoints, flow logs and Direct Connect are in part 2.

## Notes

### Site-to-Site VPN

| Component | Where | What it is |
|-----------|-------|------------|
| VPN connection | Between both | The IPsec link from the VPC to on-premises equipment |
| VPN tunnel | Between both | The encrypted path. Two per connection, so a device failure or AWS maintenance fails over to the other |
| Customer gateway (CGW) | AWS resource | A record describing your device: BGP ASN, public IP, optional private certificate |
| Customer gateway device | On premises | The real router or software appliance. You configure it; AWS offers sample config files for common vendors |
| Virtual private gateway (VGW) | AWS side | VPN endpoint attached to one VPC |
| Transit gateway | AWS side | VPN endpoint shared by many VPCs |

| Fact | Detail |
|------|--------|
| Protocol | IPsec, which authenticates and encrypts each packet. IKEv2 and NAT traversal are supported |
| VGW ASN | Amazon default 64512, or your own private ASN. Cannot be changed after creation |
| IPv6 | Not supported on a VGW. Use a transit gateway |
| CIDR blocks | On-premises and VPC ranges must not overlap |
| Pricing | Per connection-hour, plus data transfer out |
| Versus Direct Connect | Direct Connect is private but not encrypted. Add a VPN when encryption is required |

### Client VPN

| Item | Detail |
|------|--------|
| Purpose | Individual users on laptops connect to AWS or on-premises resources |
| Client | OpenVPN-based. AWS has its own desktop client and a self-service portal |
| Authentication | Mutual (certificates), Active Directory, or SAML federation for single sign-on |
| Access control | Security groups and Active Directory groups |
| Tunnel | One per session, no redundancy |
| Typical use | Reach an RDS database in a private subnet without a bastion |

### NAT gateway and NAT instance

NAT rewrites the address in the IP header while a packet is in transit. Here it maps private IPv4 addresses to a public one. IPv6 does not need it.

| | NAT gateway | NAT instance |
|---|-------------|--------------|
| Managed by | AWS | You: an EC2 instance |
| Scaling | Automatic | Fixed by instance size |
| Availability | Redundant inside one AZ | A single instance; failover is your job |
| Cost | About $0.045 an hour each, idle or not, plus a per-GB charge. Three for a month is about $98 before data | The instance price only |
| Address | Elastic IP required | Public IP or Elastic IP |
| Source/destination check | Not applicable | Must be disabled |
| Image | Not applicable | The AWS NAT AMI lost support in 2023. Build your own or use Marketplace |

| NAT gateway type | Reaches | Elastic IP | Subnet |
|------------------|---------|------------|--------|
| Public (default) | The internet, through an internet gateway | Required | Public |
| Private | Other VPCs or on premises, through a transit gateway or VGW | Not allowed | Private |

- Outbound only: nothing on the internet can open a connection to the private instances.
- Create one NAT gateway per AZ and point each AZ's private route table at its own.
- DNS64 (a subnet setting) and NAT64 (on the NAT gateway) let IPv6-only workloads reach IPv4-only services.

NAT instance setup from the lab (Amazon Linux 2023):

```bash
sudo sysctl -w net.ipv4.ip_forward=1
sudo yum install -y iptables
sudo iptables -t nat -A POSTROUTING -o ens5 -j MASQUERADE
sudo iptables -t nat -L -v        # confirm the rule exists
```

Then point `0.0.0.0/0` in the private route table at the instance. He also changes an instance networking setting so it accepts traffic not addressed to itself. That is the source/destination check, though he never names it.

### Bastions

- A hardened EC2 instance in a public subnet, also called a jump box. You SSH (port 22) or RDP (port 3389) to it, then hop to private instances.
- He says AWS has no managed bastion; use a Marketplace image or build one.
- Session Manager is his preferred replacement. A bastion still suits teams that want their own audit or connection controls.
- A NAT gateway is the way out for private instances. It is never a way in.

### Peering versus Transit Gateway

| | VPC peering | Transit Gateway |
|---|-------------|-----------------|
| Shape | One-to-one links | Hub and spoke, a regional virtual router |
| Transitive routing | No | Yes |
| Scale | One connection per pair of VPCs | Up to 5,000 attachments |
| Connects | VPCs only | VPCs, Site-to-Site VPN, Direct Connect, third-party appliances, other transit gateways |
| Accounts and Regions | Same or different, both supported | Shared across accounts with AWS RAM |
| In the data path | No hardware, so no bottleneck or single point of failure | An ENI in a subnet of each attached VPC; bandwidth limit per attachment |
| Cost | No hourly fee. Data is free inside one AZ, charged across AZs or Regions | About $0.05 per attachment-hour plus a per-GB charge |
| VPC route table entry | Peer CIDR → `pcx-...` | Remote CIDR → `tgw-...` |

Peering rules:
- CIDR blocks must not overlap. IPv4 and IPv6 both work.
- No edge-to-edge routing: VPC B cannot use VPC A's internet gateway or NAT gateway.
- Routes are needed in both VPCs, or nothing flows.
- A security group rule can reference a security group in the peer VPC.

```bash
aws ec2 create-vpc-peering-connection --vpc-id vpc-AAA --peer-vpc-id vpc-BBB
aws ec2 accept-vpc-peering-connection --vpc-peering-connection-id pcx-123
aws ec2 create-route --route-table-id rtb-AAA \
  --destination-cidr-block <CIDR of B> --vpc-peering-connection-id pcx-123
aws ec2 create-route --route-table-id rtb-BBB \
  --destination-cidr-block <CIDR of A> --vpc-peering-connection-id pcx-123
```

Transit Gateway in three steps: create it, create one attachment per VPC, add a route to the remote CIDR in every VPC route table. With default settings the transit gateway's own route table fills itself by propagation. Delete attachments before the gateway.

### Reachability Analyzer

- Found under Network Manager in the VPC console. Choose a source and a destination, and optionally a protocol and port.
- The result is "Reachable" or "Not reachable", a hop-by-hop path, and an explanation naming the blocking component.
- It checks the forward path only. Run a second analysis in the other direction for return traffic.
- Refresh the page to see the result.

### Traffic Mirroring, DNS Firewall, Network Firewall

| Service | What it does | Remember |
|---------|--------------|----------|
| Traffic Mirroring | Copies packets from a source ENI to another ENI, a Network Load Balancer with a UDP listener, or a Gateway Load Balancer | Feeds security and monitoring appliances. Adds a VXLAN header. Needs a mirror source, target, filter and session |
| Route 53 Resolver DNS Firewall | Checks DNS queries leaving a VPC against domain lists, with allow and deny rules | Stops DNS exfiltration: a compromised host hiding stolen data in lookups to an attacker's domain. Managed centrally with Firewall Manager |
| AWS Network Firewall | Managed stateful firewall with intrusion detection and prevention at the edge of a VPC | Suricata-compatible rules. Filters traffic through internet gateways, NAT gateways, VPN and Direct Connect. Domain lists and deep packet inspection |

### VPC Lattice

- Application-level networking between services, across VPCs and accounts, even with overlapping CIDRs.
- Building blocks: service network (associated with VPCs) → service → listener with rules → target group (instances, IPs, Lambda functions, Kubernetes pods).
- Adds IAM authorisation and weighted routing for blue/green and canary releases.
- It is not named in the SOA-C03 in-scope service list. Know what it is and move on.

### Not in the video

| Topic | Fact |
|-------|------|
| NAT gateway and security groups | A NAT gateway cannot have a security group. Filter with NACLs and the instances' own security groups |
| NAT gateway capacity | 5 Gbps, scaling automatically to 100 Gbps. CloudWatch `ErrorPortAllocation` above zero means port exhaustion |
| VPN routing | Static routes or BGP. Enable route propagation on the VPC route table so VGW routes appear by themselves |
| VPN monitoring | CloudWatch `TunnelState`: 1 is up, 0 is down. Configure both tunnels |
| Transit gateway route tables | Each attachment is associated with one route table. Separate route tables keep VPCs isolated from each other |
| Reachability Analyzer | It analyses configuration and sends no packets. VPC Flow Logs record real traffic |

## Memory hooks

- **C**GW is the **c**ustomer side, **V**GW is the **V**PC side. One connection, two tunnels.
- NAT gateway checklist: public subnet, Elastic IP, private route. Miss one and it fails.
- Peering is a footbridge between two houses; you cannot cut through a neighbour's house to reach a third. Transit Gateway is the roundabout everyone joins.
- Firewalls by scope: security group = instance, NACL = subnet, Network Firewall = VPC edge, DNS Firewall = domain lookups.

## Exam traps

- "Private instances cannot reach the internet although a NAT gateway exists": the private route table lacks `0.0.0.0/0` → NAT gateway, or the NAT gateway is not in a subnet that routes to an internet gateway.
- "A NAT instance does not forward traffic": disable the source/destination check.
- "One AZ failed and every private subnet lost internet access": there was a single NAT gateway. Use one per AZ.
- "A is peered with B and B with C, but A cannot reach C": peering is not transitive. Peer A with C, or use a transit gateway.
- "The Site-to-Site VPN must carry IPv6": end it on a transit gateway, not a virtual private gateway.
- "Find which component blocks traffic between two resources": Reachability Analyzer.

## Outdated or wrong in the video

- **"You need a NAT gateway per subnet."** Per Availability Zone. Many subnets can route to one NAT gateway. Very sure.
- **NAT gateways are described as zonal only.** Since November 2025 there is also a regional NAT gateway that spans AZs by itself and needs no public subnet. Older questions still expect one per AZ. Fairly sure.
- **"Peering is a star of one central VPC and four others."** There is no limit of four. The default quota is 50 active peering connections per VPC, adjustable to 125. Very sure.
- **"Something like 50,000 VPCs" and "50 Gbps per attachment" for Transit Gateway.** The quota is 5,000 attachments, as he says earlier. Bandwidth is now up to 100 Gbps per VPC attachment per AZ. Very sure.
- **"AWS has no built-in jump box service."** EC2 Instance Connect Endpoint (2023) gives SSH and RDP to private instances with no bastion and no public IP. Very sure.
- **"A public IP cannot be added after launch."** Since April 2024 the auto-assigned public IPv4 can be switched on for a running instance under "Manage IP addresses". Fairly sure.
- **"Lattice cannot do gRPC."** A target group can use gRPC as its protocol version. Fairly sure.
- **App Mesh as the older alternative to Lattice.** AWS App Mesh reached end of support on 30 September 2026. Very sure.
- **`12.0.0.0/16` for VPC B.** That is public address space. Use private ranges: `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`. Very sure.

Related: [[FCC 15.1 - VPC core, NACLs, security groups and route tables]] · [[FCC 15.2 - VPC gateways, IPv6, endpoints and flow logs]] · [[FCC 14 - Cloud Networking]] · [[FCC 16 - Amazon Route 53]] · [[FCC 22 - AWS Systems Manager]] · [[FCC 00 - Watch guide]]
