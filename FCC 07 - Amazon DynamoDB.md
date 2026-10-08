# FCC 07 - Amazon DynamoDB

> Video chapter [06:26:35](https://youtu.be/5JTBS09e7ew?t=23195) to 06:57:34 · 31 min total · about 5 min lecture, 26 min labs
> Fast path: read this note and skip the chapter. The labs are mostly typo-fixing, and the lecture is thin.

## Summary

- DynamoDB is a fully managed NoSQL key-value and document database, stored on SSDs across three Availability Zones.
- A table holds items (rows) made of attributes (columns). The primary key is a partition key, optionally with a sort key.
- Reads are eventually consistent by default. Strongly consistent reads are optional and slower.
- You pay either for provisioned read and write capacity or per request (on-demand).

## Labs and demos

| Time | Len | What it shows | Takeaway | Verdict |
|------|-----|---------------|----------|---------|
| [06:31:50](https://youtu.be/5JTBS09e7ew?t=23510) | 17m | Deploys a table with AWS SAM (`AWS::Serverless::SimpleTable`), then inserts one item with `aws dynamodb put-item` | The deploy fails until he notices he pasted the wrong resource type. `put-item` fails four times: missing `file://`, missing type descriptors, wrong table name, wrong key case. | Skip |
| [06:49:01](https://youtu.be/5JTBS09e7ew?t=24541) | 9m | Deploys a full `AWS::DynamoDB::Table` with CloudFormation: partition key, sort key, provisioned 5 read and 5 write units | Fails once because `AttributeDefinitions` listed non-key attributes. Only key attributes go there. | Skip |

## Lecture index

| Time | Topic |
|------|-------|
| [06:26:35](https://youtu.be/5JTBS09e7ew?t=23195) | RDS Extended Support (left over from the RDS chapter, 1 min) |
| [06:27:22](https://youtu.be/5JTBS09e7ew?t=23242) | NoSQL, key-value and document stores, DynamoDB features, table anatomy |
| [06:30:14](https://youtu.be/5JTBS09e7ew?t=23414) | Read consistency |

## Notes

### Table anatomy

| Part | Meaning |
|------|---------|
| Table | The container |
| Item | One record, like a row |
| Attribute | One field of an item, like a column |
| Partition key | Required; decides where the item is stored (`HASH`) |
| Sort key | Optional; orders items that share a partition key (`RANGE`) |

### Read consistency

| | Eventually consistent | Strongly consistent |
|---|-----------------------|---------------------|
| Default | Yes | No, you request it |
| Result | May be stale for up to about a second | Always the latest write |
| Trade-off | Fast | Higher latency |

### Capacity and table options

| Setting | Options |
|---------|---------|
| Billing mode | `PROVISIONED` (you set read and write capacity units) or `PAY_PER_REQUEST` (on-demand) |
| Table class | Standard, or Standard-Infrequent Access |

### CLI item format

```bash
aws dynamodb put-item --table-name my-table --item file://item.json
```

```json
{ "id": { "S": "hello" }, "fruit": { "S": "apple" } }
```

- Every value carries a type descriptor: `S` string, `N` number, `BOOL` boolean.
- A file argument needs the `file://` prefix.
- Attribute names are case sensitive, so `ID` is not `id`.

### From the RDS leftover: Extended Support

- Lets you keep running a major engine version after standard support ends, for a fee, for up to three years. After that AWS upgrades the engine automatically.

## Memory hooks

- **P**artition key picks the **p**lace, **s**ort key **s**orts within it.
- Eventually consistent is like a group chat on bad signal: everyone gets the message, a moment apart.

## Exam traps

- "Unpredictable or spiky traffic" points to on-demand; "steady, predictable traffic" points to provisioned capacity.
- `ProvisionedThroughputExceededException` means the table is being throttled.
- A strongly consistent read uses twice the read capacity of an eventually consistent one.

## Outdated or wrong in the video

- **The feature list reads as if everything is built in.** Multi-Region replication (global tables), in-memory caching (DAX) and point-in-time recovery are options you enable; DAX is a separate service with its own cost. Fairly sure.
- **Coverage is thin.** This chapter does not explain capacity units, auto scaling, indexes, streams, backups or global tables, which are the DynamoDB topics an operations exam tends to ask about. Fairly sure.

Related: [[FCC 06 - RDS]] · [[FCC 08 - Amazon ElastiCache]] · [[FCC 29 - AWS CloudFormation]] · [[FCC 00 - Watch guide]]
