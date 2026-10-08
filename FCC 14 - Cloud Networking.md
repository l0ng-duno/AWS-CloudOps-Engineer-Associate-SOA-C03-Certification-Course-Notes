# FCC 14 - Cloud Networking

> Video chapter [27:08:51](https://youtu.be/5JTBS09e7ew?t=97731) to 28:29:36 · 1h21m total · about 38 min lecture, 43 min subnetting challenges (short console demos, each followed by an answer slide)
> Fast path: read this note and skip the video. If you already know OSI, binary and CIDR, everything before [27:42:31](https://youtu.be/5JTBS09e7ew?t=99751) is general theory, so read only the AWS rules table and the challenge table. Optional: 2 minutes from [28:21:59](https://youtu.be/5JTBS09e7ew?t=102119), where a misaligned subnet is refused.

## Summary

- Remember OSI layers 3 (network, IP), 4 (transport, TCP/UDP) and 7 (application, HTTP). He says so three times.
- A CIDR prefix `/n` fixes the first n bits. Addresses in the block = 2^(32 − n). `/24` is 256; each step up halves it, each step down doubles it.
- A VPC and a subnet must be between `/16` (65,536 addresses) and `/28` (16).
- AWS reserves 5 addresses in every subnet: the first four and the last. A `/24` subnet has 251 usable, a `/28` has 11.
- Subnets must sit inside the VPC range, must not overlap, and must start on a multiple of their own size.

## Labs and demos

All in the console, in an unused Region with its default VPC deleted. Sizes are checked on cidr.xyz.

| Time | Len | What it shows | Takeaway | Verdict |
|------|-----|---------------|----------|---------|
| [27:48:20](https://youtu.be/5JTBS09e7ew?t=100100) | 5m | 1.1: VPC with 256 addresses | `10.0.0.0/24`: 8 host bits, 2^8 = 256. | Skip |
| [27:53:00](https://youtu.be/5JTBS09e7ew?t=100380) | 3m | 1.2: VPC with 3 usable addresses | 3 + 5 reserved = 8, so `/29`, but the console only accepts `/16` to `/28`. Designed to fail. | Skip |
| [27:56:22](https://youtu.be/5JTBS09e7ew?t=100582) | 3m | 1.3: 65,531 usable addresses | `/16` is 65,536, minus 5. AWS suggests `/16` when unsure. | Skip |
| [27:58:50](https://youtu.be/5JTBS09e7ew?t=100730) | 2m | 1.4: 256 network addresses | No answer: nobody counts networks in the cloud. | Skip |
| [28:02:48](https://youtu.be/5JTBS09e7ew?t=100968) | 3m | 2.1: one subnet filling a `/24` VPC | Subnet `10.0.0.0/24` shows 251 available. No room for another. | Skip |
| [28:06:12](https://youtu.be/5JTBS09e7ew?t=101172) | 4m | 2.2: two halves of a `/24` | `10.0.0.0/25` and `10.0.0.128/25`, 123 available each. | Skip |
| [28:10:14](https://youtu.be/5JTBS09e7ew?t=101414) | 7m | 2.3: 64, 32, 16 and 8 addresses in `192.168.0.0/24` | `.0/26`, `.64/27`, `.96/28`. A reused start address gives an overlap error. `.112/29` is refused. | Skip |
| [28:17:38](https://youtu.be/5JTBS09e7ew?t=101858) | 3m | 2.4: two `/24` in `172.16.0.0/20` | `172.16.0.0/24`, then `172.16.1.0/24`: after .255 the third octet goes up. | Skip |
| [28:20:55](https://youtu.be/5JTBS09e7ew?t=102055) | 4m | 2.5: a `/25` then a `/24` in `172.16.0.0/23` | `172.16.0.128/24` is refused; it must be `172.16.1.0/24`. Addresses .0.128 to .0.255 stay unused. | Skim: [28:21:59](https://youtu.be/5JTBS09e7ew?t=102119) to 28:23:49 |
| [28:25:48](https://youtu.be/5JTBS09e7ew?t=102348) | 4m | 3.1: IPv6 on a VPC and two subnets | The VPC gets an Amazon-provided `/56`. Per subnet, type two hex digits (`01`, `02`) to pick its `/64`. | Skip |

## Lecture index

| Time | Topic |
|------|-------|
| [27:08:51](https://youtu.be/5JTBS09e7ew?t=97731) | OSI model overview |
| [27:10:20](https://youtu.be/5JTBS09e7ew?t=97820) | Protocols, PDUs, the TCP/IP suite |
| [27:12:35](https://youtu.be/5JTBS09e7ew?t=97955) | Layer 1 physical |
| [27:13:29](https://youtu.be/5JTBS09e7ew?t=98009) | Layer 2 data link, MAC addresses |
| [27:15:00](https://youtu.be/5JTBS09e7ew?t=98100) | Layer 3 network |
| [27:16:18](https://youtu.be/5JTBS09e7ew?t=98178) | Layer 4 transport, TCP and UDP |
| [27:18:49](https://youtu.be/5JTBS09e7ew?t=98329) | Layers 5 and 6, session and presentation |
| [27:20:52](https://youtu.be/5JTBS09e7ew?t=98452) | Layer 7 application |
| [27:21:50](https://youtu.be/5JTBS09e7ew?t=98510) | PDU at each layer |
| [27:23:27](https://youtu.be/5JTBS09e7ew?t=98607) | NICs and AWS ENIs |
| [27:25:08](https://youtu.be/5JTBS09e7ew?t=98708) | IP addresses, IPv4 and IPv6 |
| [27:28:15](https://youtu.be/5JTBS09e7ew?t=98895) | Binary maths |
| [27:30:58](https://youtu.be/5JTBS09e7ew?t=99058) | Classful addressing |
| [27:33:32](https://youtu.be/5JTBS09e7ew?t=99212) | Network, subnet, host |
| [27:35:05](https://youtu.be/5JTBS09e7ew?t=99305) | Leading bits, network ID and host ID |
| [27:37:40](https://youtu.be/5JTBS09e7ew?t=99460) | Private ranges, RFC 1918 |
| [27:39:35](https://youtu.be/5JTBS09e7ew?t=99575) | Subnet masks |
| [27:42:31](https://youtu.be/5JTBS09e7ew?t=99751) | AWS reserved addresses |
| [27:43:30](https://youtu.be/5JTBS09e7ew?t=99810) | CIDR notation |
| [27:45:15](https://youtu.be/5JTBS09e7ew?t=99915) | DHCP and VPC DHCP option sets |
| [27:47:06](https://youtu.be/5JTBS09e7ew?t=100026) | How the subnetting challenges work |

## Notes

### OSI model

| # | Layer | PDU | Job | Examples |
|---|-------|-----|-----|----------|
| 7 | Application | Data | Closest to the user | HTTP, SSH, DNS, DHCP, LDAP |
| 6 | Presentation | Data | Formats, encrypts, compresses | JPEG, PNG |
| 5 | Session | Data | Opens, keeps and closes sessions | NetBIOS, RPC |
| 4 | Transport | Segment | End-to-end delivery, flow control | TCP, UDP |
| 3 | Network | Packet | Logical addressing and routing | IPv4, IPv6, IPsec, ICMP |
| 2 | Data link | Frame | Delivery on one network, by MAC address | Ethernet, switch |
| 1 | Physical | Bits | Raw electrical or optical signal | Cables, radio |

- TCP acknowledges every packet, so delivery is guaranteed (web applications). UDP does not, so it is faster and can lose packets (games).
- An ENI is AWS's virtual network card. An instance needs one to communicate and can have several.

### IP addresses and binary

- IPv4 is 32 bits: four octets in dotted decimal, about 4.3 billion addresses. IPv6 is 128 bits, written in hexadecimal.
- An octet is 8 bits worth 128, 64, 32, 16, 8, 4, 2, 1. Add the ones. It holds 256 values, 0 to 255.
- A subnet mask is a run of 1s (network part, fixed) then 0s (host part). `/21` is twenty-one 1s, leaving 11 host bits: 2,048 addresses.

### Classes and private ranges

Classful addressing (1981 to 1993) is dead, but its names survive in the RFC 1918 private ranges.

| Class | Leading bits | First octet | Default mask | Private range |
|-------|--------------|-------------|--------------|---------------|
| A | 0 | 0 to 127 | `/8` | `10.0.0.0/8` |
| B | 10 | 128 to 191 | `/16` | `172.16.0.0/12` |
| C | 110 | 192 to 223 | `/24` | `192.168.0.0/16` |

### CIDR sizes to memorise

| Prefix | Mask | Addresses | Usable in an AWS subnet |
|--------|------|-----------|-------------------------|
| `/16` | 255.255.0.0 | 65,536 | 65,531 |
| `/24` | 255.255.255.0 | 256 | 251 |
| `/25` | 255.255.255.128 | 128 | 123 |
| `/26` | 255.255.255.192 | 64 | 59 |
| `/27` | 255.255.255.224 | 32 | 27 |
| `/28` | 255.255.255.240 | 16 | 11 |
| `/32` | 255.255.255.255 | 1 | One address, for rules and routes |

### AWS rules for VPC and subnet CIDRs

| Rule | Detail |
|------|--------|
| VPC and subnet IPv4 size | `/16` to `/28`; a subnet sits inside the VPC range and overlaps no other subnet |
| Reserved in each subnet | `.0` network, `.1` VPC router, `.2` DNS, `.3` future use, last address (broadcast) |
| Alignment | A block starts on a multiple of its size: a `/25` at .0 or .128, a `/26` at .0, .64, .128 or .192, a `/24` at x.x.N.0 |
| Next free subnet | Previous start + previous size, rounded up to the next aligned start |
| Subnets per VPC | 200 by default |
| IPv6 | `/56` for the VPC, `/64` per subnet; add at creation or later |
| DHCP option set | Sets domain name, DNS servers, NTP servers, NetBIOS name servers and node type for the VPC |

### Not in the video

| Topic | Fact |
|-------|------|
| Why layers 3, 4 and 7 | ALB is layer 7, NLB layer 4, Gateway Load Balancer layer 3. Security groups and NACLs filter at layers 3 and 4. WAF inspects layer 7. |
| Resizing | A VPC or subnet CIDR cannot be changed. Add a secondary CIDR to the VPC and create new subnets in it. |
| DHCP option sets | Cannot be edited. Create a new set and associate it with the VPC. |
| Overlap | VPCs with overlapping CIDRs cannot be peered. |

## Memory hooks

- Layers 1 to 7: "Please Do Not Throw Sausage Pizza Away".
- Anchor on `/24` = 256. Count up to halve (`/25` 128, `/26` 64). Count down to double (`/23` 512, `/22` 1,024, `/20` 4,096).
- AWS takes five: 251 from a `/24`, 11 from a `/28`.

## Exam traps

- "How many instances fit in a `/28` subnet": 11, not 16.
- "Smallest VPC or subnet AWS allows": `/28`. Largest: `/16`.
- "The subnet has run out of addresses": add a secondary CIDR and a new subnet. Existing blocks cannot be resized.
- "Allow exactly one IP address in a security group rule": `/32`.
- "Instances must use a custom DNS server or domain name": a new DHCP option set.

## Outdated or wrong in the video

- **Binary example: the bits add up to 214, then he says the number is really 213.** It is 214. Nothing is subtracted. Very sure.
- **Layers 1 to 4 are media layers, ICMP is a transport protocol, RPC logs in to Windows.** Media layers are 1 to 3. ICMP is layer 3. Remote login to Windows is RDP. Very sure.
- **The third reserved address is reserved "for whatever reason".** It is for the Amazon DNS resolver, which answers at the VPC network address plus two (`10.0.0.2`). Very sure.
- **Five addresses are deducted from the VPC.** They are reserved in every subnet, so four subnets lose 20. Very sure.
- **Answer 2.5: a subnet that starts at the start of an octet can end anywhere.** The rule is alignment to the block size: `172.16.1.0/23` starts on an octet boundary and is still invalid. Very sure.
- **IPv6 sizes are fixed at `/56` and `/64`.** True for the default Amazon-provided block. Since late 2023, blocks allocated through IPAM can be `/44` to `/60` for a VPC and `/44` to `/64` for a subnet. Fairly sure.

Related: [[FCC 15.1 - VPC core, NACLs, security groups and route tables]] · [[FCC 15.2 - VPC gateways, IPv6, endpoints and flow logs]] · [[FCC 16 - Amazon Route 53]] · [[FCC 19 - ELB]] · [[FCC 00 - Watch guide]]
