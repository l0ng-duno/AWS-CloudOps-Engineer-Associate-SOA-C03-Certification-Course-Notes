# FCC 15.1 - VPC core, NACLs, security groups and route tables

> Video section [28:29:36](https://youtu.be/5JTBS09e7ew?t=102576) to 31:48:03 (part 1 of 3 of the VPC chapter) · 3h18m total · about 34 min lecture, 2h44m labs
> Fast path: read this note; the NACL, security group and route table sections are the exam material. Then watch about 17 minutes: the CLI lab from [29:28:25](https://youtu.be/5JTBS09e7ew?t=106105) (6 min), the end of the NACL lab from [31:09:59](https://youtu.be/5JTBS09e7ew?t=112199) (9 min) and the security group rule change at [31:31:43](https://youtu.be/5JTBS09e7ew?t=113503) (2 min). The rest of the labs is editor trouble and CloudFormation typing.

## Summary

- A VPC is a private network inside one Region. All its CLI commands live under `aws ec2`.
- Internet access needs four things: an internet gateway attached to the VPC, a `0.0.0.0/0` route to it in the subnet's route table, a public IP on the instance, and security group plus NACL rules that allow the traffic.
- Security group: stateful, on the instance's network interface, allow rules only. NACL: stateless, on the subnet, numbered allow and deny rules, lowest number first.
- Every subnet uses exactly one route table. With no explicit association it falls back to the main route table.
- Each Region has one default VPC (`172.31.0.0/16`). `aws ec2 create-default-vpc` replaces a deleted one.

## Labs and demos

| Time | Len | What it shows | Takeaway | Verdict |
|------|-----|---------------|----------|---------|
| [28:36:18](https://youtu.be/5JTBS09e7ew?t=102978) | 13m | Console tour, then two builds: the "VPC and more" wizard, and a VPC made piece by piece (two /20 subnets, internet gateway, default route) | The wizard makes everything in one go, and deleting a VPC in the console removes it all again. A hand-made VPC starts with only a main route table. The gateway must be attached before it can be a route target. Two VPCs with the same CIDR are accepted. | Skip |
| [28:49:20](https://youtu.be/5JTBS09e7ew?t=103760) | 62m | Writes two bash scripts with the CLI: one builds a VPC with a public subnet, one tears it down | See "Useful moments". A VPC from `create-vpc` has DNS hostnames off and its subnet gives out no public IPs. `delete-route-table` fails on the main route table. About 12 minutes are lost to Gitpod freezing and a move to Codespaces (most of 29:02 to 29:17). | Skim: [29:28:25](https://youtu.be/5JTBS09e7ew?t=106105) to 29:33:55 |
| [29:56:49](https://youtu.be/5JTBS09e7ew?t=107809) | 2m | Deletes the default VPC in the console and recreates it from CloudShell | The console warns that every launch must now name a subnet. `aws ec2 create-default-vpc` brings a new one back in seconds. | Skip |
| [30:00:09](https://youtu.be/5JTBS09e7ew?t=108009) | 19m | Shares a subnet with a second account through RAM, then launches instances from both accounts | The share is made in RAM: pick the subnet, then the other account ID ([30:03:51](https://youtu.be/5JTBS09e7ew?t=108231)). The VPC then shows up in the second account. Launching there with the owner's security group fails: "specify a security group that you own" ([30:11:03](https://youtu.be/5JTBS09e7ew?t=108663)). Neither account sees the other's instance. Much of the lab is confusion over which account is which. | Skip |
| [30:22:31](https://youtu.be/5JTBS09e7ew?t=109351) | 58m | Blocks his own IP address with a NACL deny rule in front of an Apache web server | See "Useful moments". Most of the first 47 minutes builds the test instance with CloudFormation. The NACL part is the last 9: deny rule 90 for his `/32` makes the page time out, and removing it brings the page back at once. | Skim: [31:09:59](https://youtu.be/5JTBS09e7ew?t=112199) to 31:19:00 |
| [31:25:48](https://youtu.be/5JTBS09e7ew?t=113148) | 8m | Reuses the template, then edits a security group's inbound rule live | With "SSH from My IP" as the only rule the web page hangs. Changing it to HTTP loads the page immediately. Earlier, 3 minutes go on a stack that fails because he uploaded the old template; a stack that fails on first create must be deleted before a retry. | Skim: [31:31:43](https://youtu.be/5JTBS09e7ew?t=113503) to 31:33:45 |
| [31:45:36](https://youtu.be/5JTBS09e7ew?t=114336) | 2m | Creates a custom route table in the console and opens "Edit routes" | Shows subnet associations, the local route that cannot be removed and the list of route targets. | Skip |

### Useful moments inside the CLI lab

| Time | What you learn |
|------|----------------|
| [29:01:20](https://youtu.be/5JTBS09e7ew?t=104480) | `--query Vpc.VpcId --output text` returns a bare ID that a script can store in a variable |
| [29:24:36](https://youtu.be/5JTBS09e7ew?t=105876) | Finding the main route table: `describe-route-tables` filtered on `vpc-id` and `association.main` |
| [29:28:25](https://youtu.be/5JTBS09e7ew?t=106105) | DNS hostnames are disabled on a CLI-made VPC; fixed with `modify-vpc-attribute` |
| [29:30:00](https://youtu.be/5JTBS09e7ew?t=106200) | "Auto-assign public IPv4" is off on the new subnet; fixed with `modify-subnet-attribute` |
| [29:31:10](https://youtu.be/5JTBS09e7ew?t=106270) | `create-route` adds `0.0.0.0/0` to the internet gateway |
| [29:38:06](https://youtu.be/5JTBS09e7ew?t=106686) | `delete-route-table` fails: "has dependencies and cannot be deleted" |
| [29:43:10](https://youtu.be/5JTBS09e7ew?t=106990) | "Max number of internet gateways": detached leftovers from failed runs filled the quota of 5 |
| [29:49:15](https://youtu.be/5JTBS09e7ew?t=107355) | The main route table is deleted with the VPC, so he drops that command |

### Useful moments inside the NACL lab

| Time | What you learn |
|------|----------------|
| [30:27:42](https://youtu.be/5JTBS09e7ew?t=109662) | The default NACL: rule 100 allows all traffic, `*` denies the rest, same in both directions |
| [30:50:15](https://youtu.be/5JTBS09e7ew?t=111015) | For Session Manager use the managed policy `AmazonSSMManagedInstanceCore`; `AmazonEC2RoleforSSM` is deprecated |
| [30:58:35](https://youtu.be/5JTBS09e7ew?t=111515) | Session Manager will not connect. Cause: no public IP, because non-default subnets do not auto-assign one |
| [31:02:48](https://youtu.be/5JTBS09e7ew?t=111768) | Fix in CloudFormation: a `NetworkInterfaces` block with a public IP; subnet and security group must move into it (error at 31:06:30) |
| [31:11:01](https://youtu.be/5JTBS09e7ew?t=112261) | `create-network-acl-entry`: rule 90, deny, his IP as `/32`, all protocols |
| [31:14:20](https://youtu.be/5JTBS09e7ew?t=112460) | The rule went on the default NACL by mistake. His own new NACL holds only the deny-all rule |
| [31:15:28](https://youtu.be/5JTBS09e7ew?t=112528) | Adds rule 100 "allow all" in both directions, then associates the subnet with the new NACL |
| [31:16:29](https://youtu.be/5JTBS09e7ew?t=112589) | Page times out. With the rule removed it still fails: the browser had switched to HTTPS (31:18:00) |
| [31:18:32](https://youtu.be/5JTBS09e7ew?t=112712) | Clean repeat: add the deny, page hangs; remove it, page loads |

## Lecture index

| Time | Topic |
|------|-------|
| [28:29:36](https://youtu.be/5JTBS09e7ew?t=102576) | VPC overview (starts mid-sentence) |
| [28:31:42](https://youtu.be/5JTBS09e7ew?t=102702) | Core components |
| [28:34:17](https://youtu.be/5JTBS09e7ew?t=102857) | Key features, quotas, what costs money |
| [29:51:29](https://youtu.be/5JTBS09e7ew?t=107489) | Default VPC |
| [29:54:23](https://youtu.be/5JTBS09e7ew?t=107663) | Deleting a VPC |
| [29:55:10](https://youtu.be/5JTBS09e7ew?t=107710) | Default (catch-all) route |
| [29:58:34](https://youtu.be/5JTBS09e7ew?t=107914) | Shared VPCs and Resource Access Manager |
| [30:19:11](https://youtu.be/5JTBS09e7ew?t=109151) | NACLs |
| [31:20:42](https://youtu.be/5JTBS09e7ew?t=112842) | Security groups |
| [31:34:05](https://youtu.be/5JTBS09e7ew?t=113645) | Stateful versus stateless |
| [31:38:42](https://youtu.be/5JTBS09e7ew?t=113922) | Route tables: main, custom, destinations, targets |

## Notes

### Quotas, defaults and cost

| Item | Value |
|------|-------|
| Scope | One Region; a VPC never spans Regions |
| VPCs per Region | 5, adjustable |
| Internet gateways per Region | 5, rises with the VPC quota; one per VPC |
| Subnets per VPC | 200 |
| CIDR blocks per VPC | 5 IPv4 and 5 IPv6, up to 50 on request |
| Free | VPC, subnets, route tables, NACLs, security groups, internet gateway, peering |
| Charged | NAT gateway, VPC endpoints, VPN, every public IPv4 address ($0.005 an hour, about $3.65 a month), traffic between Regions |
| DNS hostnames | A VPC setting. On in the default VPC, off after `create-vpc` |
| Overlapping CIDRs | Allowed between separate VPCs, but such VPCs cannot be peered later |

Gateways, endpoints and flow logs are in part 2. VPN, NAT, Transit Gateway and peering are in part 3.

### Default VPC

| Item | Detail |
|------|--------|
| CIDR | `172.31.0.0/16` in every Region (65,536 addresses) |
| Subnets | One public `/20` per Availability Zone (4,096 addresses each) |
| Comes with | Internet gateway, main route table with a route to it, default security group, default NACL, DHCP option set |
| Public IPs | Default subnets auto-assign a public IPv4; subnets you create do not |
| If deleted | Launches must name a subnet in another VPC |
| Recreate | `aws ec2 create-default-vpc --region <region>` |
| Rules | A deleted one cannot be restored. An existing VPC cannot be made the default. Only one per Region. |

### Building and deleting a VPC with the CLI

```bash
VPC_ID=$(aws ec2 create-vpc --cidr-block 10.0.0.0/16 \
  --query Vpc.VpcId --output text)
aws ec2 modify-vpc-attribute --vpc-id $VPC_ID --enable-dns-hostnames '{"Value":true}'
IGW_ID=$(aws ec2 create-internet-gateway \
  --query InternetGateway.InternetGatewayId --output text)
aws ec2 attach-internet-gateway --internet-gateway-id $IGW_ID --vpc-id $VPC_ID
SUBNET_ID=$(aws ec2 create-subnet --vpc-id $VPC_ID --cidr-block 10.0.0.0/20 \
  --query Subnet.SubnetId --output text)
aws ec2 modify-subnet-attribute --subnet-id $SUBNET_ID --map-public-ip-on-launch
RT_ID=$(aws ec2 describe-route-tables \
  --filters Name=vpc-id,Values=$VPC_ID Name=association.main,Values=true \
  --query 'RouteTables[0].RouteTableId' --output text)
ASSOC_ID=$(aws ec2 associate-route-table --route-table-id $RT_ID \
  --subnet-id $SUBNET_ID --query AssociationId --output text)
aws ec2 create-route --route-table-id $RT_ID \
  --destination-cidr-block 0.0.0.0/0 --gateway-id $IGW_ID
```

Name tags are added with `--tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=my-vpc}]'`. Delete in reverse order. The console does this for you when you delete a VPC; the CLI does not.

```bash
aws ec2 detach-internet-gateway --internet-gateway-id $IGW_ID --vpc-id $VPC_ID
aws ec2 delete-internet-gateway --internet-gateway-id $IGW_ID
aws ec2 disassociate-route-table --association-id $ASSOC_ID
aws ec2 delete-subnet --subnet-id $SUBNET_ID
aws ec2 delete-vpc --vpc-id $VPC_ID     # the main route table goes with it
```

Other things that block `delete-vpc`: custom security groups, NACLs and route tables, gateway endpoints, egress-only internet gateways.

### Checklist: instance cannot be reached from the internet

| Check | Where it lives | Fix |
|-------|----------------|-----|
| Internet gateway exists and is attached | VPC | `attach-internet-gateway` |
| Route `0.0.0.0/0` (and `::/0` for IPv6) points to the gateway | The subnet's route table | `create-route` |
| Instance has a public IP | Subnet auto-assign setting, read only at launch | `modify-subnet-attribute`, then relaunch, or attach an Elastic IP |
| Security group allows the port inbound | Network interface | `authorize-security-group-ingress` |
| NACL allows the request in and the reply out | Subnet | `create-network-acl-entry` |

- A hang or timeout points to a security group, NACL or route. "Connection refused" means the packet arrived and nothing listens on that port.
- Security group and NACL changes apply immediately.

### NACL versus security group

| | NACL | Security group |
|---|------|----------------|
| Attached to | Subnet | Network interface of an instance |
| State | Stateless: the reply needs its own rule | Stateful: the reply is allowed automatically |
| Rule types | Allow and deny | Allow only |
| Evaluation | By rule number, lowest first, first match wins | All rules together, no order |
| Block one IP address | Yes, with a deny rule | No |
| Source or destination | CIDR | CIDR, another security group, managed prefix list |
| Out of the box | The default NACL allows everything; one you create denies everything | A new group allows nothing in and everything out |
| How many | One NACL per subnet; one NACL can serve many subnets | 5 per network interface (16 maximum); one group can cover instances in many subnets |
| Boundary | One VPC | One VPC |

| Traffic | NACL | Security group |
|---------|------|----------------|
| Request leaving the instance | Checked against outbound rules | Checked against outbound rules (allow all by default) |
| Reply to that request coming back | Checked against inbound rules | Let through |
| Request arriving from outside | Checked against inbound rules | Checked against inbound rules |
| Reply to that request going out | Checked against outbound rules | Let through |

### NACL rules

- Rule numbers run from 1 to 32,766. Number in steps of 10 or 100 so there is room to insert a rule later.
- A deny must have a lower number than the allow it overrides: deny at 90, allow at 100.
- The final `*` rule denies whatever matched nothing. It cannot be removed.

```bash
NACL_ID=$(aws ec2 create-network-acl --vpc-id $VPC_ID \
  --query NetworkAcl.NetworkAclId --output text)
aws ec2 create-network-acl-entry --network-acl-id $NACL_ID --ingress \
  --rule-number 90 --protocol -1 --cidr-block 203.0.113.7/32 --rule-action deny
```

`--protocol -1` means all protocols and ports. Moving a subnet to another NACL is `replace-network-acl-association`.

### Security group details

| Item | Value |
|------|-------|
| Groups per Region | 2,500 by default, adjustable |
| Rules per group | 60 inbound and 60 outbound, adjustable |
| Groups per network interface | 5 by default, 16 maximum |
| Single address | `/32` for IPv4, `/128` for IPv6 |
| Group as source | "Allow the web tier's group" on the database group; no IP lists to maintain as instances come and go |
| Not filtered at all | Amazon DNS, DHCP, instance metadata, ECS task metadata, Windows licence activation, Amazon Time Sync, the VPC router's reserved addresses |

- CLI order: `create-security-group --vpc-id`, then `authorize-security-group-ingress`, then attach the group to the instance.
- RDS and Fargate use security groups too: a group really attaches to a network interface.

### Route tables

| Fact | Detail |
|------|--------|
| Association | Each subnet uses one route table; one table can serve many subnets |
| Main route table | Created with the VPC, cannot be deleted, used by every subnet without an explicit association |
| Custom route table | For subnets that need different routes, such as a VPN or a security appliance |
| A route | Destination (which addresses: a CIDR or a prefix list) plus target (which gateway or interface carries them) |
| Local route | Always present, cannot be deleted; lets everything inside the VPC reach everything else |
| Default route | `0.0.0.0/0` for IPv4 and `::/0` for IPv6; add both when you use both |
| Router | Exists but is managed by AWS; you only edit its tables |

| Target | Sends traffic to |
|--------|------------------|
| `local` | Other addresses in the VPC |
| Internet gateway | The internet, in and out, IPv4 and IPv6 |
| NAT gateway | The internet, outbound only, for private IPv4 instances |
| Egress-only internet gateway | The internet, outbound only, IPv6 |
| Virtual private gateway | Your own network over VPN |
| Peering connection | Another VPC |
| Transit gateway | A hub joining many VPCs and on-premises networks |
| Instance or network interface | One instance that forwards traffic |
| Gateway Load Balancer endpoint | Third-party security appliances |
| Carrier gateway, core network, Outposts local gateway | Wavelength, Cloud WAN, an Outposts rack |

### Shared VPC with Resource Access Manager (RAM)

| Fact | Detail |
|------|--------|
| What is shared | Subnets, not the VPC |
| With whom | Accounts or OUs in the same AWS Organization, same Region; sharing must be enabled for the organisation in RAM |
| Not shareable | Subnets of a default VPC |
| Set-up | A resource share in RAM: the resources (subnets) and the principals (who gets them) |
| Participant account | Sees the VPC as if local, with the owner's account ID shown, and launches its own resources into it |
| Visibility | Each account sees only the resources it created |
| Why use it | One centrally managed network, with separate accounts for billing and access control |

### Not in the video

| Topic | Fact |
|-------|------|
| Ephemeral ports | Because a NACL is stateless, replies need an outbound (or inbound) allow for ports 1024 to 65535. A NACL that allows only port 443 in both directions breaks the connection. |
| Reserved addresses | AWS keeps 5 addresses in every subnet: the first four and the last. A `/24` gives 251 usable. |
| CIDR size | A VPC or subnet block can be `/16` to `/28`. A subnet sits in exactly one Availability Zone. |
| Route choice | When several routes match, the most specific one (longest prefix) wins. |
| Default security group | Allows inbound only from members of the same group and allows all outbound. It cannot be deleted. |
| NACL size | 20 rules per direction by default, 40 at most. |

## Memory hooks

- **N**ACL: **N**umbered, **N**o memory, at the **N**eighbourhood (subnet) gate, and it can say **N**o.
- A security group is a doorman on the instance with a guest list. Whoever he lets out, he lets back in.
- "Public" is a property of the route table: a subnet is public because it has a route to an internet gateway.
- Destination is where the packet is going; target is the door it leaves through.

## Exam traps

- "Block one malicious IP address": NACL deny rule with a lower number than the allow rule. A security group cannot deny.
- "Subnet moved to a new custom NACL and everything times out": a new NACL denies all. Add allows in both directions, including ephemeral ports for replies.
- "Two instances in the same security group cannot reach each other": add a rule whose source is the group itself.
- "Database must accept only the application servers, which scale in and out": use the application's security group as the source.
- "A subnet meant to be private can reach the internet": with no explicit association it uses the main route table, which routes to the internet gateway.
- "Session Manager cannot connect to an instance in a new VPC": no path to the Systems Manager endpoints. It needs a public IP with an internet gateway route, a NAT gateway, or interface endpoints.

## Outdated or wrong in the video

- **"Public subnets allow public IP addresses, private subnets disallow them."** A subnet is public when its route table has a route to an internet gateway. Auto-assigning public IPs is a separate setting. Very sure.
- **"Stateless means NACLs have allow and deny rules"** and **"security groups act as a stateless firewall".** Slips; his own stateful versus stateless section has it right. Very sure.
- **"Instances in the same security group can automatically talk to each other."** Only if the group has a rule naming itself as source. Groups you create do not. Very sure.
- **"`create-default-vpc` is only available in the CLI."** The console has it too: Your VPCs, Actions, Create default VPC. Very sure.
- **"You cannot switch which route table is the main one."** You can: Actions, Set main route table. Very sure.
- **"There is no limit increase for the 60 rules per security group."** It is adjustable, as long as rules per group times groups per interface stays at or below 1,000. Very sure.
- **He builds VPCs on `172.1.0.0/16` and `12.0.0.0/16`.** AWS accepts them, but they are public internet ranges. Use `10.0.0.0/8`, `172.16.0.0/12` or `192.168.0.0/16`. Very sure.
- **"You need a cross-account role to see the other account's instance in a shared VPC."** Nothing is broken. In a shared subnet each account sees and manages only its own resources. Very sure.
- **A participant "must own" the security group it uses in a shared subnet.** Since October 2024 the VPC owner can also share a security group through RAM. Fairly sure.
- **"AWS recommends that you use the default VPC."** AWS calls it suitable for getting started quickly. Production workloads normally go in a custom VPC with private subnets. Fairly sure.

Related: [[FCC 15.2 - VPC gateways, IPv6, endpoints and flow logs]] · [[FCC 15.3 - VPC VPN, NAT, bastions, Transit Gateway and peering]] · [[FCC 14 - Cloud Networking]] · [[FCC 02 - Amazon EC2]] · [[FCC 29 - AWS CloudFormation]] · [[FCC 00 - Watch guide]]
