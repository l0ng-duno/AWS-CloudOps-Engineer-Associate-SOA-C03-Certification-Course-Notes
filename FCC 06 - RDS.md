# FCC 06 - RDS

> Video chapter [05:16:02](https://youtu.be/5JTBS09e7ew?t=18962) to 06:26:35 · 1h11m total · about 36 min lecture, 35 min lab
> Fast path: read this note and skip the lab. It is 30 minutes of CloudFormation debugging for a 3-minute result.

## Summary

- RDS is a managed relational database service for MySQL, MariaDB, PostgreSQL, Oracle, SQL Server, Db2 and Aurora.
- Multi-AZ is for availability: a synchronous standby with automatic failover. Read replicas are for scaling reads: asynchronous copies you can promote by hand.
- Restoring any backup creates a new DB instance with a new endpoint, and it is slow. Plan your recovery time around that.
- Encryption at rest can only be turned on at creation. To encrypt later, snapshot, copy with encryption, restore.
- A connection that hangs is almost always a security group problem.

## Labs and demos

| Time | Len | What it shows | Takeaway | Verdict |
|------|-----|---------------|----------|---------|
| [05:34:50](https://youtu.be/5JTBS09e7ew?t=20090) | 34m | Adapts an old CloudFormation template to launch a PostgreSQL instance, stores the password in Secrets Manager with the CLI, then connects with `psql` from an EC2 instance | Most of the time goes on template errors: a missing `Resources` key, a wrong `Ref`, and a subnet list that was split twice. `psql` only connects after he adds the default security group to the DB instance. The connection part starts at [06:04:25](https://youtu.be/5JTBS09e7ew?t=21865). | Skip |

## Lecture index

| Time | Topic |
|------|-------|
| [05:16:02](https://youtu.be/5JTBS09e7ew?t=18962) | What RDS is, engines |
| [05:19:08](https://youtu.be/5JTBS09e7ew?t=19148) | Encryption |
| [05:20:22](https://youtu.be/5JTBS09e7ew?t=19222) | Backups |
| [05:22:04](https://youtu.be/5JTBS09e7ew?t=19324) | Restoring a backup |
| [05:23:26](https://youtu.be/5JTBS09e7ew?t=19406) | DB subnet groups |
| [05:24:26](https://youtu.be/5JTBS09e7ew?t=19466) | Multi-AZ |
| [05:25:30](https://youtu.be/5JTBS09e7ew?t=19530) | Read replicas |
| [05:27:50](https://youtu.be/5JTBS09e7ew?t=19670) | Multi-AZ vs read replicas |
| [05:28:58](https://youtu.be/5JTBS09e7ew?t=19738) | DB instances, classes, storage |
| [05:32:48](https://youtu.be/5JTBS09e7ew?t=19968) | Performance Insights |
| [05:33:18](https://youtu.be/5JTBS09e7ew?t=19998) | RDS Custom |
| [06:09:19](https://youtu.be/5JTBS09e7ew?t=22159) | RDS Proxy |
| [06:10:57](https://youtu.be/5JTBS09e7ew?t=22257) | Optimized reads and writes |
| [06:12:37](https://youtu.be/5JTBS09e7ew?t=22357) | IAM authentication |
| [06:14:40](https://youtu.be/5JTBS09e7ew?t=22480) | Kerberos |
| [06:15:52](https://youtu.be/5JTBS09e7ew?t=22552) | Secrets Manager and the master password |
| [06:18:10](https://youtu.be/5JTBS09e7ew?t=22690) | Parameter groups |
| [06:19:13](https://youtu.be/5JTBS09e7ew?t=22753) | Public access |
| [06:20:01](https://youtu.be/5JTBS09e7ew?t=22801) | Connecting publicly and privately |
| [06:24:55](https://youtu.be/5JTBS09e7ew?t=23095) | Security groups |
| [06:25:43](https://youtu.be/5JTBS09e7ew?t=23143) | Blue/green deployments |

## Notes

### Multi-AZ vs read replicas

| | Multi-AZ | Read replica |
|---|----------|--------------|
| Purpose | Availability and durability | Read scaling |
| Replication | Synchronous | Asynchronous |
| Readable | Standby is not used for traffic | Every replica serves reads |
| Scope | Two AZs in one Region | Same AZ, another AZ, or another Region |
| Failover | Automatic | Manual promotion, which breaks replication |
| Backups | Taken from the standby | Not configured by default |
| Prerequisite | None | Automated backups must be enabled |

### Backups

| | Automated backup | Manual snapshot |
|---|------------------|-----------------|
| Retention | 0 to 35 days; 0 turns backups off | Kept until you delete it |
| Restore | Point in time within the retention period | To the moment of the snapshot |
| After the instance is deleted | Gone, unless you keep them | Still there |
| Extras | Stored in S3, taken in your backup window | Copy across Regions, share across accounts, export to S3 |

### Security and access

| Feature | What it does | Remember |
|---------|--------------|----------|
| Encryption at rest | KMS-encrypts storage, backups, snapshots and replicas | Only at creation |
| IAM authentication | Log in with a 15-minute token in place of a password | MySQL, MariaDB, PostgreSQL |
| Secrets Manager | Generates and rotates the master password | `--manage-master-user-password`, rotates every 7 days by default |
| Kerberos | Single sign-on through Active Directory | SQL Server, PostgreSQL, Oracle |
| Security group | Must allow the database port from the client | First thing to check when a connection hangs |
| Public access | Makes the endpoint resolve publicly | The security group still applies |

### Other features

| Feature | One-line meaning |
|---------|------------------|
| DB subnet group | The set of subnets, in at least two AZs, that RDS may launch into |
| Parameter group | Engine settings such as `max_connections`, applied to one or more instances |
| RDS Proxy | A connection pool in front of the database, so many short-lived clients such as Lambda functions do not exhaust connections |
| RDS Custom | OS and host access for Oracle and SQL Server when you need custom patches or third-party software |
| Performance Insights | Shows which queries are loading the database; 7 days of history free, up to 2 years paid |
| Optimized reads and writes | Uses local NVMe storage for temporary tables on supported instance classes |
| Blue/green deployment | A synchronized staging copy for testing upgrades, then a switchover |

### Facts worth memorising

- Storage can be increased but never decreased.
- Each read replica has its own DNS endpoint.
- The DB instance identifier is part of the endpoint name. It is not the database name.
- Default ports: PostgreSQL 5432, MySQL and MariaDB 3306, Oracle 1521, SQL Server 1433.
- Without `--apply-immediately`, a change such as enabling Multi-AZ waits for the maintenance window.
- Private database, how to connect: an instance in the same VPC, a bastion, Session Manager, Client VPN, or Direct Connect. CloudShell cannot reach it by default.

```bash
aws rds generate-db-auth-token --hostname mydb.abc123.us-east-1.rds.amazonaws.com \
  --port 5432 --username andrew
```

## Memory hooks

- Multi-AZ = a spare tyre (there for failure). Read replica = extra lanes (there for traffic).
- **S**ync for **s**afety, **a**sync for **a**dditional reads.
- Restore always means "new instance, new endpoint".

## Exam traps

- "Improve read performance" points to read replicas. "Survive an AZ failure" points to Multi-AZ.
- "Lambda functions cause too many connections" points to RDS Proxy.
- "Encrypt an existing unencrypted database": snapshot, copy the snapshot with encryption, restore from the copy.
- "Application lost its connection string after a restore": the restore created a new endpoint.
- "Remove long-lived database passwords" points to IAM database authentication.

## Outdated or wrong in the video

- **"Multi-AZ instance is for RDS and Multi-AZ cluster is for Aurora."** Both are RDS options. A Multi-AZ DB cluster is RDS for MySQL or PostgreSQL with one writer and two readable standbys. Very sure.
- **"Up to 5 read replicas for MySQL, MariaDB and PostgreSQL."** The limit is now 15 for those engines; Oracle and SQL Server stay at 5. Fairly sure.
- **"RDS Proxy is specifically for Lambda."** Any application can use it, and it also shortens failover time. Very sure.
- **Performance Insights** is being replaced by CloudWatch Database Insights. The exam guide still uses the old name. Fairly sure.

Related: [[02 - Amazon Data Life Cycle Manager]] · [[FCC 07 - Amazon DynamoDB]] · [[FCC 08 - Amazon ElastiCache]] · [[FCC 29 - AWS CloudFormation]] · [[FCC 00 - Watch guide]]
