# FCC 02 - Amazon EC2

> Video chapter [00:34:39](https://youtu.be/5JTBS09e7ew?t=2079) to 02:34:01 · 1h59m total · about 50 min lecture, 70 min labs and demos
> Fast path: read this note, then watch only the IMDS lab (11 min). Everything else is covered below.

## Summary

- EC2 is a resizable virtual server. You pick an AMI, an instance type, storage, then extras: security group, key pair, user data, instance profile.
- User data is a script that cloud-init runs once at first boot. It can be bash, cloud-config YAML, or PowerShell on Windows.
- Instance metadata (IMDS) is read from inside the instance at `169.254.169.254`. IMDSv2 adds a session token and is the secure default.
- An instance type name encodes family, generation, processor, extras and size, e.g. `c7gn.xlarge`.
- Five ways to connect: SSH, EC2 Instance Connect, Session Manager, Fleet Manager remote desktop, Serial Console.

## Labs and demos

| Time | Len | What it shows | Takeaway | Verdict |
|------|-----|---------------|----------|---------|
| [00:38:20](https://youtu.be/5JTBS09e7ew?t=2300) | 11m | Opens an AL2023 instance through Session Manager and reads the cloud-init files | Logs are in `/var/log/cloud-init.log` and `/var/log/cloud-init-output.log`. Config is in `/etc/cloud/cloud.cfg` and `/etc/cloud/cloud.cfg.d/`. He browses without reaching a conclusion. | Skip |
| [00:52:25](https://youtu.be/5JTBS09e7ew?t=3145) | 12m | Launches three instances that install Apache through user data: Windows with PowerShell, Linux with bash, Linux with cloud-config | Bash and cloud-config work. The PowerShell one installs and runs Apache but the page never loads, and he leaves it unsolved. His first attempt failed because the VPC had no internet gateway. PowerShell user data must be wrapped in `<powershell>` tags. | Skim |
| [01:09:45](https://youtu.be/5JTBS09e7ew?t=4185) | 11m | Queries IMDS with `curl` and `wget`, then forces IMDSv2 with `aws ec2 modify-instance-metadata-options` and fetches a token | After enforcing v2, the plain `curl` stops working at once. His hand-typed token command returns 403; the one copied from the docs works. | **Watch** |
| [01:26:56](https://youtu.be/5JTBS09e7ew?t=5216) | 4m | Tours the instance type picker, the "Compare instance types" table and Service Quotas | Some instance families have a quota of zero until you request an increase. He advises against trusting the Amazon Q recommendation. | Skip |
| [01:44:53](https://youtu.be/5JTBS09e7ew?t=6293) | 12m | EC2 Instance Connect from the console, then from a terminal with `aws ec2-instance-connect send-ssh-public-key` | The pushed public key is valid for 60 seconds, so SSH in straight away. About 4 minutes are spent setting up access keys in Gitpod. Jump to [01:51:46](https://youtu.be/5JTBS09e7ew?t=6706) for the actual command. | Skim |
| [01:56:47](https://youtu.be/5JTBS09e7ew?t=7007) | 9m | Connects to Windows by RDP file, then by Fleet Manager remote desktop | Needs inbound port 3389. He adds the rule to outbound by mistake and loses several minutes. The Administrator password is decrypted with the key pair's private key. | Skim |
| [02:06:10](https://youtu.be/5JTBS09e7ew?t=7570) | 10m | EC2 Serial Console | Three prerequisites he discovers one by one: enable it in account settings, use a Nitro instance (t3, not t2), and set a root password first through Session Manager. | Skip |

## Lecture index

*Use only if you want the slide.*

| Time | Topic |
|------|-------|
| [00:34:39](https://youtu.be/5JTBS09e7ew?t=2079) | What EC2 is, launch steps |
| [00:36:27](https://youtu.be/5JTBS09e7ew?t=2187) | cloud-init |
| [00:49:21](https://youtu.be/5JTBS09e7ew?t=2961) | User data |
| [01:04:53](https://youtu.be/5JTBS09e7ew?t=3893) | Metadata service, IMDSv1 and v2 |
| [01:20:38](https://youtu.be/5JTBS09e7ew?t=4838) | Instance type naming |
| [01:22:51](https://youtu.be/5JTBS09e7ew?t=4971) | Instance families |
| [01:31:20](https://youtu.be/5JTBS09e7ew?t=5480) | Processors |
| [01:34:54](https://youtu.be/5JTBS09e7ew?t=5694) | Burstable instances |
| [01:38:14](https://youtu.be/5JTBS09e7ew?t=5894) | Source/destination check |
| [01:39:04](https://youtu.be/5JTBS09e7ew?t=5944) | System log |
| [01:40:38](https://youtu.be/5JTBS09e7ew?t=6038) | Placement groups |
| [01:41:43](https://youtu.be/5JTBS09e7ew?t=6103) | Ways to connect |
| [02:16:25](https://youtu.be/5JTBS09e7ew?t=8185) | Amazon Linux versions |
| [02:22:04](https://youtu.be/5JTBS09e7ew?t=8524) | Instance sizes |
| [02:23:24](https://youtu.be/5JTBS09e7ew?t=8604) | Instance profile |
| [02:25:40](https://youtu.be/5JTBS09e7ew?t=8740) | Instance lifecycle and protections |
| [02:28:48](https://youtu.be/5JTBS09e7ew?t=8928) | Console screenshot |
| [02:29:39](https://youtu.be/5JTBS09e7ew?t=8979) | Hostnames |
| [02:32:27](https://youtu.be/5JTBS09e7ew?t=9147) | Default usernames |

## Notes

### User data and metadata

| | User data | Instance metadata |
|---|-----------|-------------------|
| What | Script run by cloud-init at first boot | Facts about the instance, read from inside it |
| Formats / access | Bash, cloud-config YAML, PowerShell | `curl http://169.254.169.254/latest/meta-data/` |
| Note | Base64 only when calling the API directly; console and CLI encode it for you | IPv6 endpoint is `[fd00:ec2::254]` |

| | IMDSv1 | IMDSv2 |
|---|--------|--------|
| Method | Plain GET | PUT for a token, then GET with the token header |
| Protects against | Nothing | SSRF, open proxies and firewalls |
| Enforce | n/a | `--http-tokens required` |
| Symptom when v1 is used on a v2-only instance | n/a | 401 Unauthorized |

```bash
TOKEN=$(curl -s -X PUT "http://169.254.169.254/latest/api/token" \
  -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")
curl -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/public-ipv4
```

### Instance type name: `c7gn.xlarge`

| Part | Example | Meaning |
|------|---------|---------|
| Family | `c` | Workload it is built for |
| Generation | `7` | Higher is newer hardware |
| Processor | `g` | `g` Graviton (ARM), `a` AMD, none usually Intel |
| Extra capability | `n` | e.g. enhanced networking |
| Size | `xlarge` | vCPU and memory; each step up roughly doubles both and the price |

### Instance families

| Family | Letters | Use |
|--------|---------|-----|
| General purpose | T, M | Web servers, code repos. T is burstable and cheapest, M is steady. |
| Compute optimized | C | Modelling, game servers, ad serving |
| Memory optimized | R, X, Z | In-memory caches and databases |
| Accelerated | P, G, F, Inf | Machine learning, GPU, FPGA |
| Storage optimized | I, D, H | NoSQL, data warehousing on local disks |

### Burstable (T) instances

| Mode | Behaviour |
|------|-----------|
| Standard | Bursts above baseline only while it has CPU credits |
| Unlimited | Keeps bursting past its credits and bills the extra CPU |

### Placement groups (free)

| Type | Layout | Use | Limit |
|------|--------|-----|-------|
| Cluster | Packed together in one AZ | HPC, low latency | Single AZ only |
| Partition | One rack per partition | Hadoop, Cassandra, Kafka | |
| Spread | Every instance on its own rack | A few critical instances | 7 per AZ, can span AZs |

### Ways to connect

| Method | Port open? | Key points |
|--------|-----------|------------|
| SSH client | 22 | Key pair made at launch |
| EC2 Instance Connect | 22 | Short-lived key pushed through IAM, Linux only |
| Session Manager | None | Needs the SSM role on the instance, audited, lands as `ssm-user` |
| Fleet Manager remote desktop | None | RDP in the browser for Windows |
| Serial Console | None | Hardware-level troubleshooting, Nitro only |

### AL2 vs AL2023

| | Amazon Linux 2 | Amazon Linux 2023 |
|---|----------------|-------------------|
| Package manager | `yum` | `dnf` |
| Extra packages | Amazon Linux Extras, EPEL 7 | Large built-in set, no EPEL |
| Python | 2.7 | 3 |
| SELinux | Off by default | On by default |
| Root volume | gp2 | gp3 |
| cron | Installed | Not installed by default |

### Other points

- **Instance profile**: a wrapper that passes one IAM role to an instance. The console creates it silently; with the CLI you create it, add the role, then associate it.
- **Source/destination check**: on by default. Turn it off only when the instance forwards traffic, such as a NAT instance.
- **Lifecycle**: pending → running → stopping → stopped → shutting-down → terminated. Reboot is a soft restart. Terminated cannot be recovered.
- **Protections**: termination protection, stop protection, shutdown behaviour (stop or terminate), auto-recovery. All set with `modify-instance-attribute`.
- **Troubleshooting without logging in**: system log (boot output) and instance console screenshot.
- **Default users**: `ec2-user` for Amazon Linux, `ubuntu` for Ubuntu, `admin` for Debian.
- **Hostnames**: IPv4 instances get `ip-10-0-0-1.ec2.internal` in us-east-1 and `ip-10-0-0-1.<region>.compute.internal` elsewhere. IPv6-only instances use the instance ID instead.

## Memory hooks

- IMDS address: 169.254 twice.
- v2 is "two steps": PUT a token, then GET.
- **C**luster = **c**lose, **P**artition = **p**er rack group, **S**pread = **s**eparate racks, **s**even.
- T is for "tiny and bursty", M is for "main, steady".

## Exam traps

- 401 from the metadata endpoint means the instance requires IMDSv2.
- "Connect with no open ports and no key pair" points to Session Manager.
- Cluster placement groups cannot span Availability Zones.
- A NAT instance needs the source/destination check disabled.
- The instructor says he has seen the `169.254.169.254` address asked directly.

## Outdated or wrong in the video

- **"You download the public key."** You download the private key (`.pem`); AWS keeps the public key. Very sure.
- **"Standard is the default burst mode."** That is true only for T2. T3 and T4g launch in Unlimited mode, which can add charges. Very sure.
- **"Spread: max 7 instances."** The limit is 7 running instances per Availability Zone per group. Very sure.
- **"You may have to restart after changing metadata options."** No restart is needed; his own demo shows the change applying at once. Fairly sure.
- **"A hard reboot is required after attaching the first instance profile."** EC2 does not require it. A reboot only makes the SSM Agent notice the role sooner. Fairly sure.
- **Amazon Linux 2** is described as still supported. It reached end of support on 30 June 2026, so use AL2023. Fairly sure of the date.

Related: [[00 - Cloud init EC2]] · [[01 - Module 3 Computing Servers]] · [[FCC 03 - Amazon Machine Images AMIs]] · [[FCC 05 - EBS]] · [[FCC 22 - AWS Systems Manager]] · [[FCC 00 - Watch guide]]
