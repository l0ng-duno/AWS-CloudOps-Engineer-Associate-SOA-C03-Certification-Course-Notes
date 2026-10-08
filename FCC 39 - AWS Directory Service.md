# FCC 39 - AWS Directory Service

> Video chapter [66:10:28](https://youtu.be/5JTBS09e7ew?t=238228) to 66:12:59 · 3 min total · all lecture, no lab
> Fast path: read this note and skip the video. It is one slide listing the directory types. The only hands-on use of the service is inside the Transfer Family lab (FCC 38).

## Summary

- Directory Service gives AWS workloads a Microsoft Active Directory (AD), either by running one for you or by connecting to yours.
- Three real choices: AWS Managed Microsoft AD, AD Connector, Simple AD. The console also lists Amazon Cognito user pools.
- Pick by one question: where do the users live, and do you need full AD features?

## Labs and demos

There is no lab in this chapter.

## Lecture index

| Time | Topic |
|------|-------|
| [66:10:30](https://youtu.be/5JTBS09e7ew?t=238230) | What the service is; Simple AD and Samba 4 |
| [66:11:25](https://youtu.be/5JTBS09e7ew?t=238285) | AD Connector, AWS Managed Microsoft AD, Cognito |
| [66:12:05](https://youtu.be/5JTBS09e7ew?t=238325) | Simple AD limits: few Regions, no PowerShell automation |

## Notes

### What it is, when it is the answer, what it is confused with

| Type | What it is | Choose it when |
|------|------------|----------------|
| AWS Managed Microsoft AD | A real Windows Server Active Directory that AWS runs for you | You need full AD features in the cloud, or a trust with an on-premises AD |
| AD Connector | A proxy that forwards requests to your on-premises AD; it stores nothing in AWS | Users must stay in the existing on-premises directory |
| Simple AD | An AD-compatible directory built on Samba 4, cheap and basic | A small, standalone directory with simple needs |
| Amazon Cognito user pools | Sign-up and sign-in for the users of your web or mobile app | The users are customers of an app, not staff |

Do not confuse these with IAM Identity Center, which gives staff single sign-on to AWS accounts. It can use Managed Microsoft AD or AD Connector as its user source.

### Simple AD limits he states

- It only emulates AD. It is not the full product.
- It is offered in a few Regions only.
- You cannot script its setup with the AD PowerShell tools. Users and groups are made by hand from a domain-joined Windows machine.

### Not in the video

| Point | Detail |
|-------|--------|
| Trust relationships | Only AWS Managed Microsoft AD supports trusts with another AD |
| AD Connector network | Needs a VPN or Direct Connect link to the on-premises domain controllers |
| High availability | A directory is placed in two subnets in different Availability Zones |
| Sizes | Simple AD: Small (up to 500 users) or Large (up to 5,000). Managed Microsoft AD: Standard or Enterprise edition |
| IAM Identity Center | Does not work with Simple AD |

## Memory hooks

- Managed = the real thing, rented. Connector = a pipe to yours. Simple = a small look-alike.
- "Connector" connects. It keeps no copy of your users.
- Cognito is for customers. Directory Service is for the company.

## Exam traps

- "Use existing on-premises AD credentials in AWS without copying the directory": AD Connector.
- "Needs a trust between AWS and the on-premises AD": AWS Managed Microsoft AD.
- "Lowest cost directory for a few hundred users, basic features": Simple AD.
- "Sign-in for mobile app users, including social logins": Cognito, not Directory Service.

## Outdated or wrong in the video

- **"In Europe it's probably EU East 1; Tokyo 1."** Simple AD is offered in six Regions: N. Virginia (`us-east-1`), Oregon (`us-west-2`), Ireland (`eu-west-1`), Singapore, Sydney and Tokyo (`ap-northeast-1`). Very sure.
- **"Cognito is probably for LDAP-aware apps."** Cognito user pools do not speak LDAP. Apps use them through OAuth 2.0, OpenID Connect or SAML. Very sure.

Related: [[FCC 38 - AWS Transfer Family]] · [[FCC 21 - Identity and Access Management IAM]] · [[FCC 12 - FSx]] · [[FCC 06 - RDS]] · [[FCC 00 - Watch guide]]
