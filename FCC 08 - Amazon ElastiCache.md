# FCC 08 - Amazon ElastiCache

> Video chapter [06:57:34](https://youtu.be/5JTBS09e7ew?t=25054) to 07:42:14 · 45 min total · about 8 min lecture, 37 min lab
> Fast path: read this note and skip the lab. Most of it is a struggle to install `redis-cli`.

## Summary

- ElastiCache is a managed in-memory data store running Memcached or Redis, used to cache data and cut response times.
- In-memory means very fast and not durable. Treat a cache as something you can lose.
- A cache is reachable only from inside its VPC, or from networks connected to that VPC.
- Two deployment choices: serverless (scales itself) or node-based clusters you size like EC2 instances.

## Labs and demos

| Time | Len | What it shows | Takeaway | Verdict |
|------|-----|---------------|----------|---------|
| [07:05:05](https://youtu.be/5JTBS09e7ew?t=25505) | 37m | Creates a serverless Redis cache with the CLI, fails to reach it from outside the VPC, then launches an EC2 instance to connect with `redis-cli` | About 20 minutes go on installing `redis-cli`: the instance turns out to be Amazon Linux 2, then he switches to Ubuntu. The connection keeps dropping until he adds `--tls`. The working `set` and `get` are at [07:39:23](https://youtu.be/5JTBS09e7ew?t=27563). | Skip |

## Lecture index

| Time | Topic |
|------|-------|
| [06:57:34](https://youtu.be/5JTBS09e7ew?t=25054) | Caching, in-memory stores, ElastiCache features, serverless vs standard, Memcached vs Redis |
| [07:03:25](https://youtu.be/5JTBS09e7ew?t=25405) | Memcached commands |

## Notes

### Serverless vs node-based

| | Serverless | Node-based (standard) |
|---|------------|-----------------------|
| Best for | Unpredictable workloads | Predictable workloads |
| Scaling | Automatic | You manage nodes and clusters |
| Billing | Data stored plus ElastiCache processing units | Number and type of nodes |
| Savings | None | Reserved nodes |

### Memcached vs Redis

| | Memcached | Redis |
|---|-----------|-------|
| Data model | Simple key and string value | Rich data structures: lists, sets, hashes, sorted sets |
| Persistence and snapshots | No | Yes |
| Replication and Multi-AZ failover | No | Yes |
| Pub/sub, transactions, Lua, geospatial | No | Yes |
| Typical use | Caching HTML fragments and simple objects | Sessions, leaderboards, queues, anything beyond plain caching |

### Other features

- **Multi-AZ**: deploy across Availability Zones for high availability.
- **Global Datastore**: replicates a Redis cache to other Regions.
- **Backups**: can be automated (Redis).
- **RBAC**: user and permission management from Redis 6.
- **Outposts**: node-based clusters can run on premises.

```bash
aws elasticache create-serverless-cache --serverless-cache-name my-cache \
  --engine redis --major-engine-version 7
redis-cli -h <endpoint> -p 6379 --tls
```

## Memory hooks

- Memcached is a sticky note: one key, one string, gone when it falls off.
- Redis is a toolbox: many data structures, and it can be backed up.
- Cache connection hangs = wrong network. Cache connection drops = missing TLS.

## Exam traps

- "Need replication, failover, backups or complex data types" points to Redis. "Simplest possible cache, multithreaded" points to Memcached.
- "Reduce read load on a database" can be ElastiCache or read replicas; ElastiCache fits when the same queries repeat.
- A client outside the VPC cannot connect to the endpoint without VPN, peering or a similar link.
- Redis default port is 6379; Memcached is 11211.

## Outdated or wrong in the video

- **Valkey is missing.** ElastiCache now also offers Valkey, the open-source fork of Redis, and prices it lower. New questions may say "Valkey or Redis OSS". Very sure.
- **He is unsure how MemoryDB differs.** MemoryDB is a durable Redis-compatible database meant to be a primary data store; ElastiCache is a cache. Very sure.
- **"Memcached on ElastiCache has no encryption."** Recent Memcached versions support encryption in transit. Fairly sure.

Related: [[FCC 06 - RDS]] · [[FCC 07 - Amazon DynamoDB]] · [[FCC 17 - CloudFront]] · [[FCC 00 - Watch guide]]
